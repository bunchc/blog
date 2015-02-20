---
title: "Raspberry Pi as PXE Server"
date: 2014-08-19
categories: 
---

As I start to move from OpenStack Compute Cells in Devstack to OpenStack Compute Cells physicall, I needed to re-think my homelab some. It had been running some variation of vsphere and the OpenStack Cookbook work from various projects prior. Basically, it was a Hodor of a home lab.

## Hodor!

Rather than lose a bit of hardware to foreman, or one of the new razor forks (I may still go this route later), I decided instead to pound a Raspberry Pi into service. To turn the rPI into a usable provisioning server, I did the following:

1. Provision a 16gb card with Raspbian
2. Beat Networking Into Submission
3. Setup IP Tables for Nat
4. Configure DHCP, PXE, TFTP, etc

We'll skip step 1 as the folks at [raspberrypi.org](http://www.raspberrypi.org/) cover that pretty well already.

## Beat Networking Into Submission

So the rPI does some auto-hotplugging bits that can cause you some issues if you try to use both wifi and ethernet at the same time. If you're not expecting them, well... let's just say I spent too long trying to solve before googling the problem. Here's what my networking config files look like:

```
# cat /etc/network/interfaces
auto lo
iface lo inet loopback

auth eth0
allow-hotplug eth0
iface eth0 inet static
    address 172.16.0.1
    netmask 255.255.255.0

auto wlan0
allow-hotplug wlan0
iface wlan0 inet static
wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
    address 10.0.1.15
    netmask 255.255.255.0
    broadcast 10.0.1.255
    gateway 10.0.1.1

iface default inet dhcp

# cat /etc/default/ifplugd
INTERFACES="eth0"
HOTPLUG_INTERFACES="eth0"
ARGS="-q -f -u0 -d10 -w -I"
SUSPEND_ACTION="stop"
```

Once you have that, reboot the rPI and you should be good to go.

## Setup IP Tables for NAT

This is also pretty well straight forward, but for some reason I end up googling it each time. Here it is for reference:

```
# NAT
iptables --table nat --append POSTROUTING --out-interface wlan0 -j MASQUERADE
iptables --append FORWARD --in-interface eth0 -j ACCEPT
iptables-save | sudo tee /etc/iptables.conf
iptables-restore < /etc/iptables.conf
sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
sed -i "s/exit 0/iptables-restore < \/etc\/\iptables.conf \nexit 0/g" /etc/rc.local
sed -i "s/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g" /etc/sysctl.conf
```

## Configure DHCP, PXE, TFTP, etc

This one is a bit more involved, but is all done via the following bash commands:

