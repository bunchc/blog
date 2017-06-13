---
title: "Upgrading UniFi Controller on UBNT Cloud Key by Hand"
date: 2017-06-12
layout: "post"
categories: UBNT, Unifi, Ubiquiti
---

Today I needed to update the version of the UniFi Controller that runs on my UBNT Cloud Key. To do this, I basically followed the directions [here](https://help.ubnt.com/hc/en-us/articles/216655518-UniFi-How-to-Manually-Change-the-Cloud-Key-s-Controller-Version-via-SSH).

## Upgrading UniFi Controller on UBNT Cloud Key

To do the upgrade:

* Log directly into the CloudKey over ssh
* wget the new firmware
* dpkg -i new_firmware.deb
* Hop over to the web portal for any additional steps

Here is what that looks like from the cli:

```


 ___ ___      .__________.__
|   |   |____ |__\_  ____/__|
|   |   /    \|  ||  __) |  |   (c) 2013-2016
|   |  |   |  \  ||  \   |  |   Ubiquiti Networks, Inc.
|______|___|  /__||__/   |__|
           |_/                  http://www.ubnt.com

     Welcome to UniFi CloudKey!

root@UniFi-CloudKey:~# cd /tmp
root@UniFi-CloudKey:/tmp# wget http://dl.ubnt.com/unifi/4.8.12/unifi_sysvinit_all.deb
--2017-06-12 20:37:51--  http://dl.ubnt.com/unifi/4.8.12/unifi_sysvinit_all.deb
Resolving www.ubnt.com (www.ubnt.com)... 52.24.172.6, 35.162.195.66, 54.69.255.156
Connecting to www.ubnt.com (www.ubnt.com)|52.24.172.6|:443... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: http://dl.ubnt.com/unifi/4.8.12/unifi_sysvinit_all.deb [following]
converted 'http://dl.ubnt.com/unifi/4.8.12/unifi_sysvinit_all.deb' (ANSI_X3.4-1968) -> 'http://dl.ubnt.com/unifi/4.8.12/unifi_sysvinit_all.deb' (UTF-8)
--2017-06-12 20:37:52--  http://dl.ubnt.com/unifi/4.8.12/unifi_sysvinit_all.deb
Resolving dl.ubnt.com (dl.ubnt.com)... 54.230.81.210
Connecting to dl.ubnt.com (dl.ubnt.com)|54.230.81.210|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 145135070 (138M) [application/x-debian-package]
Saving to: 'unifi_sysvinit_all.deb'

unifi_sysvinit_all.deb 100%[===========================>] 138.41M   932KB/s   in 2m 47s

2017-06-12 20:40:39 (851 KB/s) - 'unifi_sysvinit_all.deb' saved [145135070/145135070]

root@UniFi-CloudKey:/tmp# dpkg -i unifi_sysvinit_all.deb
(Reading database ... 15695 files and directories currently installed.)
Preparing to unpack unifi_sysvinit_all.deb ...
Unpacking unifi ...
Setting up unifi ...
Processing triggers for systemd ...
```

Once that wraps up, you can head over to your web portal. If it's anything like  mine, you'll see a DB migration in process:

![Database Migration](https://i.imgur.com/jC3NSiX.png)

After that is done, you can log in.

## Wrapping up the upgrade

If you have other UBNT gear on your network, it too may need an upgrade. For this I tend to work from the outside in. In this case that means first upgrading the AP's, then the switch:

![Rolling AP Upgrade](https://i.imgur.com/uO9vhl3.png)

Rolling updates can be found under the APS tab of the UniFi interface:

![Find the rolling updates](https://i.imgur.com/rX9w72I.png)

With that complete, upgrade the rest as needed.

