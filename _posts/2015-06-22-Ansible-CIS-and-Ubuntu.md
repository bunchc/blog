---
title: "Ansible, CIS, and Ubuntu"
date: 2015-06-22
layout: "post"
categories: 
---

Following on from my [RHEL and CIS with Ansible post](http://blog.codybunch.com/2015/05/15/Using-the-CIS-Ansible-Role-against-CentOSRHEL/), comes qutie a bit of work to proceed down the Ubuntu path in applying CIS benchmarks with Ansible. Before we get too deep, however, it is important to call out that this Ansible role is still based on the RHEL benchmarks, just applied to the applicable systems in Ubuntu. This is because the benchmarks for RHEL have been further developed and harden many parts of the system the Ubuntu benchmarks didn't touch.

To begin with, we'll use the adapted Ansible role from [here](https://github.com/bunchc/ansible-role-cis). Like so:

```git clone https://github.com/bunchc/ansible-role-cis /etc/ansible/roles/cis-ubuntu```

From there, create a playbook.yaml that contains the following:


    - hosts: all
      user: root
      tasks:
        - group_by: key=os_{{ ansible_distribution }}

    - hosts: os_CentOS
      user: root
      roles:
        - cis-centos

    - hosts: os_Ubuntu
      user: root
      roles:
        - cis-ubuntu


Your playbook file contains three sections. The first uses a 'group_by' task to separate hosts out by operating system. The last two sections then apply the right CIS role according to the OS reported back in. 

Finally, apply the playbooks as follows:

```ansbile-playbook -i /etc/ansible/hosts ./playbook.yaml```

