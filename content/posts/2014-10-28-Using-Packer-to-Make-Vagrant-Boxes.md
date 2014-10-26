---
title: "Using Packer to Make Vagrant Boxes"
date: 2014-10-28
categories: 
---

Part of working on the 3rd edition of the OpenStack Cookbook required, among other things, a new release of Ubuntu that came pre-loaded with the Juno OpenStack packages. Not a problem! Excepting that there were no ready made images on [Vagrant Cloud](http://vagrantcloud.com). Enter Packer.

## Packer?

What the deuce is packer? From the [packer.io](https://packer.io/) site:

> Packer is a tool for creating identical machine images for multiple platforms from a single source configuration.

Said another way, it takes the 'configuration is code' and pushes it back to the golden images, to a degree. The idea is one can shrink the time to deploy if instead of starting with the most generic box and adding, you start somewhere in the middle, adding common things to a 'golden' image, and then deploying on top of that.

In the use-case for the OpenStack Cookbook? Well, we just needed a manila Ubuntu 14.10 box for Fusion and Virtualbox. As there are three of us on the project now, having a common starting ground is ideal.

## Our Packer Setup

I'll leave installing Packer as an exercise for the reader. The [docs](https://packer.io/docs) are pretty good in the regard.

For our build, we use the following packer.json:

```
{
  "builders": [
    {"type": "virtualbox-iso",
    "guest_os_type": "Ubuntu_64",
    "iso_url": "http://mirror.anl.gov/pub/ubuntu-iso/CDs/utopic/ubuntu-14.10-server-amd64.iso",
    "iso_checksum": "91bd1cfba65417bfa04567e4f64b5c55",
    "iso_checksum_type":"md5",
    "http_directory": "preseed",
    "ssh_username": "vagrant",
    "ssh_password": "vagrant",
    "boot_wait":"5s",
    "output_directory": "ubuntu64_basebox_virtualbox",
    "shutdown_command": "echo 'shutdown -P now' > shutdown.sh; echo 'vagrant'|sudo -S sh 'shutdown.sh'",
    "boot_command": [
      "<esc><esc><enter><wait>",
      "/install/vmlinuz noapic ",
      "ks=http://{{ .HTTPIP }}:{{ .HTTPPort }}/preseed.cfg ",
      "debian-installer=en_US auto locale=en_US kbd-chooser/method=us ",
      "hostname={{ .Name }} ",
      "fb=false debconf/frontend=noninteractive ",
      "keyboard-configuration/modelcode=SKIP keyboard-configuration/layout=USA ",
      "keyboard-configuration/variant=USA console-setup/ask_detect=false ",
      "initrd=/install/initrd.gz -- ",
      "<enter>"]
    },
    {"type": "vmware-iso",
    "guest_os_type": "ubuntu-64",
    "iso_url": "http://mirror.anl.gov/pub/ubuntu-iso/CDs/utopic/ubuntu-14.10-server-amd64.iso",
    "iso_checksum": "91bd1cfba65417bfa04567e4f64b5c55",
    "iso_checksum_type":"md5",
    "http_directory": "preseed",
    "ssh_username": "vagrant",
    "ssh_password": "vagrant",
    "boot_wait":"5s",
    "output_directory": "ubuntu64_basebox_vmware",
    "shutdown_command": "echo 'shutdown -P now' > shutdown.sh; echo 'vagrant'|sudo -S sh 'shutdown.sh'",
    "boot_command": [
      "<esc><esc><enter><wait>",
      "/install/vmlinuz noapic ",
      "ks=http://{{ .HTTPIP }}:{{ .HTTPPort }}/preseed.cfg ",
      "debian-installer=en_US auto locale=en_US kbd-chooser/method=us ",
      "hostname={{ .Name }} ",
      "fb=false debconf/frontend=noninteractive ",
      "keyboard-configuration/modelcode=SKIP keyboard-configuration/layout=USA ",
      "keyboard-configuration/variant=USA console-setup/ask_detect=false ",
      "initrd=/install/initrd.gz -- ",
      "<enter>"]
    }],
  "provisioners": [{
    "type": "shell",
    "execute_command": "echo 'vagrant' | sudo -S sh '{{ .Path }}'",
    "inline": [
      "apt-get update -y",
      "apt-get install -y linux-headers-$(uname -r) build-essential dkms",
      "apt-get clean",
      "mount -o loop VBoxGuestAdditions.iso /media/cdrom",
      "sh /media/cdrom/VBoxLinuxAdditions.run",
      "umount /media/cdrom",
      "mkdir /home/vagrant/.ssh",
      "mkdir /root/.ssh",
      "wget -qO- https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub >> ~/.ssh/authorized_keys",
      "echo 'vagrant ALL=NOPASSWD:ALL' > /tmp/vagrant",
      "chmod 0440 /tmp/vagrant",
      "mv /tmp/vagrant /etc/sudoers.d/"
    ]}],
  "post-processors": [
  {
      "type": "vagrant",
      "override": {
        "virtualbox": {
          "output": "utopic-x64-virtualbox.box"
        },
        "vmware": {
          "output": "utopic-x64-vmware.box"
        }
      }
    }
  ]
}
```

This in turn calls a kickstart (yes kickstart on Ubuntu) file stored in the preseed folder. This looks like:

```
install
text

cdrom

lang en_US.UTF-8
keyboard us

network --device eth0 --bootproto dhcp

timezone --utc America/Chicago

zerombr
clearpart --all --initlabel
bootloader --location=mbr

part /boot --fstype=ext3 --size=256 --asprimary
part pv.01 --size=1024 --grow --asprimary
volgroup vg_root pv.01
logvol swap --fstype swap --name=lv_swap --vgname=vg_root --size=1024
logvol / --fstype=ext4 --name=lv_root --vgname=vg_root --size=1024 --grow

auth --enableshadow --enablemd5

# rootpw is vagrant
rootpw --iscrypted $1$dUDXSoA9$/bEOTiK9rmsVgccsYir8W0
user --disabled

firewall --disabled

skipx

reboot

%packages
ubuntu-minimal
openssh-server
openssh-client
wget
curl
git
man
vim
ntp

%post
apt-get update

apt-get upgrade -y linux-generic

update-grub

useradd -m -s /bin/bash vagrant
echo vagrant:vagrant | chpasswd

mkdir -m 0700 -p /home/vagrant/.ssh

curl https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub >> /home/vagrant/.ssh/authorized_keys

chmod 600 /home/vagrant/.ssh/authorized_keys
chown -R vagrant:vagrant /home/vagrant/.ssh

echo "vagrant ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

sed -i 's/^PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config

apt-get clean

rm -rf /tmp/*

rm -f /var/log/wtmp /var/log/btmp

history -c
```

Once both files are in place and packer is installed, building the images is as simple as:

```packer build packer.json```

This will take a *long* time, but it does eventually get there, and will produce two .box files in the output folders you specified in the packer.json. From there, you can upload them somewhere and have vagrantcloud make them available, as we have under ```bunchc/utopic-x64```.

## Summary

In this post we looked at how to use packer.io to build Vagrant boxes for both VMware Workstation / Fusion and Virtualbox.