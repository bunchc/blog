---
title: "Configuring Hierarchical Token Bucket (QoS) on Ubuntu 16.04"
date: 2016-10-17
layout: "post"
categories: Networking, QoS
---

Hierarchical Token Bucket, or HTB is one of a bunch of ways to perform QoS on Linux. This post will cover the HTB concept at a very (very) high level, and provide some example configurations that provide for both a traffic guarantee and a traffic limit.

# Linux Traffic Control

As I've recently discovered, Linux has an extremely robust network traffic management system. This goes well beyond network name spaces and iptables rules and into the world of QoS. Network QoS on Linux is handled by the 'traffic control' subsystem.

TC, or Traffic Control refers to the entirety of the queue management for a given network. For a good overview of TC, start [here.](http://tldp.org/HOWTO/Traffic-Control-tcng-HTB-HOWTO/intro.html#intro-tc)

## Hierarchical Token Bucket

The best explanation of Hierarchical Token Bucket, or HTB I've seen so far is to imagine a fixed size bucket into which traffic flows. This bucket can only drain network traffic at a given rate. Each token, or packet, is checked against a defined hierarchy, and released from the bucket accordingly.

This allows the system administrator to define both minimum guarantees for a traffic classification as well as limits for the same. Our configuration below explores this on two Ubuntu 16.04 VMs.

### Configuration

Configuring HTB has 3 basic steps:

+ Change the interface queuing to HTB
+ Define traffic classes
+ Configure filters to map traffic to it's class

The following example configures eth2 to use HTB, defines a class of traffic to be shaped to 1Mbit, and then uses the TC classifier to market all packets with a source or destination port of 8000 and map it to that class.


```
sudo su - 
# Change the queuing for eth2 to HTB
tc qdisc add dev eth2 root handle 1: htb

# Create a traffic class
tc class add dev eth2 parent 1: classid 1:8000 htb rate 1Mbit

# Create traffic filters to match the class
tc filter add dev eth2 protocol ip parent 1: prio 1 u32 \
    match ip dport 8000 0xffff flowid 1:8000

tc filter add dev eth2 protocol ip parent 1: prio 1 u32 \
    match ip sport 8000 0xffff flowid 1:8000
```

The commands above will produce no output if successful. If you would like to confirm your changes, the following commands will help:

```
## Report the queuing defined for the interface
$ tc qdisc show dev eth2

qdisc htb 1: root refcnt 2 r2q 10 default 0 direct_packets_stat 21 direct_qlen 1000

## Report the class definitions
$ tc class show dev eth2

class htb 1:8000 root prio 0 rate 1Mbit ceil 1Mbit burst 1600b cburst 1600b

## Report the filters configured for a given device
$ tc filter show dev eth2

filter parent 1: protocol ip pref 1 u32
filter parent 1: protocol ip pref 1 u32 fh 800: ht divisor 1
filter parent 1: protocol ip pref 1 u32 fh 800::800 order 2048 key ht 800 bkt 0 flowid 1:80
  match 00001f40/0000ffff at 20
filter parent 1: protocol ip pref 1 u32 fh 800::801 order 2049 key ht 800 bkt 0 flowid 1:80
  match 1f400000/ffff0000 at 20
```

With these configured, it is now time to test our configuration...

### Testing HTB

For our test scenario, we are going to use two virtual machines on the same L2 network, something like this:

```
         +-------------------------------------+
         | vSwitch                             |
         +-------------------------------------+
         eth1 |                       | eth2
         +--------------+       +--------------+
         | server01     |       | server02     |
         | 172.16.1.101 |       | 172.16.2.101 |
         +--------------+       +--------------+
```

First, you'll need to install iperf on both nodes:

```
sudo apt-get install -qy iperf
```

Then on server02, start iperf in server mode listening on port 8000 as we specified in our filter:

```
sudo iperf -s -p 8000
```

On server01, we then run the iperf client. We specify to connect to server02, run the test for 5 minutes (300 seconds), bidirectionally (-d) on port 8000 (-p) and report statistics at a 5 second interval (-i):

```
sudo iperf -c 172.16.2.101 -t 300 -d -p 8000 -i 5
```

If you have configured things correctly you will see output like the following:

```
[  5] 45.0-50.0 sec   379 KBytes   621 Kbits/sec
[  3] 45.0-50.0 sec   768 KBytes  1.26 Mbits/sec
[  5] 50.0-55.0 sec   352 KBytes   577 Kbits/sec
```

with that, you have configured and tested your first HTB traffic filter.

# Summary

In this post there was a lot going on. The tl;dr is, Linux supports network QoS through a mechanism called Traffic Control or TC. HTB is one of many techniques you can use to control said traffic. In this post, we also explored how to configure and test an HTB traffic queue.

# Resources

+ [http://tldp.org/HOWTO/Traffic-Control-HOWTO/intro.html](http://tldp.org/HOWTO/Traffic-Control-HOWTO/intro.html)
+ [http://lartc.org/manpages/tc.txt](http://lartc.org/manpages/tc.txt)
+ [http://lartc.org/howto/lartc.cookbook.fullnat.intro.html](http://lartc.org/howto/lartc.cookbook.fullnat.intro.html)
+ [https://github.com/bunchc/linux-tc-demo](https://github.com/bunchc/linux-tc-demo)
+ [https://www.linux.com/blog/tc-show-manipulate-traffic-control-settings](https://www.linux.com/blog/tc-show-manipulate-traffic-control-settings)
+ [http://tldp.org/HOWTO/Traffic-Control-tcng-HTB-HOWTO/intro.html#intro-tc](http://tldp.org/HOWTO/Traffic-Control-tcng-HTB-HOWTO/intro.html#intro-tc)
