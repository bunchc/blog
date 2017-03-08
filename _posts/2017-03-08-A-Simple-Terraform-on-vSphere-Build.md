---
title: "A Simple Terraform on vSphere Build"
date: 2017-03-08
layout: "post"
categories: packer, terraform, devops, vsphere
---

This post will talk a bit about how to use [Terraform](https://www.terraform.io/) to deploy a simple config against vSphere. Simple? Here's what we're building:

* VMs - 
    - OpenSUSE 42.2
        + 2 vCPU
        + 8GB Ram
        + 20GB Disk
    - CentOS7
        + 2 vCPU
        + 8GB Ram
        + 20GB Disk

As with prior posts, I am building this on top of a vSphere lab from [here](https://github.com/lamw/vghetto-vsphere-automated-lab-deployment). Along with the following:

* VMware Fusion 8.5.3
* Terraform 0.8.8
* vSphere 6.5
    - ESXi 6.5a
    - VCSA 6.5

## Defining the Environment

To build our two VM environment, we need to create three files at the root of the directory you plan to build from. These files are:

```
$ ls -l
-rw-r--r--  1 bunchc  staff  1788 Mar  8 16:02 build.tf
-rw-r--r--  1 bunchc  staff   172 Mar  8 15:01 terraform.tfvars
-rw-r--r--  1 bunchc  staff   162 Mar  8 15:01 variables.tf
```

Each of these files has the following use:

* build.tf - defines the infrastructure to build. This includes definitions for VMs, networks, storage, which files to copy where, and then some.
* variables.tf - defines any variables to be used in build.tf
* terraform.tfvars - supplies the actual values for the variables

In the following sections we review each file as it pertains to our environment.

### build.tf

Below I have broken out the sections of build.tf that are of interest to us. If you are following along, you will want to copy/paste each section into a single file.

This first section tells terraform how to connect to vSphere. You will notice there are no actual values provided. These come from variables.tf and terraform.tfvars

```
# Configure the VMware vSphere Provider
provider "vsphere" {
    vsphere_server = "${var.vsphere_vcenter}"
    user = "${var.vsphere_user}"
    password = "${var.vsphere_password}"
    allow_unverified_ssl = true
}
```

The second section defines the OpenSUSE VM. We do this by telling Terraform to create a resource, and then providing the type and name of said resource. The only other thing I will call out in this section is the 'disk' clause. When using 'template' inside the disk clause, you do not need to specify a disk size.

```
# Build openSUSE
resource "vsphere_virtual_machine" "opensuse-openstack" {
    name   = "opensuse-openstack"
    vcpu   = 2
    memory = 8192
    domain = "vghetto.local"
    datacenter = "${var.vsphere_datacenter}"
    cluster = "${var.vsphere_cluster}"

    # Define the Networking settings for the VM
    network_interface {
        label = "VM Network"
        ipv4_gateway = "10.0.1.1"
        ipv4_address = "10.0.1.190"
        ipv4_prefix_length = "24"
    }

    dns_servers = ["10.0.1.10", "8.8.8.8"]

    # Define the Disks and resources. The first disk should include the template.
    disk {
        template = "openSUSE-Leap-42.2-NET-x86_64.vmware"
        datastore = "virtual_machines"
        type ="thin"
    }

    # Define Time Zone
    time_zone = "America/Chicago"
}
```

The third section that follows, defines the second VM. You will see it's a repeat of the first.

```
# Build CentOS
resource "vsphere_virtual_machine" "centos-openstack" {
    name   = "centos-openstack"
    vcpu   = 2
    memory = 8192
    domain = "vghetto.local"
    datacenter = "${var.vsphere_datacenter}"
    cluster = "${var.vsphere_cluster}"

    # Define the Networking settings for the VM
    network_interface {
        label = "VM Network"
        ipv4_gateway = "10.0.1.1"
        ipv4_address = "10.0.1.180"
        ipv4_prefix_length = "24"
    }

    dns_servers = ["10.0.1.10", "8.8.8.8"]

    # Define the Disks and resources. The first disk should include the template.
    disk {
        template = "CentOS-7-x86_64.vmware"
        datastore = "virtual_machines"
        type ="thin"
    }

    # Define Time Zone
    time_zone = "America/Chicago"
}
```

### variables.tf

Next up in the files we need to make is variables.tf. I've provided in wholesale below:

```
# Variables
variable "vsphere_vcenter" {}
variable "vsphere_user" {}
variable "vsphere_password" {}
variable "vsphere_datacenter" {}
variable "vsphere_cluster" {}
```

>**Note:** We are only defining things here, not providing them values just yet.

### terraform.tfvars

Here's the good stuff. That is, this is the file that maps values to the  variables, and in our case, stores credentials. You will want to add this to .gitignore (or whatever your source control uses):

```
vsphere_vcenter = "10.0.1.170"
vsphere_user = "administrator@vghetto.local"
vsphere_password = "VMware1!"
vsphere_datacenter = "Datacenter"
vsphere_cluster = "VSAN-Cluster"
```

Yes, yes, I left my creds in. These are, after-all the default creds for the vGhetto autobuilder.

## Building the Infrastructure

Now that you have all the config files in place there are two steps left, validate and deploy. Fist, we validate:

```
terraform plan
```

If you have everything correct so far, you will see the following:

![terraform plan](https://i.imgur.com/L0K6mhB.png)

Neat! Let's build:

```
terraform apply
```

If you glance in vCenter, you will notice the build has indeed kicked off:

![vCenter screenshot](https://i.imgur.com/l1uW3Mp.png)

Terraform also produces the following output when successful:

![terraform build](https://i.imgur.com/sp1CAng.png)

