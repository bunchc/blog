---
title: "Useful bash functions for virsh/kvm"
date: 2016-10-17
layout: "post"
categories: KVM, virsh, bash
---

When working on getting [my first VM on KVM](http://blog.codybunch.com/2016/10/14/KVM-and-OVS-on-Ubuntu-1604/) started, there were some things that were missing or non-obvious to a newbie like me.

That is, listing the IP address assigned to a VM or 'dom' required some digging. (Note, this is addressed in newer releases of KVM). To that end, and after a lot of time googling, I came across [this](https://gist.github.com/kevteljeur/ad73ca9ca0cfe8f14458) gist that provides a lot of useful functions.

Specific to the IP address bit, it gives you virt-addr:

```
## List all our VMs
# virsh list --all
 Id    Name                           State
----------------------------------------------------
 2     logging1.openstackci.local     running
 3     network2.openstackci.local     running
 4     network1.openstackci.local     running
 5     infra1.openstackci.local       running
 6     infra2.openstackci.local       running
 7     infra3.openstackci.local       running
 8     compute1.openstackci.local     running
 9     compute2.openstackci.local     running
 10    cinder2.openstackci.local      running
 11    cinder1.openstackci.local      running
 12    swift1.openstackci.local       running
 13    swift3.openstackci.local       running
 14    swift2.openstackci.local       running

## List all IPs for ID 5
# virt-addr 5
10.0.0.17
172.29.236.100
10.0.0.100
10.0.0.4

## Works by name too
# virt-addr infra1.openstackci.local
10.0.0.17
172.29.236.100
10.0.0.100
10.0.0.4
```

