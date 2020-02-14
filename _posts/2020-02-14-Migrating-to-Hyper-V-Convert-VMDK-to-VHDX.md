---
title: "Migrating to Hyper-V: Convert VMDK to VHDX"
date: 2020-02-14
layout: "post"
categories: vmware, hyper-v, windows, win10, migration, qemu-img
---

I have been on a desktop OS and tools journey for the last six or nine months. Long story short, my 2012 Retina MacBook Pro gave up the ghost in a most unpleasant way. I grabbed a System76 laptop, and gave Linux on the desktop a shot for a little while. Eventually I repaved that and moved to Windows 10 full time. As part of that, I enabled WSL 2, which breaks VMware desktop products. No biggie, one can convert VMs, right?

Well...

## Requirements

It is assumed that you have the following:

* Win10
* WSL 2 [1]
* Ubuntu (in WSL that is)

> **Note:** You can get `qemu-img` as a native tool for Windows. I used the Linux version as I had a bit more experience with it.

## Process

There are three basic parts of this process: Converting the VMDK, Importing into Hyper-V, and Installing integration services.

### Converting from VMDK to VHDX

This stage will be done entirely within WSL/Ubuntu, so open a terminal and then use the following commands:

1. Ensure you have `qemu-img`:

```shell
sudo apt update
sudo apt install qemu-utils
# --output truncated-- #
```

2. Convert the image:

```shell
PathToVMDK="/mnt/c/Users/bunchc/vms/win7/win7.vmdk"
PathToVHDX="/mnt/c/Users/bunchc/vms/win7/win7.vhdx"

$ qemu-img convert $PathToVMDK -O vhdx $PathToVHDX -p
```

The command tells `qemu-img` to convert the VMDK specified in `PathToVMDK` with an output type (`-O vhdx`) of vhdx and store the resulting image at `PathToVHDX`. The `-p` tells `qemu-img` to display the progress as it converts. You will need to change the `PathTo` variables to reflect your environment.

> **Note:** There are some tools that will do this in PowerShell [2] [3]. However, they can be rather flaky depending on the source VMDK. `qemu-img` was quite a bit more robust in this regard.

### Importing into Hyper=V

Once the conversion is complete, you can import the VM into Hyper-V as you would any other vhdx. You will want to be careful when selecting which generation, that is, unless you installed the VM with UEFI, you will want to select 'Generation 1'.

Once Imported, edit the settings of the VM.

When happy with the settings power the VM on, log in, uninstall VMware Tools and reboot, then prepare to install Hyper-V integration services.

### Installing integration services

The Hyper-V integration services will generally be installed seamlessly via Windows Update. If this is not the case, or you are using an older version of Windows, download the integration tools [4], and then use the following command from an elevated command prompt to install them:

```powershell
dism.exe /Online /Add-Package /PackagePath:C:\Users\bunchc\Desktop\windows6.x-hypervintegrationservices-x86.cab
```

## Resources

The following pages were used to help make this post:

1. [Installing WSL-2 on Win10.](https://docs.microsoft.com/en-us/windows/wsl/wsl2-install)
2. [Microsoft Virtual Machine Conversion Kit.](https://www.microsoft.com/en-us/download/details.aspx?id=42497)
3. [Convert a VMware VM to Hyper-V.](https://lazyadmin.nl/it/convert-a-vmware-vmdk-to-hyper-v-vhd/)
4. [Download the integration services package.](https://support.microsoft.com/en-us/help/3063109/hyper-v-integration-components-update-for-windows-virtual-machines)
5. [Installing Hyper-V integration services manually.](https://blogs.technet.microsoft.com/virtualization/2015/07/24/integration-components-available-for-virtual-machines-not-connected-to-windows-update/)
