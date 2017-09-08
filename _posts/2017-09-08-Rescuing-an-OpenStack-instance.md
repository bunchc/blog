---
title: "Rescuing an OpenStack instance"
date: 2017-09-08
layout: "post"
categories: openstack, troubleshooting
---

# Rescuing an instance

OpenStack compute provides a handy troubleshooting tool with rescue mode. Should a user lose an SSH key, or be otherwise not be able to boot and access an instance, say, bad IPTABLES settings or failed network configuration, rescue mode will start a minimal instance and attach the disk from the failed instance to aid in recovery.

## Getting Ready

To put an instance into rescue mode, you will need the following information:

* ```openstack``` command line client
* ```openrc``` file containing appropriate credentials
* The name or ID of the instance

The instance we will use in this example is ```cookbook.test```

## How to do it...

To put an instance into rescue mode, use the following command:

```
# openstack server rescue cookbook.test
+-----------+--------------+
| Field     | Value        |
+-----------+--------------+
| adminPass | zmWJRw6C5XHk |
+-----------+--------------+
```

To verify an instance is in rescue mode, use the following command:

```
# openstack server show cookbook.test -c name -c status
+--------+---------------+
| Field  | Value         |
+--------+---------------+
| name   | cookbook.test |
| status | RESCUE        |
+--------+---------------+
```

> __Note:__ When in rescue mode, the disk of the instance in rescue mode is attached as a secondary. In order to access the data on the disk, you will need to mount it.

To exit rescue mode, use the following command:

```
# openstack server unrescue cookbook.test
```

> __Note:__ This command will produce no output if successful.

## How it works...

The command ```openstack server rescue``` provides a rescue environment with the disk of your instance attached. First it powers off the named instance. Then, boots the rescue environment, and attaches the disks of the instance. Finally, it provides you with the login credentials for the rescue instance.

Accessing the rescue instance is done via SSH. Once logged into the rescue instance, you can mount the disk using ```mount <path to disk> /mnt```.

Once you have completed your troubleshooting or recovery, the unrescue command reverses this process. First stopping the rescue environment, and detaching the disk. Then booting the instance as it was.