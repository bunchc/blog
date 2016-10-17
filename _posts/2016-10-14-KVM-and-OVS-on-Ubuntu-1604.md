---
title: "KVM and OVS on Ubuntu 16.04"
date: 2016-10-14
layout: "post"
categories: KVM, Virtualization, OVS, Open vSwitch, cloud
---

Lately I've been digging a bit deeper into what actually goes on behind the scenes when it comes to cloudy things. Specifically, I wanted to get a virtual machine booted with KVM and connected to the network using OVS. This was made difficult as the posts describing how to do this are either out of date or make knowledge assumptions that could throw a beginner off.

What follows then, are my notes on how I am currently preforming this on Ubuntu 16.04 with OVS.

## Install KVM

First up, we install KVM:

```
# Update and install the needed packages
PACKAGES="qemu-kvm libvirt-bin bridge-utils virtinst"
sudo apt-get update
sudo apt-get dist-upgrade -qy

sudo apt-get install -qy ${PACKAGES}

# add our current user to the right groups
sudo adduser `id -un` libvirtd
sudo adduser `id -un` kvm
```

The above code block first defines a list of packages we need. Next, it updates the your Ubuntu package cache, the installed system. Then it installs the packages needed to operate KVM: libvirt-bin, qemu-kvm, bridge-utils, virtinst. In order these packages handle: virtualization, management, networking, and ease of use.

The final two commands add our current user to the groups needed to operate KVM.

## Install OVS

Next up, we install and configure OVS for use with KVM.

First we need to remove the default network to prevent conflicts down the road.
```
sudo virsh net-destroy default
sudo virsh net-autostart --disable default
```

Next up, we install the OVS packages and start the service:
```
sudo apt-get install -qy openvswitch-switch openvswitch-common
sudo service openvswitch-switch start
```

Now that OVS is installed and started, it is time to start configuring things. The below code block:
- enables ipv4 forwarding
- creates and ovs bridge
- adds eth2 to the bridge

```
sudo echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sudo sysctl -p /etc/sysctl.conf

sudo ovs-vsctl add-br br0
sudo ovs-vsctl add-port br0 eth2
```

Our last task then, is to reconfigure the network interfaces, moving the IP that used to reside on eth2 to br0. To do that, your ```/etc/network/interfaces``` file should look similar to this:

> **Note:** What follows only contains the two sections of the interfaces file we need for this example.

```
# Set eth2 for bridging
auto eth2
iface eth2 inet manual

# The OVS bridge interface
auto br0
iface br0 inet static
  address 172.16.2.101
  netmask 255.255.255.0
  bridge_ports eth2
  bridge_fd 9
  bridge_hello 2
  bridge_maxage 12
  bridge_stp off
```

Finally, restart networking:

```
sudo ifdown --force -a && sudo ifup --force -a
```

## Booting your first VM

Now that we have OVS and KVM installed, it's time to boot our first VM. We do that by using the ```virt-install``` command. The virt-install command itself has like, ALL of the parameters, and we'll discuss what they do after the code block:

> **Note:** You should have DHCP running somewhere on the network that you have bridged to.

```
sudo virt-install -n demo01 -r 256 --vcpus 1 \
    --location "http://us.archive.ubuntu.com/ubuntu/dists/xenial/main/installer-i386/" \
    --os-type linux \
    --os-variant ubuntu16.04 \
    --network bridge=br0,virtualport_type='ovs' \
    --graphics vnc \
    --hvm --virt-type kvm \
    --disk size=8,path=/var/lib/libvirt/images/test01.img \
    --noautoconsole \
    --extra-args 'console=ttyS0,115200n8 serial '
```

Like I said, a pretty involved command. We'll break down the more interesting or non-obvious parameters:

-r is memory in MB
--location is the URL that contains the initrd image to boot linux
--os-variant for this one, consult the man page for virt-install to get the right name.
--network - This one took me the longest to sort out. That is, bridge=br0 was straight forward, but knowing to set virtualport_type to OVS took looking at the man page more times than I would like to admit.
--hvm and --virt-type kvm - These values tell virt-install to create a vm that will run on a hypervisor, and that the hypervisor of choice is KVM.
--disk - the two values here, size and path are straight forward. Disk size in GB and the path to where it should create the file.

Once you execute this command, virt-install will create an XML that represents the VM, boot it, and attempt to install an operating system. This will leave you a prompt to attach to the VM and finish the installation.

## Attaching to the console

When creating the VM we supplied ```--extra-args 'console=ttyS0,115200n8 serial '```. This tells virsh to supply said parameters to the VMs boot sequence, which supplies us with a serial console.

To attach to the console and finish our installation we need to first get the ID of the VM we're working with, then attach to it. You get the VM ID by running ```virsh list --all```:

```
$ sudo virsh list --all
 Id    Name                           State
----------------------------------------------------
 1     demo01                         running
```

To attach to the console: ```sudo virsh console 1``` should bring up the ubuntu installer:

```

  ┌──────────────────────┤ [!!] Select your location ├──────────────────────┐
  │                                                                         │
  │ The selected location will be used to set your time zone and also for   │
  │ example to help select the system locale. Normally this should be the   │
  │ country where you live.                                                 │
  │                                                                         │
  │ This is a shortlist of locations based on the language you selected.    │
  │ Choose "other" if your location is not listed.                          │
  │                                                                         │
  │ Country, territory or area:                                             │
  │                                                                         │
  │                         Nigeria                                         │
  │                         Philippines          ▒                          │
  │                         Singapore            ▒                          │
  │                         South Africa                                    │
  │                         United Kingdom       ▒                          │
  │                         United States                                   │
  │                                                                         │
  │     <Go Back>                                                           │
  │                                                                         │
  └─────────────────────────────────────────────────────────────────────────┘

<Tab> moves; <Space> selects; <Enter> activates buttons
```


# Resources

There were so, so many blogs used as a staring point for this that I'm sure I've missed one or two, but here we go:
- [http://blog.johngoulah.com/tag/virt-install/](http://blog.johngoulah.com/tag/virt-install/)
- [http://blog.scottlowe.org/2015/05/20/fully-automated-ubuntu-install/](http://blog.scottlowe.org/2015/05/20/fully-automated-ubuntu-install/)
- [https://raymii.org/s/articles/virt-install_introduction_and_copy_paste_distro_install_commands.html](https://raymii.org/s/articles/virt-install_introduction_and_copy_paste_distro_install_commands.html)
- [http://therandomsecurityguy.com/openvswitch-cheat-sheet/](http://therandomsecurityguy.com/openvswitch-cheat-sheet/)
- [http://blog.scottlowe.org/2012/08/17/installing-kvm-and-open-vswitch-on-ubuntu/](http://blog.scottlowe.org/2012/08/17/installing-kvm-and-open-vswitch-on-ubuntu/)
