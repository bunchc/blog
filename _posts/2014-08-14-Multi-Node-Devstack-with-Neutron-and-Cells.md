---
title: "Multi-Node Devstack with Neutron and Cells"
layout: post
date: 2014-08-14
categories: 
---

OpenStack Cells allow you to break up Nova into smaller domains. In turn, this allows for a number of interesting things. Not the least of which is scale.

From the OpenStack.org docs: **Note** *I've added the bold on some important points.*

>Cells functionality enables you to scale an OpenStack Compute cloud in a more distributed fashion without having to use complicated technologies like database and message queue clustering. It supports very large deployments.

>When this functionality is enabled, the hosts in an OpenStack Compute cloud are partitioned into groups called cells. Cells are configured as a tree. The top-level cell should have a host that runs a nova-api service, but no nova-compute services. Each child cell should run all of the typical nova-* services in a regular Compute cloud except for nova-api. **You can think of cells as a normal Compute deployment in that each cell has its own database server and message queue broker.**

So, in addition to breaking down at physical boundaries or failure domains, you can break nova-compute into smaller chunks based on DB and MQ scaling limits. 

However, not all of us have multi-geo hundred plus node compute labs to play with... so how do we test out this functionality before hand? Devstack!

## Getting Started

tl;dr - ```git clone https://github.com/bunchc/devstack-cells.git; cd devstack-cells; vagrant up```, and go to "Configuring and Creating Cells"

To get started, you'll need the following:

- virtualbox (fusion or workstation work just as well)
- a minimum of two Ubuntu 14.04 VM with:
    + About 2GB ram
    + 2x networks
        * eth0 = NAT
        * eth1 = host only

### Both Nodes

Once you have that taken care of, on each node we need to create a stack user:
```
sudo adduser --disabled-password --gecos "" stack
echo "stack ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
```

### Parent Node

Time to Download devstack:
```
sudo su - stack
git clone -b stable/icehouse https://github.com/openstack-dev/devstack 
```

> Note the sudo command. Everything from this point is done as stack

Next, let's make the local.conf file:
```
echo "
[[local|localrc]]
ADMIN_PASSWORD=$ADMIN_PASSWORD
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
SERVICE_TOKEN=a682f596-76f3-11e3-b3b2-e716f9080d50

GIT_BASE=${GIT_BASE:-https://git.openstack.org}

# Cells!
ENABLED_SERVICES+=,n-cell,n-api-meta
DISABLED_SERVICE+=,n-cpu,n-net,n-sch

# Neutron - Networking Service
# If Neutron is not declared the old good nova-network will be used
ENABLED_SERVICES+=,q-svc,q-agt,q-dhcp,q-l3,q-meta,neutron,q-lbaas,q-vpn,q-fwaas

# Neutron Stuff
OVS_VLAN_RANGES=RegionOne:1:4000
OVS_ENABLE_TUNNELING=False

## Images
# 32bit image (~660MB)
IMAGE_URLS+=",http://fedorapeople.org/groups/heat/prebuilt-jeos-images/F19-i386-cfntools.qcow2"
# 64bit image (~640MB)
IMAGE_URLS+=",http://fedorapeople.org/groups/heat/prebuilt-jeos-images/F19-x86_64-cfntools.qcow2"
IMAGE_URLS+=",http://mirror.chpc.utah.edu/pub/fedora/linux/releases/20/Images/x86_64/Fedora-x86_64-20-20131211.1-sda.qcow2"
IMAGE_URLS+=",http://launchpad.net/cirros/trunk/0.3.0/+download/cirros-0.3.0-x86_64-uec.tar.gz"

# Output
LOGFILE=/opt/stack/logs/stack.sh.log
VERBOSE=True
LOG_COLOR=False
SCREEN_LOGDIR=/opt/stack/logs
" | tee -a /home/stack/devstack/local.conf
```

