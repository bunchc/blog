---
title: "ZeroVM - IO Operations in ZeroVM"
layout: post
date: 2014-10-21
categories: 
---

If you are just joining these posts, you might want to take a few minutes and review these other posts on ZeroVM:

- [Background](http://blog.codybunch.com/posts/2014-10-15-ZeroVM---Some-Background)
- [Isolation](http://blog.codybunch.com/posts/2014-10-16-ZeroVM---Security-Isolation-and-IO)
- [Getting Started with ZeroVM](http://blog.codybunch.com/posts/2014-10-17-ZeroVM---Getting-Started-Again/) 
- [Lots of ZeroVM Resources](http://blog.codybunch.com/posts/2014-10-20-ZeroVM-Link-Dump)

## IO Operations in ZeroVM

IO in ZeroVM is an odd sort of beast. As you are not working in a container or a VM you do not inherit the unlimited read/write to arbirtrary places as you would have otherwise. Instead the abstraction unit for ZeroVM is the *channel*.

### Channels

As the intro said, a *channel* is the unit of IO abstraction within ZeroVM. That means any network communication, any file write, any input from a pipe, or output to swift, each is an individual channel. 

Additionally, channels have to be explicitly defined prior to launching a ZeroVM instance. In some ways, this is like specifying the VMDK for a traditional VM to boot from. The requirement to specify all channels before hand does provide a powerful security mechanism in that all IO is effectively sandboxed. An erroneous process can not cause damage outside of the narrow constraints defined within said channel.

Channels in ZeroVM are unique in other ways. Each channel, regardless of type, is presented to the instance as a file handle. 

### Channel Quotas

To keep with the security and isolation provided within ZeroVM, along with the explicit channel definition, one must also specify a quota for IO. You specify said quota with four bits of information: (1) Total reads; (2) Total writes; (3) Total Read Size; and (4) Total Write Size.

Quotas are handled on a 'first one hit' basis. So, if you have 10,000 reads to do, but said number of reads exceeds the total read size in bytes at read number 3,500, well, that's it. No more reads for you. The inverse of this is also true, when you run out of the number of allowed reads or writes before you hit the size quota, you will also get a failure returned.

When you do hit this limit, however, ZeroVM tries to fail gracefully, in that you will still have all the data read or written up to that point. Below is an example of what happens when you exceed the number of available reads:

```
$ dd if=/dev/urandom of=/home/vagrant/file.txt bs=1048576 count=100

$ cat > /home/vagrant/example.py <<EOF
file = open('/dev/3.file.txt', 'r')
print file.read(5)
EOF

$ su - vagrant -c "zvsh --zvm-save-dir ~/ --zvm-image python.tar --zvm-image ~/file.txt python @example.py"

$ sudo sed -i "s|Channel = /home/vagrant/file.txt,/dev/3.file.txt,3,0,4294967296,4294967296,0,0|Channel = /home/vagrant/file.txt,/dev/3.file.txt,3,0,3,3,3,3|g" /home/vagrant/manifest.1
$ sudo sed -i "s|/home/vagrant/stdout.1|/dev/stdout|g" /home/vagrant/manifest.1
$ sudo sed -i "s|/home/vagrant/stderr.1|/dev/stderr|g" /home/vagrant/manifest.1

$ zerovm -t 2 manifest.1

Traceback (most recent call last):
  File "/dev/1.example.py", line 2, in <module>
    print file.read(5)
IOError: [Errno -122] Unknown error 4294967174
```

What is happening in that example is this:

**1.** Create a 100MB file full of random bits

**2.** Create 'example.py' to read the first 5 characters in the file

**3.** Use zvsh to run example.py inside ZeroVM... this should dump some garbage characters to the console.

**4.** Change the quota from 'huge' to 3

**5.** Adjust the manifest file to allow stderr and stdout to work properly

**6.** Run ZeroVM with the updated manifest & receive IO error

### Specifying Channels in a Manifest File

So that example took me way too long to figure out. Mostly due to a few small things and one big one: Creating the manifest file properly. What follows is a quick way to create a manifest file as well as our example manifest and a breakdown thereof.

#### Creating a Manifest File

It turns out that the zvsh utility that is bundled with ZeroVM has the ability to save the manifest (and other runtime files) when you launch it. To generate the template manifest that is used and modified in this example, run the following command:

```
zvsh --zvm-save-dir ~/ --zvm-image python.tar --zvm-image ~/file.txt python @example.py
```

#### Breakdown of the Manifest File

The example file that follows was for the specific bits used in our example. It can, however, make for a good starting point for your own manifest files.

```
Node = 1
Version = 20130611
Timeout = 50
Memory = 4294967296,0
Program = /home/vagrant/boot.1
Channel = /dev/stdin,/dev/stdin,0,0,4294967296,4294967296,0,0
Channel = /dev/stdout,/dev/stdout,0,0,0,0,4294967296,4294967296
Channel = /dev/stderr,/dev/stderr,0,0,0,0,4294967296,4294967296
Channel = /home/vagrant/example.py,/dev/1.example.py,3,0,4294967296,4294967296,0,0
Channel = /home/vagrant/python.tar,/dev/2.python.tar,3,0,4294967296,4294967296,0,0
Channel = /home/vagrant/file.txt,/dev/3.file.txt,3,0,3,3,3,3
Channel = /home/vagrant/boot.1,/dev/self,3,0,4294967296,4294967296,0,0
Channel = /home/vagrant/nvram.1,/dev/nvram,3,0,4294967296,4294967296,4294967296,4294967296
```

Taken line by line:

**1.** **Node** an optional parameter specifying the node number.

**2.** **Version** mandatory. As manifest versions are incompatible. This tells both ZeroVM and you which version of the manifest you are using.

**3.** **Timeout** in seconds, is a mandatory parameter. This prevents long running ZeroVM jobs from consuming excess resources.

**4.** **Memory** Also mandatory, This is a comma separated value, where the first position is the 32-bit value representing memory. In our case, the max 4GB. The second number can be either 0 or 1 specifying the eTag used to checksum all data passing through ZeroVM

**5.** **Program** Specifies the ZeroVM cross compiled binary that we will run

**6.** **Channel** The channel definitions, these follow the format:

```
Channel = <uri>,<alias>,<type>,<etag>,<gets>,<get_size>,<puts>,<put_size>
```

## Summary

In this post we covered how IO is handled in ZeroVM. We first explained the Channel concept, how quotas are handled within channels, and finally, how to add a channel or channels at ZeroVM runtime using a manifest.
