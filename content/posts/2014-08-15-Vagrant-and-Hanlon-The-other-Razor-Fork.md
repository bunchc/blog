---
title: "Vagrant and Hanlon (The other Razor Fork)"
date: 2014-08-15
categories: 
---

You may or may not know that Razor, the bare-metal lifecycle tool, has forked from its original bits. This is a good thing, but it's still really early days on the new forks, so we'll see ultimately what plays out.

The Puppet-Labs "Razor-Server" fork can be found [here](https://github.com/puppetlabs/razor-server). If you want to work with it, or test it locally, you can use the bits written by Egle [here](http://anystacker.com/2014/01/vagrant-up-razor-server/).

Hanlon, the CSC fork [here](https://github.com/csc/Hanlon), is being perused by Tom McSweeney who helped get the original razor off the ground.

To help me work with it and contribute some, I've built a small Vagrant environment (largely basd on the Vagrant environment for 'the other' razor.).

## Vagrant Up Hanlon

To get started, clone the repo and vagrant up, like so:

```
git clone https://github.com/bunchc/vagrant-hanlon.git
cd vagrant-hanlon
vagrant up
```

What this does during the vagrant up process is:

- Install dnsmasq
- Configure IPtables for NAT
- Installs Mongodb
- Installs Java
- Installs RBENV
- Installs both jruby and ruby 1.9.3
- Downloads Hanlon
- Starts Puma (to run Hanlon)
- Pulls down a few images to add to Hanlon

Once you have Hanlon running, command wise it works very similar to the old Razor. That is:

```
hanlon node
hanlon policy
hanlon image
```

## Summary

In this post, you cloned and started to work with Hanlon, one of the derivatives of the Razor bare-metal provisioning framework.
