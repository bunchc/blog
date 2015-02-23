---
title: "Instance Shelving"
layout: post
date: 2014-10-14
categories: 
---

Shelving? No no no, not that kind of shelving. Similar however. Image Shelving in Nova allows you to power off instances without experiencing the resource penalty in keeping them around.

## Instance Shelving

Before instance shelving in OpenStack, if a user powered down an instance, the resources would still be in use on the compute node that housed said instance. Quite wasteful, no? Say an instance is powered off for 72+ hours. Shelving allows you to keep all the various bits associated with the VM while moving it to somewhere that is not the hypervisor to conserve resources.

## Working with Shelving

The assumption here is that you have either devstack or some other flavor of OpenStack running. There isn't anything additional you need to configure to make this work.

First, let's log into our controller and look around at what's running:

```
# nova list
+--------------------------------------+-------+--------+------------+-------------+-----------------------------------------------+
| ID                                   | Name  | Status | Task State | Power State | Networks                                      |
+--------------------------------------+-------+--------+------------+-------------+-----------------------------------------------+
| 05617907-3931-414f-88f3-fd180f69fde6 | test1 | ACTIVE | -          | Running     | cookbook_network_1=10.200.0.2, 192.168.100.11 |
+--------------------------------------+-------+--------+------------+-------------+-----------------------------------------------+
```

Next, let's shelve an instance:

```
# nova shelve test1
```

Did it get shelved?

```
# nova list
+--------------------------------------+-------+-------------------+------------+-------------+-----------------------------------------------+
| ID                                   | Name  | Status            | Task State | Power State | Networks                                      |
+--------------------------------------+-------+-------------------+------------+-------------+-----------------------------------------------+
| 05617907-3931-414f-88f3-fd180f69fde6 | test1 | SHELVED_OFFLOADED | -          | Shutdown    | cookbook_network_1=10.200.0.2, 192.168.100.11 |
+--------------------------------------+-------+-------------------+------------+-------------+-----------------------------------------------+
```

**Note:** The change in status to "SHELVED_OFFLOADED"

Now let's unshelve it & check status:

```
# nova unshelve test1
# nova list
+--------------------------------------+-------+-------------------+------------+-------------+-----------------------------------------------+
| ID                                   | Name  | Status            | Task State | Power State | Networks                                      |
+--------------------------------------+-------+-------------------+------------+-------------+-----------------------------------------------+
| 05617907-3931-414f-88f3-fd180f69fde6 | test1 | SHELVED_OFFLOADED | spawning   | Shutdown    | cookbook_network_1=10.200.0.2, 192.168.100.11 |
+--------------------------------------+-------+-------------------+------------+-------------+-----------------------------------------------+
```

There we go!

## Summary

In this post, we worked with one of the new-ish features in Nova, instance shelving. Good for when you need to stop instances for a long time in a non-impactful way.

## References
- [Blueprint](https://blueprints.launchpad.net/nova/+spec/shelve-instance)
- [OpenStack Docs](http://docs.openstack.org/user-guide/content/shelve_server.html)
