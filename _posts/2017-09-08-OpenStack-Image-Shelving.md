---
title: "OpenStack Image Shelving"
date: 2017-09-08
layout: "post"
categories: openstack, nova, compute, shelving
---

Somewhat unique to OpenStack Nova is the ability to "shelve" an instance. Instance shelving allows you to retain the state of an instance without having it consume resources. A shelved instance will be retained as a bootable instance for a configurable amount of time, then deleted. This is useful as part of an instance lifecycle process, or to conserve resources.

## Getting Ready

To shelve an instance, the following information is required:

* ```openstack``` command line client
* ```openrc``` file containing appropriate credentials
* The name or ID of the instance

## How to do it...

To shelve an instance, the following commands are used:

Check the status of the instance
```
# openstack server show cookbook.test -c name -c status
+--------+---------------+
| Field  | Value         |
+--------+---------------+
| name   | cookbook.test |
| status | ACTIVE        |
+--------+---------------+
```

Shelve the instance

```
# openstack server shelve cookbook.test
```

> __Note:__ This command produces no output when successful. Shelving an instance may take a few moments depending on your environment.

Check the status of the instance

```
# openstack server show cookbook.test -c name -c addresses -c status
+-----------+-------------------+
| Field     | Value             |
+-----------+-------------------+
| addresses | public=10.1.13.9  |
| name      | cookbook.test     |
| status    | SHELVED_OFFLOADED |
+-----------+-------------------+
```

> __Note:__ A shelved instance will retain the addresses it has been assigned.

Un-shelving the instance

```
# openstack server unshelve cookbook.test
```

> __Note:__ This command produces no output when successful. As with shelving the instance may take a few moments to become active depending on your environment.

Check the status

```
# openstack server show cookbook.test -c name -c addresses -c status
+-----------+------------------+
| Field     | Value            |
+-----------+------------------+
| addresses | public=10.1.13.9 |
| name      | cookbook.test    |
| status    | ACTIVE           |
+-----------+------------------+
```

## How it works...

When told to shelve an instance, OpenStack compute will first stop the instance. It then creates an instance snapshot to retain the state of the instance. The runtime details, such as number of vCPUs, memory, and IP addresses, are retained so that the instance can be unshelved and rescheduled at a later time.

This differs from shutdown an instance, in that the resources of a shutdown instance are still reserved on the host on which it resided, so that it can be powered back on quickly. A shelved instance however, will still show in ```openstack server list```, while the resources that were assigned will remain available. Additionally, as the shelved instance will need to be restored from an image OpenStack compute will perform placement as if the instance were new, and starting it will take some time.