```
MY_IP=$(ifconfig eth0 | awk '/inet addr/ {split ($2,A,":"); print A[2]}')

# Install the things
sudo apt-get install -y dnsmasq nfs-kernel-server syslinux-common

# Setup some shell folders
sudo mkdir -p /tftpboot/images/ubuntu/14.04/amd64
sudo cp -r /usr/lib/syslinux/* /tftpboot/
sudo mkdir -p /tftpboot/pxelinux.cfg/Ubuntu
sudo cp /usr/lib/syslinux/vesamenu.c32 /tftpboot/

# Create a pxe.conf file
sudo cat > /tftpboot/pxelinux.cfg/pxe.conf <<EOF
MENU TITLE  PXE Server 
NOESCAPE 1
ALLOWOPTIONS 1
PROMPT 0
MENU WIDTH 80
MENU ROWS 14
MENU TABMSGROW 24
MENU MARGIN 10
MENU COLOR border               30;44      #ffffffff #00000000 std
EOF

# Create our PXE Menu
sudo cat > /tftpboot/pxelinux.cfg/default <<EOF
DEFAULT vesamenu.c32 
TIMEOUT 600
ONTIMEOUT BootLocal
PROMPT 0
MENU INCLUDE pxelinux.cfg/pxe.conf
NOESCAPE 1
LABEL BootLocal
        localboot 0
        TEXT HELP
        Boot to local hard disk
        ENDTEXT
MENU BEGIN Ubuntu
MENU TITLE Ubuntu 
        LABEL Previous
        MENU LABEL Previous Menu
        TEXT HELP
        Return to previous menu
        ENDTEXT
        MENU EXIT
        MENU SEPARATOR
        MENU INCLUDE Ubuntu/Ubuntu.menu
MENU END
EOF

sudo cat > /tftpboot/pxelinux.cfg/Ubuntu/Ubuntu.menu <<EOF
LABEL 2
        MENU LABEL Ubuntu 14.04 (64-bit)
        kernel tftp://$MY_IP/images/ubuntu/14.04/amd64/install/netboot/ubuntu-installer/amd64/linux
        append auto=true priority=critical vga=788 initrd=tftp://$MY_IP/images/ubuntu/14.04/amd64/install/netboot/ubuntu-installer/amd64/initrd.gz locale=en_US.UTF-8 kbd-chooser/method=us netcfg/choose_interface=auto url=tftp://172.16.11.250/preseed.cfg
        TEXT HELP
        Boot the Ubuntu 14.04 64-bit DVD
        ENDTEXT
EOF

# Configure dnsmasq for tftp & dhcp
sudo cat >> /etc/dnsmasq.conf <<EOF
server=$MY_IP@eth0
interface=eth0
no-dhcp-interface=wlan0
dhcp-range=172.16.0.10,172.16.0.253,12h
dhcp-boot=pxelinux.0
pxe-service=x86PC,"Booting from Network...",pxelinux
enable-tftp
tftp-root=/tftpboot
dhcp-boot=pxelinux.0,servername,$MY_IP
EOF
sudo service dnsmasq restart

# Get 14.04 and extract the needful
wget -O ~/ubuntu-14.04-server-amd64.iso http://mirror.anl.gov/pub/ubuntu-iso/CDs/trusty/ubuntu-14.04.1-server-amd64.iso

sudo mkdir /mnt/loop
sudo mount -o loop -t iso9660 ~/ubuntu-14.04-server-amd64.iso /mnt/loop
sudo cp -R /mnt/loop/* /tftpboot/images/ubuntu/14.04/amd64
sudo cp -R /mnt/loop/.disk /tftpboot/images/ubuntu/14.04/amd64
sudo umount /mnt/loop

# Get a generic preseed
wget -O /tftpboot/preseed.cfg https://help.ubuntu.com/lts/installation-guide/example-preseed.txt

# Setup NFS mounts
echo "
/tftpboot/images/ubuntu/    *(ro,sync,no_subtree_check)" | sudo tee -a /etc/exports

# Enable RPCBind, NFS, and restart them
update-rc.d rpcbind enable && update-rc.d nfs-common enable
service rpcbind start
service nfs-kernel-server restart
```

The commands themselves are commented pretty well. Generically what they do is install dnsmasq for dhcp & tftp. From there we download and extract all the things from the ISO that we'll need, configure some menus, download a basic preseed, and restart some services.

## Summary

In this post, we took a lowly Raspberry Pi and indentured it into some network servitude as a pxe / tftp server. We did this by installing and configuring dnsmasq for tftp and dhcp. Additionally we set up some fancy pxeboot menus and configured them to boot locally as a priority and to the network in times of need. Finally, we pulled down an Ubuntu 14.04 image and generic preseed file to use for automatic installs.

## Resources
- [http://raspberrypi.stackexchange.com/questions/8851/setting-up-wifi-and-ethernet](http://raspberrypi.stackexchange.com/questions/8851/setting-up-wifi-and-ethernet)
- [http://it-joe.com/howtos/pxe.php](http://it-joe.com/howtos/pxe.php)
- [http://wernerstrydom.com/2014/05/25/automatically-installing-ubuntu-server-14-04-tftp/](http://wernerstrydom.com/2014/05/25/automatically-installing-ubuntu-server-14-04-tftp/)