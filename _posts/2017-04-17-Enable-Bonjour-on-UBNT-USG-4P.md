---
title: "Enable Bonjour on UBNT USG-4P"
date: 2017-04-17
layout: "post"
categories: ubnt, ubiquiti, yoloops
---

This here is super hacky, but it works.

# Problem:

After segmenting the home network (kids, guest, random iot devices, etc), Bonjour stopped working.

# Solution

The proposed solution is to SSH to the Security Gateway and run the following:

```
configure
set service mdns reflector
commit
save
exit
```

This works, great! Right? The problem with the above method, is that you are broadcasting your stuff out on the WAN Interface. I don't know about you, but I don't like the idea of what might show up on my AppleTV.

## A better solution:

We start as prescribed. This makes sure the right services are installed and the config files are in place:

```
configure
set service mdns reflector
commit
save
exit
```

Now, undo that:

```
configure
delete service mdns
commit
save
exit
```

Now, edit ```/etc/avahi/avahi-daemon.conf``` with your corresponding interfaces. The relevant sections of my file look like this:

```
[server]
...
allow-interfaces=eth0,eth0.20,eth0.40,eth0.50,eth0.60
...

[reflector]
enable-reflector=yes
```

Finally, we add a script to ensure these services start on reboot & start them now:

```
echo "#!/bin/bash -
#title          :bonjour-fix.sh
#description    :Starts dbus and avahi on reboot on the USG-4P
#============================================================================

for service in dbus avahi-daemon; do {
    if (( $(sudo ps -ef | grep -v grep | grep "${service}" | wc -l) > 0 )); then {
        echo "[+] ${service} is running"
    } else {
        echo "[i] Attempting to start ${service}"
        sudo /etc/init.d/"${service}" start
    } fi
} done" | sudo tee /config/scripts/post-config.d/bonjour-fix.sh

sudo chmod +x /config/scripts/post-config.d/bonjour-fix.sh

sudo /etc/init.d/dbus start
sudo /etc/init.d/avahi-daemon start

```


# Resources

This solution was put together from a number of forum posts:

* [https://community.ubnt.com/t5/EdgeMAX/run-mDNS-without-the-enabling-the-reflector/td-p/997805](https://community.ubnt.com/t5/EdgeMAX/run-mDNS-without-the-enabling-the-reflector/td-p/997805)
* [https://community.ubnt.com/t5/EdgeMAX/AppleTV-discovery-across-multiple-networks/m-p/784088](https://community.ubnt.com/t5/EdgeMAX/AppleTV-discovery-across-multiple-networks/m-p/784088)
* [https://community.ubnt.com/t5/EdgeMAX-Feature-Requests/Interface-selection-for-mdns-reflector/idi-p/823164](https://community.ubnt.com/t5/EdgeMAX-Feature-Requests/Interface-selection-for-mdns-reflector/idi-p/823164)