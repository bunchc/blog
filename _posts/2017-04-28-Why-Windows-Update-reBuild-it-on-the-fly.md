---
title: "Why Windows Update? (re)Build it on the fly."
date: 2017-04-28
layout: "post"
categories: automation, packer, boxstarter, windows, devops
---

This sound familiar? You use a *nix based OS as a daily driver. Be that OSX or some flavor of Linux. You've got say, 95% of what you need to do covered, but every now and again (like pulling up an old Java app, or running a specific version of IE), you keep a Windows VM around.

Ok you think, this not so bad. Until you find, it's been forever since you last opened said Windows VM, and now you need updates and such, and then the app updates, and suddenly you have spent more time caring and feeding Windows than doing work.

At least, I find myself here too often. I hated it. So like any good lazy-admin. I fixed it. I no longer keep a Windows VM around all the time. Instead I build it fresh, each and every time I need it, pulling in all the relevant security patches, tools, and so forth. What follows then, is the actual 'how', behind it.

But first, the short version, because this will be a huge post:

```
brew update
brew cask install packer
brew cask install vagrant
brew cask install vmware-fusion
vagrant plugin install vagrant-vmware-fusion
vagrant plugin license vagrant-vmware-fusion ~/Downloads/license.lic
git clone https://github.com/boxcutter/windows ~/projects/templates
cd ~/projects/templates
make vmware/eval-win7x64-enterprise-ssh.json
vagrant box add \
    -name Win10x64 \
    ./box/vmware/eval-win10x64-enterprise-ssh-nocm-1.0.4.box
```

# Before You Start

Before you head down this road, you will need a few tools:

* [Packer](http://packer.io/)
* [Vagrant](https://www.vagrantup.com/)
* Virtualbox or VMware Fusion/Workstation (examples here are Fusion, but should work either way)
* The ISO files. That is, if you are doing non-eval versions

# Getting Started

We'll work with the assumption that you have your tools installed and, if needed licensed. The tl;dr of that process for OSX is:

```
brew update
brew cask install packer
brew cask install vagrant
brew cask install vmware-fusion
brew cask install virtualbox
vagrant plugin install vagrant-vmware-fusion
vagrant plugin license vagrant-vmware-fusion ~/Downloads/license.lic
```

Once you have the tools, at a high level, the process is:

* Download (or write) packer definitions
* Download  (or write) post-install scripts
* Run the things

## The Packer Environment

The good folks behind boxcutter have produced quite a robust set of [packer environment files](https://github.com/boxcutter/windows) which provide an excellent foundation from which to start. They cover most recent windows versions from win7 => current desktop wise and 2008R2 => current server wise. I would advise forking that repo to provide a good jumping off point.

Now, let's customize one for Windows 10:

```
git clone https://github.com/boxcutter/windows ~/projects/templates
cd ~/projects/templates
```

In here you will want to edit either ```win10x64-enterprise-ssh.json``` or ```eval-win10x64-enterprise-ssh.json``` respectively.

> *Note:* The difference here is between licensed install media or not. Our example follows the eval version.

> *Note 2:* I'm using ssh over winrm here because that works well for me on OSX. The winrm versions of these files are similar enough, however.

You are looking for this section of the file:

```
"provisioners": [
  {
    "environment_vars": [
      "CM={{user `cm`}}",
      "CM_VERSION={{user `cm_version`}}",
      "UPDATE={{user `update`}}"
    ],
    "execute_command": "{{.Vars}} cmd /c C:/Windows/Temp/script.bat",
    "remote_path": "/tmp/script.bat",
    "scripts": [
      "script/vagrant.bat",
      "script/cmtool.bat",
      "script/vmtool.bat",
      "script/clean.bat",
      "script/ultradefrag.bat",
      "script/sdelete.bat"
    ],
    "type": "shell"
  },
  {
    "inline": [
      "rm -f /tmp/script.bat"
    ],
    "type": "shell"
  }
],
```

In that section, between the bat files and the inline script, we inject our custom powershell:

```
"provisioners": [
  {
    "type": "file",
    "source": "script/custom.ps1",
    "destination": "C:/Windows/Temp/custom.ps1"
  },
  {
    "environment_vars": [
      "CM={{user `cm`}}",
      "CM_VERSION={{user `cm_version`}}",
      "UPDATE={{user `update`}}"
    ],
    "execute_command": "{{.Vars}} cmd /c C:/Windows/Temp/script.bat",
    "remote_path": "/tmp/script.bat",
    "scripts": [
      "script/vagrant.bat",
      "script/cmtool.bat",
      "script/vmtool.bat",
      'script/custom_launcher.bat',
      "script/clean.bat",
      "script/ultradefrag.bat",
      "script/sdelete.bat"
    ],
    "type": "shell"
  },
  {
    "inline": [
      "rm -f /tmp/script.bat"
    ],
    "type": "shell"
  }
],
```

## The Install Script

As you saw in the prior section, we added three things:

* A file provisioner to copy in our PowerShell script
* The script ```script/custom_launcher.bat```
* The script ```script/custom.ps1```

The file provisioner tells packer to copy our custom PowerShell script into the VM.

The next thing we did, was right before script/clean.bat, we told packer to run ```custom_launcher.bat```. ```custom_launcher.bat``` is a helper script that launches PowerShell to run ```custom.ps1```, and is a simple one-liner:

```
Powershell.exe -executionpolicy Unrestricted -Command "& c:\Windows\Temp\custom.ps1"
```

> **Note:** We use the helper script approach because the PowerShell provisioner assumes the use of WinRM, which didn't work out of the box on OSX Sierra. If you are using Windows to run packer, you can use the PowerShell provisioner instead.

Finally, the contents of custom.ps1 are here:

<script src="https://gist.github.com/bunchc/9e4e5b5e854e3fbb9f7382880330c3ed.js"></script>

If you're paying attention, the above script is just a consolidated version of the work from [here](http://blog.codybunch.com/2017/03/27/Basic-Automated-Windows-10-post-install/).

## Build the VM

Now that you have all the bits together, it is time to build the VM. From the command line, run the following command:

```
$ pwd
/Users/bunchc/projects/packer-templates/windows

$ make vmware/eval-win10x64-enterprise-ssh
```

After a long while and a *LOT* of output, you will have a shiny Win10 VM vagrant box with Java installed, so you can well, access that one iLO board on that one box that you can't turn down, because 'reasons'.

> **Note:** To enable debug logging, you can run the make command like this: ```PACKER_LOG=1 make vmware/eval-win10x64-enterprise-ssh```

# Use the VM

I had thought to leave this step as an exercise for the end user. Why? Because, while packer will give you a Vagrant box by default, [you can also output this directly into vSphere for use that way](http://blog.codybunch.com/2017/03/08/packerio-with-vSphere/). What we'll do here, for those unfamiliar with Vagrant, is build a minimal Vagrant file and fire up the VM:

```
vagrant box add \
    -name Win10x64 \
    ./box/vmware/eval-win10x64-enterprise-ssh-nocm-1.0.4.box

mkdir -p /projects/Win10-x64

cd /projects/Win10-x64

vagrant init -m Win10x64

vagrant up --provider=vmware_fusion
```

# Cleanup & Rebuilding

Ok, so you logged into whatever thing you needed the Win10VM for, did a bunch of work, and now want the free space back:

```
cd /projects/Win10-x64

vagrant destroy -f

vagrant box remove Win10x64

cd /projects/packer-templates/windows

make clean
```

To rebuild, rerun the make command, which should pull in all the updates and what not that you need.