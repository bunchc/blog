---
title: "[OSX] Format > 32GB SD Card for Rasperry Pi"
date: 2016-01-25
layout: "post"
categories: osx, raspberry pi, rpi, embedded
---

Apparently, one cannot just use an SD Card larger than 32GB with the rPI. Rather, you need to partition it down to size first, otherwise the bootloader gets angry at you. The generall process then, is to 'overwrite' format the SD card, then create a 32GB partition and format it FAT32.

## Find the disk

```
$ diskutil list
... snip ...
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *63.9 GB    disk2
   1:                 DOS_FAT_32 PI                      32.0 GB    disk2s1
   2:                 DOS_FAT_32 DATA                    31.9 GB    disk2s2

```

In our case, it's /dev/disk2.

## Erase & Format the Disk

With that information in hand, we first erase the disk:

```
sudo diskutil unmountDisk /dev/disk2
sudo diskutil zeroDisk force /dev/disk2
```

What we did there was first unmount the disk, and then zero it out. We used the force parameter to ensure this goes off without too many a hitch.

Now that it's been nuked from orbit, format the thing:

```
sudo diskutil unmountDisk /dev/disk2
sudo diskutil partitionDisk /dev/disk2 MBR FAT32 PI 32g FAT32 DATA 0b
```

This time, we again unmounted the disk. After that we told diskUtil to partition the drive. Specifically we told it to use an MBR type partition table, and that our first partition was FAT32, and should be named PI, with a size of 32GB. The second partition we specified in much the same way, with one exception, we specified 0b, which tells diskutil to use the remainder of the disk.
