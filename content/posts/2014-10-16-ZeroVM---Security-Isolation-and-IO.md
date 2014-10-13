---
title: "ZeroVM - Isolation"
date: 2014-10-16
categories: 
---

Here we go with another post on ZeroVM. This time to help get you up to speed on how ZeroVM provides isolation. For a reminder of what ZeroVM is, how it is different than containers and other virtualization technologies, see my prior post [here](http://blog.codybunch.com/posts/2014-10-15-ZeroVM---Some-Background/), or the ZeroVM docs [here](http://docs.zerovm.org/zerovm).

## Isolation

The [ZeroVM site](http://docs.zerovm.org/zerovm/isolation_security.html) itself doesn't say much on the isolation provided by ZeroVM. This is largely because it derives it's isolation through the use of the [Google NaCL project](http://en.wikipedia.org/wiki/Google_Native_Client). The tl;dr for NaCL, is that it requires applications be ported over to it's sandboxing environment, in which a subset of processor instructions are made available.

The sandbox environment provided by NaCL is a limited subset of processor instructions that prevent various syscalls that could be destructive. Further, ZeroVM takes the 50 syscalls available in NaCL and reduces that to six. Meaning, if your code contains a syscall other than one of the six currently allowed, your code will fail when ZeroVM attempts to execute the specific syscall. The six syscalls allowed are currently:

- pread
- pwrite
- jail
- unjail
- fork
- exit

We can test this by trying an invalid write instead of pwrite with the following C++

```
#include <unistd.h>
#include <stdio.h>
#include <sys/types.h>

int main()
{
    char data[128];

    if(read(0, data, 128) < 0)
     write(2, "An error occurred in the read.\n", 31);

}
```

This is what it looks like when it implodes:
```
$ zvsh syscalls
----------/tmp/tmpl7Mi1v----------
Node = 1
Version = 20130611
Timeout = 50
Memory = 4294967296, 0
Program = /home/vagrant/syscalls/syscalls
Channel = /dev/stdin,/dev/stdin,0,0,4294967296,4294967296,0,0
Channel = /tmp/tmpOfgnG6/stdout,/dev/stdout,0,0,0,0,4294967296,4294967296
Channel = /tmp/tmpOfgnG6/stderr,/dev/stderr,0,0,0,0,4294967296,4294967296
Channel = /tmp/tmpeNIzx4,/dev/nvram,3,0,4294967296,4294967296,4294967296,4294967296
-------------------------
----------/tmp/tmpeNIzx4----------
[args]
args = syscalls
[mapping]
channel=/dev/stdin,mode=char
channel=/dev/stdout,mode=char
channel=/dev/stderr,mode=char

-------------------------
2
0
0
disable
0.00 0.00 0 0 0 0 0 0 0 0
src/loader/elf_util.c 178: Segment 0 is of unexpected type 0x6, flag 0x5

ERROR: ZeroVM return code is 8
```

What is important here, at least in terms of telling why it failed is: ```ERROR: ZeroVM return code is 8```. Error codes 1 - 17 indicate an issue in untrusted code. In our example, it was specifically the read / write bits that caused the issue.

## Down The Rabbit Hole

Time to jump down the rabbit hole. Into what actually just happened. First some diagrams:

### ZeroVM Stack
In this first diagram, you are looking at the internal structure of the ZeroVM process.
![ZVM Trusted Code Base Architecture](http://openstack.prov12n.com/screens/ZeroVM-Architecture-Design-Overview.pdf_2014-10-13_15-14-18.jpg)

Working from the outside in, the grey area represents everything that is ZeroVM, all of it's runtime data, IO Channels, virtual file systems, and most importantly, the user code to be executed.

Next, in the pink area is where end-user code gets loaded up, from there a call will progress down the stack. In the last white layer you can see the individual sys-calls as implemented in ZeroVM as well as their corresponding trappings.

One last thing to note is the two sys-calls for zvm_pread and zvm_pwrite, their traps, handlers, and this thing called a trampoline. The trampolines are a mechanisim that facilitates the switch from untrusted to trusted execution. A syscall will start in the 'untrusted' context, if it is an allowed call, the ZRT or ZeroVM Run Time traps the call, and uses the trampoline to perform the context switch needed to allow the syscall to execute in trusted space.

### ZeroVM Guest Memory Footprint

This next diagram offers a different view on the above ZeroVM guest. In the below diagram, you will see how the NaCL & ZeroVM *trampolines* are implemented in memory, how that in turn gets passed down to the corresponding syscall, the trusted ELF binary, and where the rest of ZeroVM is contained.

![Anatomy of the ZeroVM Guest Memory footprint](http://openstack.prov12n.com/screens/ZeroVM-Architecture-Design-Overview.pdf_2014-10-13_15-18-27.jpg)

## Summary

In this post, we took a deeper look into how isolation is handled within ZeorVM. We started with a high level discussion of NaCL and ZeroVM's implementation of limited syscalls. We then attempted to execute an invalid syscall. Finally we got into the nitty gritty of how that syscall is trapped via the ZeroVM runtime and where in it's memory stack context switches from trusted to untrusted code happen.

## Resources
- [Diagrams from ZeroVM G+ Group](https://docs.google.com/viewer?a=v&pid=forums&srcid=MDM5MTg3MTAwNDAwMTI4Njc5NzkBMTUyNDg3NTk2NDczMTE1MDYyMTIBcENUQTliQjJwSXNKATAuMQEBdjI)
- [ZeroVM Architecture Slideshare](http://www.slideshare.net/sgt_mac/zero-vm-architecture)