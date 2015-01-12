---
title: "Basic Server Hardening with Salt"
date: 2015-01-09
categories: 
---

Before we get started, take a gander at my last few posts on this, [here](http://openstack.prov12n.com/basic-hardening-part-2-using-heat-templates/) and [here](http://openstack.prov12n.com/basic-hardening-with-user-data-cloud-init/).

The idea here is roughly the same. That is, build a small, basic 'base' profile, template, state, or whatever, that has some simple hardening bits applied. The idea being to give you a reasonable start in turn letting you apply additional layers down the road.

The reason you'd move away from doing this with OpenStack Orchestration (Heat) and into a config management tool is that it allows you to apply the same practices more generically. That is not everyone runs OpenStack, but lots of folks are moving to some flavor of configuration management tool, if they weren't there already. You can then include these SaltStates in Heat or whatever orchestration tool of choice.

## Basic Hardening with SaltStack

For the sake of not making this a *huge-tastic* blog post, we'll skip the part where I explain the what and how of getting started with SaltStack. Many others have done better than I. What follows is my top.sls, secureserver.sls and sysctl.sys files.

### top.sls

This file controls what Salt minions get what 'states'. For securing *all* the servers, that's pretty straight forward. I would like to call out, however, the ordering of the states:

```
base:
  '*':
    - sysctl
    - secureserver
```

This defines a 'base' state that will match for all Salt minions. (Minion is Salt for agent.) It then states to apply first the sysctl state, then the secureserver state.

### sysctl.sls

There isn't much fancy to this file. In fact, it's more or less a YAML formatted version of what you'd throw into ```/etc/sysctl.conf```. What the lines below do is turn off IP routing, ignore broadcasts, responses, and all manner of other fun icmp stuff.

```
net.ipv4.icmp_echo_ignore_broadcasts:
  sysctl.present:
    - value: 1

net.ipv4.icmp_ignore_bogus_error_responses:
  sysctl.present:
  - value: 1

net.ipv4.tcp_syncookies:
  sysctl.present:
    - value: 1

net.ipv4.conf.all.log_martians:
  sysctl.present:
  - value: 1

net.ipv4.conf.default.log_martians:
  sysctl.present:
    - value: 1

net.ipv4.conf.all.accept_source_route:
  sysctl.present:
    - value: 0

net.ipv4.conf.default.accept_source_route:
  sysctl.present:
    - value: 0

net.ipv4.conf.all.rp_filter:
  sysctl.present:
    - value: 1

net.ipv4.conf.default.rp_filter:
  sysctl.present:
    - value: 1

net.ipv4.conf.all.accept_redirects:
  sysctl.present:
    - value: 0

net.ipv4.conf.default.accept_redirects:
  sysctl.present:
    - value: 0

net.ipv4.conf.all.secure_redirects:
  sysctl.present:
    - value: 0

net.ipv4.conf.default.secure_redirects:
  sysctl.present:
    - value: 0

net.ipv4.ip_forward:
  sysctl.present:
    - value: 0

net.ipv4.conf.all.send_redirects:
    sysctl.present:
    - value: 0

net.ipv4.conf.default.send_redirects:
  sysctl.present:
    - value: 0
```

### secureserver.sls

This file, unlike the sysctl bits, actually does some install and config bits. If you've read the post I mentioned at the beginning of this post, you'll recognize the packages being installed.

The configure bits are two fold: 1) We configure logwatch using a file hosted on the Salt master; 2) We configure IP tables to allow ssh and deny all the other things:

```
iptables:
  pkg:
    - installed

denyhosts:
  pkg:
    - installed

psad:
  pkg:
    - installed

fail2ban:
  pkg:
    - installed

logwatch:
  pkg:
    - installed

aide:
  pkg:
    - installed

/etc/cron.daily/00logwatch:
  file:
    - managed
    - source: salt://cron.daily/00logwatch
    - require:
      - pkg: logwatch

ssh:
  iptables.append:
    - table: filter
    - chain: INPUT
    - jump: ACCEPT
    - match: state
    - connstate: NEW
    - dport: 22
    - proto: tcp
    - sport: 1025:65535
    - save: True

allow established:
  iptables.append:
    - table: filter
    - chain: INPUT
    - match: state
    - connstate: ESTABLISHED
    - jump: ACCEPT

default to reject:
  iptables.append:
    - table: filter
    - chain: INPUT
    - jump: REJECT
```

## Summary

In this post we've covered how to do some very very basic hardening of your server using SaltStack. This likely wont work for all circumstances (like if you're going to actually run nginx or apache, you need to add those ports accordingly).

## Resources
- [https://www.thefanclub.co.za/how-to/how-secure-ubuntu-1204-lts-server-part-1-basics](https://www.thefanclub.co.za/how-to/how-secure-ubuntu-1204-lts-server-part-1-basics)
- [http://openstack.prov12n.com/basic-hardening-with-user-data-cloud-init/](http://openstack.prov12n.com/basic-hardening-with-user-data-cloud-init/)
- [http://plusbryan.com/my-first-5-minutes-on-a-server-or-essential-security-for-linux-servers](http://plusbryan.com/my-first-5-minutes-on-a-server-or-essential-security-for-linux-servers)