---
title: "Clone all the USB keys! - Method 3"
date: 2014-11-03
categories: 
---

Method 3 you ask? Wel, sure, I [wrote on this around](http://openstack.prov12n.com/usb-key-duplication-on-osx-on-the-cheap/) the time of OSCON earlier this year. However, for the OpenStack in Paris, we needed a larger axe. Thanks to [Lars](https://twitter.com/larsbutler) & [Mr. Cloud Null](https://github.com/cloudnull), we managed to apply the appropriate amount of strategic violence:

```
cat ~/fuckshitup.sh
#!/bin/bash
set -e -o -v

for i in `jot 19 3`;
    do sudo diskutil erasedisk FAT32 PARIS /dev/disk${i}
done

for i in `jot 19 3`;
    do sudo diskutil partitionDisk /dev/disk${i} MBR MS-DOS PARIS 0b
done

for i in `jot 19 3`;
    do rsync --progress -az /Volumes/PARIS "/Volumes/PARIS $i/" &
done
```

First it formats all the disks, then it rsyncs all the things.