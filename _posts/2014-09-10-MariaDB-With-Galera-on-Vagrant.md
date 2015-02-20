---
title: "MariaDB With Galera on Vagrant"
date: 2014-09-10
categories: 
---

Found myself in some training this last week using MariaDB, and being that I like to get a bit more hands on than most, using the class provided lab environment wasn't going to cut it. This meant wrapping some scripting into a Vagrant environment so I could reliably reporduce the three node lab.

## Getting Started

You'll need Vagrant and Git. It's also preferred that you have vagrant-cachier installed. You should have vagrant-cachier anyways, but alas, that is not for right now. Once you have these out of the way, do the following:

1. ```git clone https://github.com/bunchc/mariadb-galera-vagrant.git```
2. ```cd mariadb-galera-vagrant```
3. ```vagrant up```

## Validating the Cluster

After a few minutes, you should be able to log into any of the nodes. Specifically, node-01 will be used to 'bootstrap' the cluster, the other two nodes will join from there.

To validate the cluster:

```
vagrant ssh mariadb-02
sudo su -
mysql -uroot
SHOW GLOBAL STATUS LIKE 'wsrep%';
```

You should be able to do the above from any of the nodes in the cluster.