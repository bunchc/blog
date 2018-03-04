---
title: "Linux Filesystem Performance for Virt Workloads"
date: 2018-03-03
layout: "post"
categories: benchmark, filesystem, performance, virt, vagrant, virtualbox, kvm
---

As I spend quite a bit of time (an understatement, I assure you) standing up and tearing down different virtualized lab environments, I wanted to spend a little bit less on it overall. Thus, in addition to tuning some parameters at runtime, I spent some time benchmarking the difference between Virtualization engines, filesystems, and IO schedulers.

Before we begin, let's get a few things out of the way:

* TL;DR - The winner was libvirt/kvm with ext4 in guest, xfs on host, with noop
* Yes, my hardware is old.
* No, this is not exhaustive, nor super scientific

## Test Hardware

CPU: 2x Quad-Core AMD Opteron(tm) Processor 2374 HE
RAM: 64GB
Disk: 4x 1T 5400 rpm disks, Raid 1
Controller: LSI MegaRAID 8708EM2 Rev: 1.40
OS: Ubuntu 16.04 LTS

> Old hardare is old. But hey, it is a workhorse.

## Test Workload

Building an [openstack-ansible](https://governance.openstack.org/tc/reference/projects/openstackansible.html) All-In-One, for the Pike release of OpenStack.

__Rationale:__
To be honest, this is the workload I spend the most time with. Be it standing one up to replicate a customer issue, test an integration, build a solution, and so on. Any time I save provisioning, is time I can spend doing work

Further, the build process for an all-in-one is quite extensive and encompasses a wide variety of sub workloads: haproxy, rabbitmq, galera, lxc containers, and so on.

## Test Matrix

The test matrix worked out to 8 tests in all:

| Host FS | Guest FS | IO Sched | Virt Engine |
|:-------:|:--------:|:--------:|:-----------:|
| xfs     | xfs      |  noop    | KVM         |
| xfs     | xfs      |  noop    | vbox        |
| xfs     | ext4     |  noop    | KVM         |
| xfs     | ext4     |  noop    | vbox        |
| xfs     | xfs      | deadline | KVM         |
| xfs     | xfs      | deadline | KVM         |
| xfs     | ext4     | deadline | vbox        |
| xfs     | ext4     | deadline | vbox        |

## Test Process

__Prepwork:__

* Create four different boxes with Packer (2 filesystems * 2 virt engines).
* Creating a vagrant file that corresponded to each scenario
* Create bash script to loop through the scenarios

__Test:__

As the goal was to reduce the time spent waiting on environments, each environment was tested with:

```
$ time (vagrant up --provider=$PROVIDER_NAME)
```

## Results

__Test 1:__

__Host FS:__ xfs
__Guest FS:__ xfs
__Virt Engine:__ libvirt/kvm
__Host IO Scheduler:__ noop
__Total Time:__ 174m48.193s

__Test 2:__

__Host FS:__ xfs
__Guest FS:__ xfs
__Virt Engine:__ vbox
__Host IO Scheduler:__ noop
__Total Time:__ 213m35.169s

__Test 3:__

__Host FS:__ xfs
__Guest FS:__ ext4
__Virt Engine:__ libvirt/kvm
__Host IO Scheduler:__ noop
__Total Time:__ 172m5.682s

__Test 4:__

__Host FS:__ xfs
__Guest FS:__ ext4
__Virt Engine:__ vbox
__Host IO Scheduler:__ noop
__Total Time:__ 207m53.895s

__Test 5:__

__Host FS:__ xfs
__Guest FS:__ xfs
__Virt Engine:__ libvirt/kvm
__Host IO Scheduler:__ deadline
__Total Time:__ 172m44.424s

__Test 6:__

__Host FS:__ xfs
__Guest FS:__ xfs
__Virt Engine:__ vbox
__Host IO Scheduler:__ deadline
__Total Time:__ 235m34.411s

__Test 7:__

__Host FS:__ xfs
__Guest FS:__ ext4
__Virt Engine:__ libvirt/kvm
__Host IO Scheduler:__ deadline
__Total Time:__ 172m31.418s

__Test 4:__

__Host FS:__ xfs
__Guest FS:__ ext4
__Virt Engine:__ vbox
__Host IO Scheduler:__ deadline
__Total Time:__ 209m43.955s

# Conclusions

The combination that won overall was an ext4 guest filesystem, with an xfs host filesystem, on libvirt/kvm with the noop IO scheduler.

While I expected virtualbox to be slower than KVM, an entire hours difference was pretty startling. Another surprise was that ext4 on xfs outperformed xfs on xfs in all cases.