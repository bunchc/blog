---
title: "Getting Started with Ansible and vSphere"
date: 2020-03-06
layout: "post"
categories: "vSphere, DevOps, Ansible, automation, vmware"
---

My work has started to shift around some recently and I find myself returning to VMware products after a hiatus. In addition to running into a [blog post I wrote a decade ago](https://vbrownbag.com/2010/09/powercli-one-liner-rescan-all-hbas/), it has been interesting to see how else VMware has kept up.

In particular, I have spent A LOT of time automating various things with [Ansible](https://www.ansible.com/), so discovering the extensive list of [VMware modules](https://docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#vmware). This post shows how to use the Ansible VMware modules to launch your first VM.

## Getting Started

There are quite a few prerequisites to getting this going. Buckle up!

First, the requirements for your 'Control Node', where you will run Ansible from:

* Ansible
* Python
* Pyvmomi
* vSphere Automation Python SDK
* Access to vSphere

### Installing the prerequisites on Ubuntu 18.04

> **Note:** While you can likely install these tools directly on Windows, I've found it easier to use Ubuntu by way of WSL2.

1. Install Ansible:

```shell
root@9a45e927fc78:/# apt update -q
root@9a45e927fc78:/# apt install -y software-properties-common
root@9a45e927fc78:/# add-apt-repository --yes --update ppa:ansible/ansible
root@9a45e927fc78:/# apt install -y ansible
```

2. Install / update Python

```shell
root@9a45e927fc78:/# apt install -y python-minimal python-pip
```

3. Install Pyvmomi

```shell
root@9a45e927fc78:/# pip install pyvmomi
```

4. Install the VMware Python SDK

```shell
root@9a45e927fc78:/# pip install --upgrade git+https://github.com/vmware/vsphere-automation-sdk-python.git
```

## Creating a VM with Ansible

With everything we need installed, we can now make the VM with Ansible. We do that by creating an Ansible playbook, and then running it. An Ansible playbook is a YAML file that describes what you would like done. To create yours, open your favorite text editor and paste the following YAML into it.

```yaml
# new-vm-playbook.yml
- name: Create New VM Playbook
  hosts: localhost
  gather_facts: no
  tasks:
  - name: Clone the template
    vmware_guest:
      hostname: "vcenter.codybunch.lab"
      username: "administrator@codybunch.lab"
      password: "ultra-Secret_P@%%w0rd"
      resource_pool: "Resource Pool to create VM in"
      datacenter: "Name of the Datacenter to create the VM in"
      folder: "{{ datacenter }}/vm"
      cluster: "MyCluster"
      networks:
          - name: "Network 1"
      hardware:
          num_cpus: 2
          memory_mb: 4096
      validate_certs: False
      name: "new-vm-from-ansible"
      template: "centos-7-vsphere"
      datastore: "iscsi-datastore"
      state: poweredon
      wait_for_ip_address: yes
    register: new_vm
```

The great thing about describing the changes in this way, is that the file itself is rather readable. That said, we should go over what running this playbook will actually do.

The playbook tells Ansible to execute on the localhost (`hosts: localhost`). You can change this to be any host that can access vCenter and has pyVmomi installed. It then tells Ansible we would like to build a vm (`vmware_guest`) and supplies the details to make that happen. Make note of the `folder: "{{ datacenter }}/vm"` setting which will place the VM at the root of the datacenter. As with the other variables, change this to fit your environment.

Once you have adjusted the playbook to suit your environment, it can be run as follows:

```bash
ansible-playbook new-vm-playbook.yml
```

## Resources

* [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
* [vSphere SSL Certs for Ansible](https://docs.ansible.com/ansible/latest/scenario_guides/vmware_scenarios/vmware_requirements.html)
* [Install VMware Python modules](https://docs.ansible.com/ansible/latest/scenario_guides/vmware_scenarios/vmware_intro.html)