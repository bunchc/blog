---
title: "ZeroVM - Some Background"
layout: post
date: 2014-10-15
categories: 
---

In a little over three weeks, at the [Paris Summit](https://www.openstack.org/summit/openstack-paris-summit-2014/), I'll be helping give a 90 minute workshop on [ZeroVM](http://zerovm.org/). 90 minutes is not actually a lot of time to get a good grasp on what ZeroVM is, how it operates, and most importantly, *why* it is important.

These next few posts however, I am hoping, will shed some light on that.

## What is ZeroVM?
From the ZeroVM website:

> ZeroVM is an open source virtualization technology that is based on the Chromium Native Client (NaCl) project. ZeroVM creates a secure and isolated execution environment which can run a single thread or application

Read that over a few times. What it is saying, is where Docker provides packaging, traditional virtualization provides full OS isolation, ZeroVM provides an environment specific to a single application or thread.

Explained another way, ZeroVM provides isolation at the thread or application level. It provides a "sandbox" environment for you to run arbitrary "untrusted" code. Some examples of how this could be useful can be found [here](http://play.golang.org/) and [here](https://www.python.org/shell/).

ZeroVM does this by using the [NaCL from Google](http://en.wikipedia.org/wiki/Google_Native_Client) to provide isolation and security. ZeroVM also has a number of other layers to provide the additional services for things like a Posix file system, I/O, and channels.

### Use Cases

Due to the small size, speed, and isolation nature of ZeroVM, it is able to solve some problems which are very "big data" or cloud centric. Let's take a look at some:

- [Apache Drill](http://incubator.apache.org/drill/) 
- [ZeroCloud](http://docs.zerovm.org/en/latest/zerocloud/overview.html)
    + More [here](http://openstack.prov12n.com/getting-started-with-zerovm/)

## Where Does ZeroVM Fit?

This diagram from the Atlanta OpenStack Summit should help:

![ZeroVM vs Traditional Virt](http://openstack.prov12n.com/screens/Using_ZeroVM_and_Swift_to_Build_a_Compute_Enabled_Storage_Platform_-_YouTube_2014-10-13_14-00-42.jpg)

Additionally, [this](https://www.youtube.com/watch?v=oR1RUSdUQCs#t=424) OpenStack Summit presentation should help clarify some as well.

### Traditional Virtualization

Traditional Virtualization is perhaps the easiest to cover, it's the most familiar way of isolating the things. I've been running VMware Workstation/Fusion, their enterprise products, etc for *ages*. I imagine a lot of you have as well. To recap tho:

- Type 1 or 2 hypervisor
- Each VM is a set of processes
- CPU instructions virtualized (it's a bit more complicated now-a-days with Hardware extensions)
- Carries around a full OS installation
- All resources isolated
- Very few *published* VM escapes

A few clarifying points. I've simplified the CPU conversation for this discussion. That is, while hardware extensions provide native or near native execution, there is still quite a bit happening in software for those instructions which do not translate. Additionally, the "all resources isolated" is a bit of a simplification as well. Most modern hypervisors will do some manner of memory sharing, compression, and consolidation. However, memory is still generally isolated to a specific VM.

### Containers (Docker / LXC)

Containers are all the rage these days. Docker has popularized and simplified the way we manage them. You get the same or better benefit from the shared hardware, however, containers strip quite a few of the layers:

- Shared Kernel / OS
- Super low overhead
- Fast startup
- 'secure'
- Managed via namespaces and process isolation

The important thing to note here, are the shared Kernel bits and the low overhead. The shared Kernel basically means everyone will run from the same Linux Kernel, and thus have the same features and limits therein. However, as an upside to that, you also shed having to carry around a full OS installation, networking stack, etc, and thus have much less overhead. This enables faster startup times.

### ZeroVM

Finally, and most important to what we're talking about is ZeroVM. ZeroVM is somewhere a bit beyond containers. That is you still have:

- Shared hardware
- Low Overhead
- Fast Startup

However, you are no longer carrying an OS or Kernel around. In turn, this further minimizes the attack surface. That is, there are a total of 6 system calls available from within ZeroVM.

In addition, due to it's extremely small size one can spin up a new ZeroVM instance in miliseconds, vs seconds or minutes for the other technologies. This speed advantage lends itself well to one of ZeroVMs primary use cases in the data-pipeline / data-processing space. Indeed it is the magic that makes ZeroCloud work. It does, however, require a different way of thinking about and approaching the problem. 

That is, ZeroVM instances are designed to be extremely temporary in nature. Load your data in, handle the processing, push it back out to disk and move on. Conceptually, it takes you a few steps further down the 'everything is disposable' path, insofar as you will need to design and rewrite your apps to work with ZeroVM.

## Summary

While a wall of words, I hope I have provided some clarity around the what of ZeroVM and where it fits.