The important bit for cells in this file are:
```
ENABLED_SERVICES+=,n-cell,n-api-meta
DISABLED_SERVICE+=,n-cpu,n-net,n-sch
```

This enables the cell service, and the nova-api metadata service. It then disables compute, nova-networking, and the scheduler. Those tasks will be handled on the children.

With that out of the way run ```./stack.sh``` and grab a coffee. When it completes, add the following to ```/etc/nova/nova.conf```:

```
[cells]
enable=True
name=api
cell_type=api
```

Finally, restart the n-api serivice.

### Child Node

Time to Download devstack:
```
sudo su - stack
git clone -b stable/icehouse https://github.com/openstack-dev/devstack 
```

> Note the sudo command. Everything from this point is done as stack

Next, on the child node, we create a local.conf file
```
cd devstack
echo "
[[local|localrc]]
ADMIN_PASSWORD=$ADMIN_PASSWORD
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
SERVICE_TOKEN=a682f596-76f3-11e3-b3b2-e716f9080d50

HOST_IP=<ipaddres of eth0>
SERVICE_HOST=<ip address of controllers eth0>
MYSQL_HOST=$SERVICE_HOST
RABBIT_HOST=$SERVICE_HOST
Q_HOST=$SERVICE_HOST
GLANCE_HOSTPORT=$SERVICE_HOST:9292

GIT_BASE=${GIT_BASE:-https://git.openstack.org}

# Cells!
ENABLED_SERVICES+=,n-cell
DISABLED_SERVICE+=,n-api,key,g-api

# Neutron - Networking Service
# If Neutron is not declared the old good nova-network will be used
ENABLED_SERVICES+=,q-svc,q-agt,q-dhcp,q-l3,q-meta,neutron,q-lbaas,q-vpn,q-fwaas

# Neutron Stuff
OVS_VLAN_RANGES=RegionOne:1:4000
OVS_ENABLE_TUNNELING=False

# Output
LOGFILE=/opt/stack/logs/stack.sh.log
VERBOSE=True
LOG_COLOR=False
SCREEN_LOGDIR=/opt/stack/logs
" | tee -a /home/stack/devstack/local.conf
```

The important bit for cells is:
```
ENABLED_SERVICES+=,n-cell
DISABLED_SERVICE+=,n-api,key,g-api
```

This enables the cell service and turns of keystone along with nova-api and glance-api services. Those in turn will be handled by the parent.

Next, run ```./stack.sh``` and grab a coffee. When it completes, add the following to ```/etc/nova/nova.conf```

```
[cells]
enable=True
name=cell1
cell_type=comput
```

## Configuring and Creating Cells

Ok, what happened above was we got two nodes up and running and ready to go with Devstack and the prerequsite services for cells. Now we actually have to actuall make the cells. To do that:

### Parent

```
nova-manage cell create --name=cell1 --cell_type=child --username=guest --password=password --hostname=<child ip> --port=5672 --virtual_host=/ --woffset=1.0 --wscale=1.0
```

Then in screen, find the n-cell-* services and restart them.

### Child

```
nova-manage cell create --name=parent --cell_type=parent --username=guest --password=password --hostname=<parent ip> --port=5672 --virtual_host=/ --woffset=1.0 --wscale=1.0
```

Then in screen, find the n-cell-* services and restart them.

## Conclusion

In this post, we created two virtual machines, installed Devstack while enabling nova-cells. Finally, we actually configured the cells to talk to one another. This post was largely an amalgamation of two other posts highlighted in the resources section

## Resources
- [http://openlystacking.blogspot.com/2013/07/creating-cell-environment-using-devstack.html](http://openlystacking.blogspot.com/2013/07/creating-cell-environment-using-devstack.html)
- [http://www.xlcloud.org/bin/view/Blog/Devstack+with+Quantum+in+a+multi-node+configuration?language=en](http://www.xlcloud.org/bin/view/Blog/Devstack+with+Quantum+in+a+multi-node+configuration?language=en)
