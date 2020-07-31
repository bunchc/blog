---
title: "Fixing cgroup memory on Raspbian Buster for Kernel 5.4.x"
date: 2020-07-31
layout: "post"
categories: "rpi, raspberry pi, raspbian, IoT, k3s, k8s"
---

After running `sudo apt -y full-upgrade` my Raspberry Pi k3s cluster got upgrade from 4.x kernels to 5.x. Which turbo-broke k3s.

## The Frror

After rebooting into the new kernel and k3s not working, I found the following error in both `/var/log/syslog/` and `journalctl -xeu k3s` (or for the worker nodes `journalctl -xeu k3s-agent`)

```shell
level=fatal msg="failed to find memory cgroup, you may need to add \"cgroup_memory=1 cgroup_enable=memory\" to your linux cmdline (/boot/cmdline.txt on a Raspberry 
Pi)"
```

However, those values were totally in `/boot/cmdline.txt`:

```shell
$ cat /boot/cmdline.txt
console=serial0,115200 console=tty1 root=PARTUUID=dd5ac5d2-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait cgroup_enable=1 cgroup_memory=1 swapaccount=1 cgroup_enable=memory dwc_otg.lpm_enable=0 
```

What could it be then?

## The Fix

This fix, of course, is to update the firmware on the pi via `rpi-update`

>At the time of this writing, the fix was released 4 days ago according to this [Github issue](https://github.com/raspberrypi/linux/issues/3644).

While you can run `rpi-update` by itself and answer its warning prompt, that can get tedious for more than one or two nodes. Since I was upgrading nine nodes, something more robust was needed. Something like the following command:

```shell
$ sudo PRUNE_MODULES=1 RPI_REBOOT=1 SKIP_WARNING=1 rpi-update
```

Or for those using Ansible to manage their fleet:

```shell
ansible -i swarm.yml nodes --become -m shell -a 'sudo PRUNE_MODULES=1 RPI_REBOOT=1 SKIP_WARNING=1 rpi-update'
```

> **Note:** You should probably do at least one node the manual way to ensure the update works. The commands shown here can... *accelerate* your journey to serverless.