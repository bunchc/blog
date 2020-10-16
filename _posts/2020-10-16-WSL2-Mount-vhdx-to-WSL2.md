---
title: "[WSL2] Mount vhdx to WSL2"
date: 2020-10-16
layout: "post"
categories: "windows, win10, wsl, wsl2, windows subsystem for linux, troubleshooting"
---

Sometimes the best blog posts come from Bad Ideas&trade;. Like this one:

![Screenshot of a tweet that reads: "Why is my AppData folder 54GB? o.O Now to find out how badly Win10 breaks if I move it.](https://pbs.twimg.com/media/EkYbxAWXsAEWc_e?format=jpg&name=large)

While the cleanup from the experimental moving of AppData is the subject of another post, one thing that didn't come back properly was my default WSL distribution. This post shows how I mounted the `ext4.vhdx` to a *different* WSL distribution to export the data.

## Mount additional vhdx to WSL2

> **Note:** Do not use this for anything other than data recovery. Even then, there are likely better methods. Why? This method uses FUSE mounts which are s-l-o-w, and in general works against the seamlessness that WSL aims for.

In order to mount a VHD or VHDX file to WSL, you need to first install some tools, and, likely a Kernel.

```shell
$ sudo apt update
$ sudo apt install -y libguestfs-tools linux-image-generic
```

The kernel also needs to be accessible by our user:

```shell
$ sudo chmod 0777 /boot/initrd.img-5.4.0-51-generic
$ sudo chmod 0777 /boot/vmlinuz-5.4.0-51-generic
```

Once your tools are installed, have a look at the partitions and filesystems on the `vhdx` using `virt-filesystems`:

```shell
$ virt-filesystems -a /mnt/c/oldUbuntu/ext4.vhdx -l
Name      Type        VFS   Label  Size          Parent
/dev/sda  filesystem  ext4  -      274877906944  -   
```

Cool, as you see from the output, there is one disk `/dev/sda` with an `ext4` filesystem on it. Next, we make a spot to mount it to, and then, mount it using `guestmount`:

```shell
$ cd ~
$ mkdir import
$ guestmount \
  --add /mnt/c/oldUbuntu/ext4.vhdx \
  --ro \
  -i ./import/
```

The `guestmount` command above adds (`--add`) an image (`/mnt/c/oldUbuntu/ext4.vhdx`), specifies we want it read-only (`--ro`), and to inspect it (`-i`) for filesystems to mount to our destination (`./import/`).

> **Note:** This failed for me the first few times until I had a kernel installed in `/boot` that was readable by the user.

Now we can check our mounts and confirm that it is indeed mounted, and even browse the filesystem if we'd like:

```shell
$ mount
...
/dev/fuse on /home/bunchc/import type fuse (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)

$ ls import/
bin   dev  home  lib    lost+found  mnt  proc  run   snap  sys usr
boot  etc  init  lib64  media       opt  root  sbin  srv   tmp  var  
```

## Summary

In this post we used `guestmount` to mount a `vhdx` into WSL2 for data recovery. While it is possible to do this for mounting and using secondary disk files, in that case you likely just want a Virtual Machine. It will be faster and more stable.

## References

* [https://github.com/microsoft/WSL/issues/4574](https://github.com/microsoft/WSL/issues/4574)
* [xmount](https://www.mankier.com/1/xmount)
* [https://stackoverflow.com/questions/36819474/how-can-i-attach-a-vhdx-or-vhd-file-in-linux](https://stackoverflow.com/questions/36819474/how-can-i-attach-a-vhdx-or-vhd-file-in-linux)
* [https://libguestfs.org/guestmount.1.html](https://libguestfs.org/guestmount.1.html)
* [https://libguestfs.org/virt-filesystems.1.html](https://libguestfs.org/virt-filesystems.1.html)
* [https://stackoverflow.com/questions/11395134/how-to-make-a-tar-backup-of-a-root-filesystem](https://stackoverflow.com/questions/11395134/how-to-make-a-tar-backup-of-a-root-filesystem)