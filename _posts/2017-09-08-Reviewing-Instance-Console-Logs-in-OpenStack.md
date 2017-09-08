---
title: "Reviewing Instance Console Logs in OpenStack"
date: 2017-09-08
layout: "post"
categories: openstack, troubleshooting, compute, nova
---

Console logs are critical for troubleshooting the startup process of an instance. These logs are produced at boot time, before the console becomes available. However when working with a cloud hosted instances, accessing these can be difficult. OpenStack Compute provides a mechanism for accessing the console logs.

## Getting Ready

To access the console logs of an instance, the following information is required:

* ```openstack``` command line client
* ```openrc``` file containing appropriate credentials
* The name or ID of the instance

For this example we will view the last 5 lines of the ```cookbook.test``` instance.

## How to do it...

To show the console logs of an instance, use the following command:

```
# openstack console log show --lines 5 cookbook.test

[[0;32m  OK  [0m] Started udev Coldplug all Devices.
[[0;32m  OK  [0m] Started Dispatch Password Requests to Console Directory Watch.
[[0;32m  OK  [0m] Started Set console font and keymap.
[[0;32m  OK  [0m] Created slice system-getty.slice.
[[0;32m  OK  [0m] Found device /dev/ttyS0.
```

## How it works...

The ```openstack console log show``` command collects the console logs, as if you were connected to the server via a serial port or sitting behind the keyboard and monitor at boot time. The command will, by default, return all of the logs generated to that point. To limit the amount of output, the ```--lines``` parameter can be used to return a specific number of lines from the end of the log.
