---
title: "Updated Vagrant Boxes for OpenStack Cookbook 3rd Edition"
date: 2017-05-31
layout: "post"
categories: devops, openstack, vagrant, openstack cookbook, book
---

A reader reached out a day or so ago to point out that the Vagrant images used in the 3rd edition of the OpenStack Cookbook had gone missing.

Why? Well, long story short, I accidentally the images.

That said, I buit new ones, which live on [Atlas here.](https://atlas.hashicorp.com/bunchc/boxes/trusty-x64/versions/0.0.2)

Changes in this release:

* Updated to ubuntu 14.04.5
* Updated NIC drivers
    - Virtualbox: Virtio
    - VMware: vmxnet3
* Changed from preseed to kickstart

The images are still rather large. However, as we are in the process of writing the 4th edition, I'll likely leave them that size.
