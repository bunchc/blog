---
title: "Run OpenStack Tempest in openstack-ansible"
date: 2017-09-05
layout: post
categories: openstack, testing, ansible, tempest, openstack-ansible
---

It took me longer than I would like to admit to get tempest running in openstack-ansible, so here is how I did it, in the hopes I'll remember (or that it will save someone else time).

## Getting Started

This post assumes a relatively recent version of openstack-ansible. For this post, that is an Ocata all-in-one build (15.1.7 specifically). Log into the deployment node.

## Installing Tempest

Before we can run tempest, we have to install it first. To install tempest, on the deployment node, run the following:

```
# cd /opt/openstack-ansible/playbooks
# openstack-ansible -vvvv os-tempest-install.yml

<<lots of output>>

PLAY RECAP *********************************************************************
aio1_utility_container-b81d907c : ok=68   changed=1    unreachable=0    failed=0
```

The prior command uses the openstack-ansible wrapper to pull in the appropriate variables for your openstack environment, and installs tempest.

## Running tempest

To run tempest, log into the controller node (this being an AIO, it is the deployment node, controller, compute, etc). Then attach to the utility container:

```
# lxc-ls | grep utility

aio1_utility_container-b81d907c

# lxc-attach --name aio1_utility_container-b81d907c
root@aio1-utility-container-b81d907c:~# ls
openrc
```

Once attached to the utility container, activate the tempest venv:

```
# cd /openstack/venvs/tempest-15.1.7
# source bin/activate
```

Once in the venv, we need to tell Tempest what workspace to use. Fortunately, the ```os-tempest-install``` playbook prepares a tempest 'workspace' for you. Lets move into that directory and launch our tempest tests:

```
(tempest-15.1.7) # cd /openstack/venvs/tempest-15.1.7/workspace
(tempest-15.1.7) # tempest run --smoke -w 4

<<so.much.output>>

======
Totals
======
Ran: 93 tests in 773.0000 sec.
 - Passed: 80
 - Skipped: 11
 - Expected Fail: 0
 - Unexpected Success: 0
 - Failed: 2
Sum of execute time for each test: 946.8377 sec.
```

All done!