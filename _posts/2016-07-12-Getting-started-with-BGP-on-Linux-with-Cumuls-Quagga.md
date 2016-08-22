---
title: "Getting started with BGP on Linux with Cumuls Quagga"
date: 2016-07-12
layout: "post"
categories: networking, routing, bgp
---

Before we begin, I'd like to admit something: "I spent a good number of years as a Windows sysadmin." There, I said it. Now, I'm proud of my time as a Windows guy, so that's' not why I bring it up. I mention my time as a Windows admin perhaps to provide a reason why it took me so long to get BGP between some Linux VMs.

# What we're building

![](https://i.imgur.com/MDit7Wr.png)

The above figure depicts a three node, three router network that we will create in this post. Following these steps:

1. Install the routers
2. Configure the routers
3. Configure the hosts
4. Test connectivity

## Install the Routers

In our setup we start with Ubuntu 14.04 boxes with two interfaces. We assume eth0 is on the 10.0.1.0/24 network, and eth1 is on the 172.x.100.0/24 network.

First download the 14.04 package from [here](https://cumulusnetworks.com/routing-on-the-host/download/thanks/ubuntu1404/). In the commands below wget this file from another server in the environment, you can get it there however you like.

### Configure the interfaces

On each router, configure your interfaces, substituting network addresses where relevant:

```
# cat /etc/network/interfaces

# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet static
  address 10.0.1.19/24
  gateway 10.0.1.1
  dns-nameservers 4.2.2.2 8.8.8.8

auto eth1
iface eth1 inet static
  address 172.10.100.1
  netmask 255.255.255.0
```

Then, restart the interfaces:
```
sudo service networking restart
```

### Enable routing

```
sudo echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sudo sysctl -p

```

### Install Quagga

As root, run the following commands:

```
apt-get update && apt-get -qq dist-upgrade && reboot
wget http://location/of/the/download/roh-ubuntu1404.tar
tar -xvf roh-ubuntu1404.tar
cd ubuntu1404/
dpkg -i quagga_0.99.23.1-1+cl3u2_trusty_amd64.deb duagga-dbg_0.99.23.1-1+cl3u2_trusty_amd64.deb quagga-doc_0.99.23.1-1+cl3u2_trusty_all.deb
```

We then need to provide some initial start up configuration for Quagga. We do this by changing the /etc/quagga/daemons file to reflect the services we would like started. In this case, the obligatory Zebra, and BGP:

```
zebra=yes
bgpd=yes
ospfd=no
ospf6d=no
ripd=no
ripngd=no
isisd=no
```

### Start Quagga

Now that Quagga is installed, we need to start said service:

```
service quagga start
```

## Configure the Routers

with Quagga installed and running on each Quagga host, we can configure the needed bits to get BGP operational. The commands and configuration that follow configure the 10.0.1.19/24 router from our diagram above. Note, you will want to choose Private AS numbers from here: [Private BGP AS Numbers](https://tools.ietf.org/html/rfc6996). In the example, these are 64512, 513, and 514 respectively.

```
# vtysh
conf t

hostname cab1-router
password zebra
enable password zebra
log file /var/log/quagga/bgpd.log
!
router bgp 64512
 bgp router-id 10.0.1.19
 network 172.10.100.0/24
 neighbor 10.0.1.20 remote-as 64513
 neighbor 10.0.1.21 remote-as 64514
 distance bgp 150 150 150
!
exit
exit
wr mem
```

## Configure the Hosts

For each host behind out of our Quagga routers, you need to set the interfaces file to an appropriate address and gateway per host:

```
$ cat /etc/network/interfaces
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet static
  address 172.30.100.100/24
  gateway 172.30.100.1
```

## Testing Connectivity

Now that all three routers and all three VMs are configured, we will test connectivity. First between routers, then between VMs:

From router 1:
```
cab1-router# ping 172.20.100.1
PING 172.20.100.1 (172.20.100.1) 56(84) bytes of data.
64 bytes from 172.20.100.1: icmp_seq=1 ttl=64 time=0.367 ms
^C
--- 172.20.100.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.367/0.367/0.367/0.000 ms
cab1-router# ping 172.20.100.100
PING 172.20.100.100 (172.20.100.100) 56(84) bytes of data.
64 bytes from 172.20.100.100: icmp_seq=1 ttl=63 time=0.448 ms
^C
--- 172.20.100.100 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.298/0.373/0.448/0.075 ms
```

From VM3:

```
[root@cab3-test-vm:~]
[17:03:59] $ ping 172.20.100.100
PING 172.20.100.100 (172.20.100.100) 56(84) bytes of data.
64 bytes from 172.20.100.100: icmp_seq=1 ttl=62 time=0.443 ms
64 bytes from 172.20.100.100: icmp_seq=2 ttl=62 time=0.475 ms
^C
--- 172.20.100.100 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.443/0.459/0.475/0.016 ms

[root@cab3-test-vm:~]
[17:10:06] $ ping 172.10.100.1
PING 172.10.100.1 (172.10.100.1) 56(84) bytes of data.
64 bytes from 172.10.100.1: icmp_seq=1 ttl=63 time=0.179 ms
64 bytes from 172.10.100.1: icmp_seq=2 ttl=63 time=0.410 ms
^C
--- 172.10.100.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.179/0.294/0.410/0.116 ms
```

# Summary

In this post we built three L3 networks, and configured BGP routing between them using Quagga on a Ubuntu 14.04 host. We then tested connectivity between these networks.

# References
- [Private BGP AS Numbers](https://tools.ietf.org/html/rfc6996)
- [Static IP on Ubuntu](http://www.sudo-juice.com/how-to-set-a-static-ip-in-ubuntu-the-proper-way/)
- [BGP Config](https://wiki.quagga.net/wiki/index.php/ConfigurationExamples)
- [BGP on the host?](https://support.cumulusnetworks.com/hc/en-us/articles/216805858-Routing-on-the-Host-An-Introduction)
- [Installing Quagga](https://docs.cumulusnetworks.com/display/ROH/Installing+the+Cumulus+Quagga+Package+on+a+Host+Server#InstallingtheCumulusQuaggaPackageonaHostServer-InstallingtheCumulusQuaggaPackageonaServerHost)
