---
title: "[Lab] Automated Install of ESXi with netboot.xyz"
date: 2021-05-04
layout: "post"
categories: "lab, automation, vmware, vsphere, esx, esxi, pxe, netboot"
---

Earlier this year, Jake Robinson tweeted about doing a manual ESXi install, and based on some replies to it I shared that I automate ESXi host installation in my lab using [netboot.xyz](https://netboot.xyz/). I then promised a writeup and then procrastinated on getting that done. No longer, it is time!

## Requirements

You will need a number of things in order for this to work. Here is a list:

* A TFTP server to serve netboot.xyz ipxe files from
* Ability to set options on your DHCP server
  * 66 (TFTP Server IP)
  * 67 (Network boot program)
* An HTTP server to serve
  * netboot.xyz & custom menus
  * ESXi installer files
* Access to ESXi install media
## Getting Started

There are a few moving parts here, that we'll tackle in the following order:

* Extracting ESXi files
* Customizing ESXi `boot.cfg`
* Customizing ESXi Kickstart
* Installing netboot.xyz
* Adding a custom menu to netboot.xyz

### Extracting ESXi files

The first thing we need to do is make the ESXi install files available over http so your hosts can install them. The following steps cover this. It is also important to remember the file names are case sensitive, and some utilities will change them all to upper or lower case when extracting.

1. Download the ESXi installer ISO.
2. Mount the image (the steps on how to do this will vary by system).
3. Copy all of the files to a folder on your webserver. e.g. `web/VMware/ESXi/v7.0.0U2/`
4. Ensure the extracted files are lowercase.
### Customizing ESXi `boot.cfg`

Next, make a copy of the `boot.cfg` file from the extracted ESXi installer. You will need to make the following modifications:

* Change the `prefix=` line to point to the location where you extracted the installer to. e.g. `prefix=http://boot.codybunch.lab/VMware/ESXi/v7.0.0U2`
* **[Optional]** If using a kickstart file, add that to the end of the `prefix=` line. `ks=http://boot.codybunch.lab/VMware/ESXi/kickstart.cfg`
* Remove `cdromBoot` from the `kernelopt=` line.
  * Depending on your hardware, you may need to add other options here, e.g. `allowLegacyCPU=TRUE`
* Remove the forward slash `/` from the file names in `kernel=` and `modules=` lines.
  * `kernel=/b.b00` changes to `kernel=b.b00`
  * `modules=/jumpstart.gz --- /useropts.gz ...` changes to `modules=jumpstart.gz --- useropts.gz ...`
### [Optional] Customizing ESXi Kickstart

Using a Kickstart file can automate the installation to where it is non interactive. My kickstart file follows. You can find various options for this file in the [Resources](#resources) section.

<script src="https://gist.github.com/bunchc/a926b222e0df48a364d8485c49a58dcb.js"></script>

### Installing netboot.xyz

Installing netboot.xyz can be done by following the [self hosting instructions in the netboot.xyz documentation.](https://netboot.xyz/selfhosting/) Paying special attention to the custom options section [here.](https://netboot.xyz/selfhosting/#self-hosted-custom-options)

In order to boot from netboot.xyz, you need to set DHCP options 66 and 67 as discussed earlier. Specifically, set option 66 or `next-server` to the IP address of your TFTP server, and option 67 or `boot-file` to the netboot.xyz. The following diagram shows what this looks like for my Ubiquiti homelab setup:

<img src="https://i.imgur.com/Zu01vgP.png">

### Adding a custom menu to netboot.xyz

Per the ['Self Hosted Custom Options'](https://netboot.xyz/selfhosting/#self-hosted-custom-options) section in the netboot.xyz docs, you can add custom menu structures by copying and editing their sample menu:

`cp etc/netbootxyz/custom /etc/netbootxyz/custom`

The menu file I use is as follows:

<script src="https://gist.github.com/bunchc/c5249f11dd9f046597e4efc5ffef4a3c.js"></script>

There are a few interesting bits I would like to call out:

* Line 19 uses the `sanboot` option to stream the ESXi ISO to the host. This allows you to walk the installation as if you had booted directly from the ISO.
* Line 23 loads the customized `boot.cfg` we created earlier in the post without loading a kickstart. This allows you to walk the installer by hand.
* Line 27 loads the kickstart file, and performs the installation for you.
* The lines (20, 24, 28) that read `boot || goto custom_exit` tell ipxe to attempt to boot the host, or (`||`) if that fails, to exit the custom menu and return to the main menu.
## Summary

netboot.xyz allows for a good deal of flexibility when running a home lab. Both for rapidly changing the OS on your physical hosts as well as having installers available for any number of other operating systems and utilities available via network boot. You can also add custom options to install things like ESXi, as was shown in this post.
