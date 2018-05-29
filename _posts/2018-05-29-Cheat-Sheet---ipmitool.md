---
title: "Cheat Sheet - ipmitool"
date: 2018-05-29
layout: "post"
categories: ipmi, ipmitool, cheatsheet, cheat sheet, remote management
---

Some quick commands I've found handy for operating remote systems via ipmitool:

__Check status:__

```
root@lab-c:~# ipmitool -I lan -H 10.127.20.10 -U root chassis status
Password:
System Power         : off
Power Overload       : false
Power Interlock      : inactive
Main Power Fault     : false
Power Control Fault  : false
Power Restore Policy : unknown
Last Power Event     :
Chassis Intrusion    : inactive
Front-Panel Lockout  : inactive
Drive Fault          : false
Cooling/Fan Fault    : false
Sleep Button Disable : allowed
Diag Button Disable  : allowed
Reset Button Disable : allowed
Power Button Disable : allowed
Sleep Button Disabled: false
Diag Button Disabled : false
Reset Button Disabled: false
Power Button Disabled: false
```

__Power Operations:__

Useful here are on, off, soft, cycle, reset:

```
root@lab-c:~# ipmitool -I lan -H 10.127.20.10 -U root power on
Chassis Power Control: Up/On

root@lab-c:~# ipmitool -I lan -H 10.127.20.10 -U root power off
Chassis Power Control: Down/Off

root@lab-c:~# ipmitool -I lan -H 10.127.20.10 -U root power soft

root@lab-c:~# ipmitool -I lan -H 10.127.20.10 -U root power reset
```

__Change to / from pxe boot:__

```
root@lab-c:~# ipmitool -I lan -H 10.127.20.10 -U root chassis bootdev pxe
Set Boot Device to pxe

root@lab-c:~# ipmitool -I lan -H 10.127.20.10 -U root chassis bootdev bios
Set Boot Device to bios
```

__Reset the ipmi controller:__

> Note: This may need to be sent more than once to actually do the thing.

```
root@lab-c:~# ipmitool -I lan -H 10.127.20.10 -U root  mc reset [ warm | cold ]
Sent cold reset command to MC
```

__Set a bunch of hosts to pxe & reboot:__

Here I'll supply two of these. The first will do hosts in parallel, with a random delay up to ```MAXWAIT```. This is for any number of reasons. The primary being to be nice to the power infrastructure where you are performing the resets. It is also a useful snippet for chaos style resets.

```
seq -f "10.127.20.%g" 1 100 | xargs -I X "sleep $((RANDOM % MAXWAIT)) ipmitool -I lan -H X -U root chassis bootdev pxe && ipmitool -I lan -H X -U root power reset"
```

This second option is a bit more yolo, and fires off all the resets ta once.

```
seq -f "10.127.20.%g" 1 100 | xargs -I X "ipmitool -I lan -H X -U root chassis bootdev pxe && ipmitool -I lan -H X -U root power reset"
```