---
title: "ASIC Mining on Raspberry Pi"
date: 2018-03-12
layout: "post"
categories: bitcoin, Raspberry Pi, rPi, asic
---

I've had some pretty terribad ideas in the past. Not the least of which is [OpenStack Swift on USB Keys](https://vbrownbag.com/2013/05/openstack-swift-raspberry-pi-23-usb-keys-aka-ghettosan-v2/), or the pre-chaos engineering [random VM snapshot deleter](https://vbrownbag.com/2010/07/roll-dice-powercli-fun-with-vmware-snapshots/). In that vein, I bring you ASIC Bitcoin mining on Raspberry Pi.

![Raspberry Pi Mining Cluster](https://i.imgur.com/gohYdnM.jpg)

As you read, keep in mind, that the goal here, like in the afore mentioned posts is not to be practical. Rather this is a "because I can" project. The conclusion of which will be to run the miners inside containers backed by Kubernetes. But, that is for another time.

## The Gear

For this project, I reused my Kubernetes / OpenFaaS cluster, and added some ASICs. Here's a reminder of the parts:

* 7x [Raspberry Pi 2 Model B](https://smile.amazon.com/Raspberry-Pi-Model-Desktop-Linux/dp/B00T2U7R7I/ref=sr_1_3?ie=UTF8&qid=1520887844&sr=8-3&keywords=raspberry+pi+2)
* [Netgear 8-port gig switch](https://smile.amazon.com/gp/product/B00M1C0186/ref=oh_aui_search_detailpage?ie=UTF8&psc=1)
* 7x 1' [Orange ethernet cables](https://smile.amazon.com/InstallerParts-Pack-Ethernet-Cable-Orange/dp/B06XBYGL2T/ref=sr_1_3?ie=UTF8&qid=1520889542&sr=8-3&keywords=1+ft+orange+ethernet+cable)
* [Some plastic stacking cases](https://smile.amazon.com/gp/product/B00MYFAAPO/ref=oh_aui_search_detailpage?ie=UTF8&psc=1)
* 2x [GekoScience ASIC miners](https://smile.amazon.com/gp/product/B016CWBYJK/ref=oh_aui_search_detailpage?ie=UTF8&psc=1)

> Note: Those are Amazon links. I'm not sure if my affiliate account is still active, but if so, this is full disclosure that they may indeed be affiliate links.

## The setup

These are still configured as they were in my [OpenFaaS post](http://blog.codybunch.com/2018/01/05/OpenFaaS-on-Kubernetes-on-Raspberry-Pi/).

## Installing cgminer

For these particular ASICs, one needs to first compile cgminer with the appropriate support. To ensure I can do this again at some point, I wrote an Ansible playbook to do the heavy lifting for me:

<script src="https://gist.github.com/bunchc/940d0621ccf7c183dbfb5315d4665261.js"></script>

For those not familiar with Ansible, here's what it is doing:

* 12-17: Use apt to install prerequisite packages (build-essential, and so on)
* 19-29: Create, and then ensure directories exist for the source and build
* 31-39: Downloads the patched cgminer source with 2PAC support
* 41-59: Runs both the prebuild setup and then compiles cgminer
* 61-65: Configures cgminer
* 67-95: Sets up cgminer to start on boot

This is then installed, sort of like this:

```
ansible-playbook -i inventory.yml playbooks/install_cgminer.yml
```

After a long while (these are raspberry pi's after all), the service is installed, and you are mining:

Service status:

```
pi@node-02:~ $ sudo systemctl status cgminer
● cgminer.service - cgminer
   Loaded: loaded (/etc/systemd/system/cgminer.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2018-02-16 06:49:23 UTC; 3 weeks 3 days ago
 Main PID: 433 (screen)
   Memory: 7.6M
      CPU: 1.471s
   CGroup: /system.slice/cgminer.service
           ├─433 /usr/bin/SCREEN -S cgminer -L -Dm /home/pi/cgminer.sh
           ├─452 /bin/bash /home/pi/cgminer.sh
           └─466 ./cgminer --config /home/pi/cgminer.conf
```

Check in on cgminer itself:

```
# screen -r cgminer

cgminer version 4.10.0 - Started: [2018-03-12 21:10:08.545]
-----------------------------------------------------------------------------------------
 (5s):9.456G (1m):11.10G (5m):8.409G (15m):4.248G (avg):10.46Gh/s
 A:960  R:4096  HW:1  WU:138.4/m | ST: 1  SS: 0  NB: 2  LW: 1987  GF: 0  RF: 0
 Connected to mint.bitminter.com diff 64 with stratum as user evad
 Block: 3fa89e3b...  Diff:3.29T  Started: [21:11:25.670]  Best share: 1.77K
-----------------------------------------------------------------------------------------
 [U]SB management [P]ool management [S]ettings [D]isplay options [Q]uit
 0: GSD 10019882: COMPAC-2 100.00MHz (16/236/390/1) | 10.59G / 10.47Gh/s WU:138.4/m A:960
 R:0 HW:1--------------------------------------------------------------------------------
```

# Summary

This was a fun one. To get this a bit more stable, I likely need to relocate the "cluster" to my server cabinet for better cooling, the little USB keys get painfully hot. Another thing on the todo list for this, is to have cgminer run inside a container, and then on K8S.