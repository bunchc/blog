---
title: "Using the CIS Ansible Role against CentOS/RHEL"
date: 2015-05-15
layout: "post"
categories: 
---

Nothing much new here, excepting before a day or so ago, I was completely unfamiliar with Ansible. Now I'm slightly less so, but felt this needed to be written down before I forgot. Who knows, maybe it'll help you as well.

## Center for Internet Security Benchmarks & You

The [CIS](http://www.cisecurity.org/) puts out a number of [testable security benchmarks](http://benchmarks.cisecurity.org/). If you've not read one of these, there are some... 400 or so items that are scored as part of the benchmark. The benchmark also has various levels, and when you follow the guidance, will allow you to produce some manner of attestation that you have followed the guide and produced a reasonably secure system. At least, that's the idea.

In the world where the productivity of an operator or administrator (when did that pendulum swing back?) is measured in the **tens of thousands** per admin on staff, and with these being 'cattle' and being cycled constantly, hardening these against each of these checks gets to be... interesting.

## Using a Config Management Tool

As I've written on [before](http://blog.codybunch.com/2015/05/12/Update-Userdata-Hardening-Script/) this can be handled at in userdata. Additionally, you can do this in a very OpenStack way using [Heat](http://openstack.prov12n.com/basic-hardening-part-2-using-heat-templates/). This can also be done in a config management tool of your choice, like [Salt](http://blog.codybunch.com/2015/01/09/Basic-Server-Hardening-with-Salt/).

## Using Ansible-Role-CIS

In this case, and after a long winded way of getting here, we're going to use the Ansible CIS role from [here](https://github.com/major/ansible-role-cis).

Now assuming you have the various ssh bits taken care of per the [ansible docs](http://docs.ansible.com/intro_getting_started.html), you will need to do the following:

On the box to receive the role:

```
sudo yum install -y libselinux-python
```

On the 'control' node, you will need to create a playbook.yaml as follows:

```
- hosts: all
  user: root
  roles:
    - cis
```

Once that's saved, you can run the following:

```
ansbile-playbook -i /etc/ansible/hosts -C ./playbook.yaml
```

This will run the playbook in a 'read only' or test mode and will tell you what is out of compliance and where. To push the changes, remove the -C, like this:

```
ansbile-playbook -i /etc/ansible/hosts ./playbook.yaml
```

# Summary

In this post we've talked about the CIS hardening benchmarks as well as some practices for how to harden cloud servers in an automated way. We wrapped up showing you how to do this using Ansible against a RHEL or CentOS style server.

# Acknowledgment!

Many thanks to [Major Hayden](https://major.io/) for getting this out there.