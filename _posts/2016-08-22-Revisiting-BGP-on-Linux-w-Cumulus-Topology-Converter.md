---
title: "Revisiting BGP on Linux w/ Cumulus Topology Converter"
date: 2016-08-22
layout: "post"
categories: BGP, Spine Leaf, Networking
---

A post or two ago, we explored [setting up BGP to route between networks using Linux and Cumulus Quagga](http://blog.codybunch.com/2016/07/12/Getting-started-with-BGP-on-Linux-with-Cumuls-Quagga/). As fun as this exercise was, I would not want to have to set it up this way each and every time I needed a lab. To that point, I haven't actually touched the homelab it was built on since that point. Why? because complicated.

Then enters Scott Lowe, and [Technology Short Take #70](http://blog.scottlowe.org/2016/08/12/technology-short-take-70/). At the bottom of the networking section, as if it weren't the coolest thing on the list, there is this gem: 

    "This looks neat—I need to try to find time to give it a spin."

So that's what we're doing!

Topology converter's job, is to take a formatted text file that represents the lab you'd like built, and to generate it as a Vagrantfile that can then be spun up with all of the plumbing taken care of for you. 

## Getting Started

My lab setup is a 2012 15" Retina MacBook, 16bg ram, and the latest Vagrant (1.8.5) & Virtualbox (5.1.4). From there we start with the installation instructions found [here.](https://github.com/CumulusNetworks/topology_converter/tree/master/documentation#installation). Additionally, you'll want to clone the repo: ```git clone https://github.com/CumulusNetworks/topology_converter && cd topology_converter```

You'll also want to install the vagrant-cumulus plugin: ```vagrant plugin install vagrant-cumulus```

## Remaking our BGP Lab

Now that we've got the tools installed, we need to create our topology. Remember, we used this last time:

![Topology](https://i.imgur.com/MDit7Wr.png)

In the language of topology converter, that looks like this:

```
graph dc1 {
  "spine01" [function="spine" os="CumulusCommunity/cumulus-vx" version="3.0.1" memory="512" config="./helper_scripts/extra_switch_config.sh" mgmt_ip="172.10.100.1"]
  "spine02" [function="spine" os="CumulusCommunity/cumulus-vx" version="3.0.1" memory="512" config="./helper_scripts/extra_switch_config.sh" mgmt_ip="172.20.100.1"]
  "spine03" [function="spine" os="CumulusCommunity/cumulus-vx" version="3.0.1" memory="512" config="./helper_scripts/extra_switch_config.sh" mgmt_ip="172.30.100.1"]
  "leaf01" [function="leaf" os="CumulusCommunity/cumulus-vx" version="3.0.1" memory="512" config="./helper_scripts/extra_switch_config.sh" mgmt_ip="172.10.100.2"]
  "leaf02" [function="leaf" os="CumulusCommunity/cumulus-vx" version="3.0.1" memory="512" config="./helper_scripts/extra_switch_config.sh" mgmt_ip="172.20.100.2"]
  "leaf03" [function="leaf" os="CumulusCommunity/cumulus-vx" version="3.0.1" memory="512" config="./helper_scripts/extra_switch_config.sh" mgmt_ip="172.30.100.2"]
  "server01" [function="host" os="boxcutter/ubuntu1404" memory="512" ubuntu=True config="./helper_scripts/extra_server_config.sh" mgmt_ip="172.10.100.3"]
  "server02" [function="host" os="boxcutter/ubuntu1404" memory="512" ubuntu=True config="./helper_scripts/extra_server_config.sh" mgmt_ip="172.20.100.3"]
  "server03" [function="host" os="boxcutter/ubuntu1404" memory="512" ubuntu=True config="./helper_scripts/extra_server_config.sh" mgmt_ip="172.30.100.3"]

  "leaf01":"swp40" -- "spine01":"swp1"
  "leaf02":"swp40" -- "spine02":"swp1"
  "leaf03":"swp40" -- "spine03":"swp1"

  "spine01":"swp31" -- "spine02":"swp31"
  "spine02":"swp32" -- "spine03":"swp32"
  "spine03":"swp33" -- "spine01":"swp33"

  "server01":"eth0" -- "leaf01":"swp1"
  "server02":"eth0" -- "leaf02":"swp1"
  "server03":"eth0" -- "leaf03":"swp1"
}
```

There are four sections to this file. The first is the node definitions. Spine, leaf, and server. Or, routers, switches, and hosts. Each is given an OS to run and a management IP.

The next three sections define the connections between the nodes. In our case, interface 40 on the switches uplinks to their router. The routers each link to one another, and the hosts to the switches.

Once you have this file saved as bgplab.dot (or similar), run the topology converter command (which produces a very verbose bit of output):

```
$ python ./topology_converter.py ./bgplab.dot

######################################
          Topology Converter
######################################
>> DEVICE: spine02
     code: CumulusCommunity/cumulus-vx
     memory: 512
     function: spine
     mgmt_ip: 172.20.100.1
     config: ./helper_scripts/config_switch.sh
     hostname: spine02
     version: 3.0.1
       LINK: swp1
               remote_device: leaf02
               mac: 44:38:39:00:00:04
               network: net2
               remote_interface: swp40
       LINK: swp31
               remote_device: spine01
               mac: 44:38:39:00:00:10
               network: net8
               remote_interface: swp31
       LINK: swp32
               remote_device: spine03
               mac: 44:38:39:00:00:0d
               network: net7
               remote_interface: swp32
...
```

## Starting the lab

Ok, so what we've done so far was install topology converter, write a topology file to represent our lab, and finally, we converted that to a Vagrant file. We have one more pre-flight check to run before we can fire up the lab, and that is to make sure Vagrant recognizes what we're trying to do:

```
$ vagrant status
Current machine states:

spine02                   not created (vmware_fusion)
spine03                   not created (vmware_fusion)
spine01                   not created (vmware_fusion)
leaf02                    not created (vmware_fusion)
leaf03                    not created (vmware_fusion)
leaf01                    not created (vmware_fusion)
server01                  not created (vmware_fusion)
server03                  not created (vmware_fusion)
server02                  not created (vmware_fusion)
```

Looks good, excepting that vmware_fusion bit. Thankfully, that's an artifact of my local environment, and can be worked around by specifying ```--provider=virtualbox``` 

So, let's do that:
**Warning, this will take a WHILE**

```
vagrant up --provider=virtualbox


LOTS OF OUTPUT

$ vagrant status
Current machine states:

spine02                   running (virtualbox)
spine03                   running (virtualbox)
spine01                   running (virtualbox)
leaf02                    running (virtualbox)
leaf03                    running (virtualbox)
leaf01                    running (virtualbox)
server01                  running (virtualbox)
server03                  running (virtualbox)
server02                  running (virtualbox)
```

## Accessing the lab

Ok, to recap one more time, we've installed topology converter, written a topology file, converted that to a Vagrant environment, and fired that up within virtualbox. That's A LOT of work, so if you need a coffee, I understand.

While we've done all the work of creating the lab, we haven't configured anything as yet. While we'll leave that as an exercise for another post, we will show you how to access a node in said lab. The most straight forward way, is with ```vagrant ssh [nodename]```

```
vagrant ssh spine02

Welcome to Cumulus VX (TM)

Cumulus VX (TM) is a community supported virtual appliance designed for
experiencing, testing and prototyping Cumulus Networks' latest technology.
For any questions or technical support, visit our community site at:
http://community.cumulusnetworks.com

The registered trademark Linux (R) is used pursuant to a sublicense from LMI,
the exclusive licensee of Linus Torvalds, owner of the mark on a world-wide
basis.
vagrant@spine02:~$ sudo su - cumulus
```

# Summary

Long post is long. However, in this post, we have used Cumulus Topology converter to create a lab network topology with 3 routers, 3 switches, and 3 hosts that are now waiting for configuration.
