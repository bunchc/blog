---
title: "Running Rackspace Private Cloud on the Rackspace Public Cloud"
date: 2014-10-13
categories: 
---

Private Cloud on the Public Cloud? As odd as that sounds, or as inception as it makes you feel (Clouds in clouds?!), I've found that since downsizing my homelab quite a bit, I've needed to find other ways to work on and try out various things that exceed the capacity of my laptop. RPC 9 is one of them.

**Disclaimer:** I work for Rackspace, and while this post is largely focused around two of our products, I put it out here in the hopes that a) someone will find it useful, and b) it'll help me later.

## Rackspace Private Cloud (RPC)

Our docs will do it a lot more justice in terms of description than I can, so I encourage you to go [here](http://docs.rackspace.com/rpc/api/v9/bk-rpc-installation/content/index.html) and take a few minutes to get familiar.

A few things I want to point out are related to the architecture of RPC9, specifically:

![RPC Infrastructure](http://docs.rackspace.com/rpc/api/v9/bk-rpc-installation/content/figures/1/a/a/a/rpc-common/figures/rpc9-environment-overview.png)

Looking over the diagram, there are a *lot* of hosts involved now. 3x Infrastructure nodes, a logging host, n-Compute hosts, deployment hosts, and finally a set of load balancers. This thing is big. Bigger than my laptop that's for sure.

## Running Cloud on Cloud

Thankfully, however, while it's big, the folks who wrote this provided some OpenStack Heat templates that make setting it up externally much easier. Those can be found [here.](https://github.com/rcbops/ansible-lxc-rpc/tree/master/scripts)

### Getting Started

To build the cloud on the cloud you'll need the following info & apps installed somewhere you have access to:

- An "heatrc" or similar file containing
    + Rackspace Username
    + Rackspace API Key
    + Endpoint(s) to deploy to
- Python-HeatClient
- An SSH Key

To create the "heatrc" file, start with the below template and then edit as needed:

```
export OS_USERNAME=rackspace_cloud_username
export OS_PASSWORD=rackspace_cloud_password
export OS_TENANT_ID=rackspace_cloud_account_number
export OS_AUTH_URL=https://identity.api.rackspacecloud.com/v2.0/
export HEAT_URL=https://ord.orchestration.api.rackspacecloud.com/v1/${OS_TENANT_ID}
```

**Note:** OS_TENANT_ID is your cloud account number. You can get to this by logging into mycloud.rackspace.com and clicking your account name in the upper right.

Once you've created the file and replaced said values, install the Heat client: ```pip install python-heatclient```

### Installing RPC 9 on the Rackspace Public Cloud

To kick off the installation, follow these commands:

```
curl https://raw.githubusercontent.com/rcbops/ansible-lxc-rpc/master/scripts/rpc9.0.0-aio-rax-heat-template.yml > rpc9-rax-heat.yaml

source ./heatrc

heat stack-create RPC9-Stack -f ./rpc9-rax-heat.yaml \
  -P image_name="Ubuntu 14.04 LTS (Trusty Tahr)" \
  -P ssh_key_name="lol_ssh_key" \
  -P flavor_name="8 GB Performance"

+--------------------------------------+------------+--------------------+----------------------+
| id                                   | stack_name | stack_status       | creation_time        |
+--------------------------------------+------------+--------------------+----------------------+
| c2b6c1b0-0098-441d-9999-c778b108a181 | RPC9-Stack | CREATE_IN_PROGRESS | 2014-10-13T15:13:58Z |
+--------------------------------------+------------+--------------------+----------------------+

```

This next part takes quite a bit of time to complete and is why we used a performance instance, to make the provision happen just a bit faster. You can keep an eye on it's build status with ```watch -n 15 heat stack-list```

Once it completes, you will need to find out what IP address it has been assigned, to do that, use these commands: 

List the stacks:
``` 
$ heat stack-list
+--------------------------------------+------------+-----------------+----------------------+
| id                                   | stack_name | stack_status    | creation_time        |
+--------------------------------------+------------+-----------------+----------------------+
| c2b6c1b0-0098-441d-9999-c778b108a181 | RPC9-Stack | CREATE_COMPLETE | 2014-10-13T15:13:58Z |
+--------------------------------------+------------+-----------------+----------------------+
```

List the available outputs:
```
$ heat output-list RPC9-Stack
+------------------+-------------------------------------------------------+
| output_key       | description                                           |
+------------------+-------------------------------------------------------+
| RPCAIO_password  | The password for all the things.                      |
| RPCAIO_public_ip | The public IP address of the newly configured Server. |
+------------------+-------------------------------------------------------+
```

Finally show the IP:
```
$ heat output-show RPC9-Stack RPCAIO_public_ip
"127.0.0.100"
```

## Summary

In this post we showed you how to nest the Rackspace Private Cloud installation on the Rackspace Public Cloud. A useful trick for testing it out without having to use *ALL* your local resources up.