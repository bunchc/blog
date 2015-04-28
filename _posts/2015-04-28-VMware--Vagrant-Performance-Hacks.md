---
title: "VMware & Vagrant Performance Hacks"
date: 2015-04-28
layout: "post"
categories: 
---

I believe I've posted on this before, however I think I deleted the gist that contained these bits of info.

Here are some Vagrantfile settings specific to VMware desktop products that will help you eek out a bit more performance.


    # VMware Fusion / Workstation
    config.vm.provider :vmware_fusion or config.vm.provider :vmware_workstation do |vmware, override|
    override.vm.synced_folder ".", "/vagrant", type: "nfs"

    # Fusion Performance Hacks
    vmware.vmx["logging"] = "FALSE"
    vmware.vmx["MemTrimRate"] = "0"
    vmware.vmx["MemAllowAutoScaleDown"] = "FALSE"
    vmware.vmx["mainMem.backing"] = "swap"
    vmware.vmx["sched.mem.pshare.enable"] = "FALSE"
    vmware.vmx["snapshot.disabled"] = "TRUE"
    vmware.vmx["isolation.tools.unity.disable"] = "TRUE"
    vmware.vmx["unity.allowCompostingInGuest"] = "FALSE"
    vmware.vmx["unity.enableLaunchMenu"] = "FALSE"
    vmware.vmx["unity.showBadges"] = "FALSE"
    vmware.vmx["unity.showBorders"] = "FALSE"
    vmware.vmx["unity.wasCapable"] = "FALSE"
    vmware.vmx["vhv.enable"] = "TRUE"
    end


The basics of what we're doing is as follows:
- Using NFS shared folders
- Turning off logging
- Tuning memory settings
- Turning off unity

Hope this saves you some time.