---
title: "packer.io with vSphere"
date: 2017-03-08
layout: "post"
categories: packer, devops, vsphere
---

Packer isn't exactly a new tool. In fact, I've covered using [packer to build Vagrant boxes](http://blog.codybunch.com/2014/10/28/Using-Packer-to-Make-Vagrant-Boxes/) a little while ago. This time around, I'm going to share some notes and the json file I used to get this to build and upload properly to vSphere.

## My Build Environment:

I am running these builds from OSX 10.12.3 with:

* VMware Fusion 8.5.3
* packer.io 0.12.3
* vSphere 6.5 
    - ESXi 6.5a
    - VCSA 6.5

## Packer json template.

The entire json I use is [here](https://gist.github.com/bunchc/6169e81148972b7c5db84a80e209cf87). I have copied the relevant sections below. First, the variables section. You will want to swap these with values specific to your environment. The values I've supplied came from the [vGhetto lab builder.](https://github.com/lamw/vghetto-vsphere-automated-lab-deployment)

```
    "variables": {
        "vsphere_host": "vcenter65-1.vghetto.local",
        "vsphere_user": "administrator@vghetto.local",
        "vsphere_pass": "VMware1!",
        "vsphere_datacenter": "Datacenter",
        "vsphere_cluster": "\"VSAN-Cluster\"",
        "vsphere_datastore": "virtual_machines",
        "vsphere_network": "\"VM Network\""
    },
```

Next, post-processors. Here be the magic. 

![post-processors](https://i.imgur.com/kiTHcph.png)

Some highlights:

* type - tells packer we're uploading to vsphere
* keep_input_artifact - setting this to true helps troubleshooting
* only - tells packer to only run this post-processor for the named builds.
* the remaining lines - the vSphere specific variables from the prior section.

>**Note:** Only change the variables rather than specifying names directly. Otherwise, OVFTool will get stupid angry about escaping characters.

## The Packer to vSphere Build

Once you have all the parts in place, you can run the following command to kick off the packer build that will dump it's artifacts into vSphere:

```
packer build -parallel=false ubuntu-14.04.json
```

Now, the packer command will produce a *LOT* of output, even without debugging enabled. If you would like to review said output or dump it to a file in case sometehing goes sideways:

```
time { packer build -parallel=false ubuntu-14.04.json; } 2>&1 | tee -a /tmp/packer.log
```

This will time how long packer takes to do it's thing and dumps all output to /tmp/packer.log

When the command finishes you'll see the following output:

```
==> ubuntu-14.04.amd64.vmware: Running post-processor: vsphere
    ubuntu-14.04.amd64.vmware (vsphere): Uploading output-ubuntu-14.04.amd64.vmware/packer-ubuntu-14.04.amd64.vmware.vmx to vSphere
Build 'ubuntu-14.04.amd64.vmware' finished.
```

![vSphere client](https://i.imgur.com/7GrGzK2.png)

With that, all should be in working order.