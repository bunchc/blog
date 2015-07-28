---
title: "Pi-Hole in the cloud"
date: 2015-07-28
layout: "post"
categories: 
---

Pi Hole? Cloud? What?

The short intro is that the Pi-Hole is a Raspberry Pi project designed to set your rPI up as a DNS server that blacklists and blocks ADs. This is great and all, but rather limiting if one does not have a Pi (or their Pi is currently in use in about a million other ways).

## Getting started

Before we get too deep into things, you'll need to get some requirements into place first. Specifically, you'll need some manner of Ubuntu 14.04 host. As I've stated we're doing this in the cloud, I'll be using an 8GB 'standard' instance for this.

Once you have the host available, log in and run apt-get update/upgrade:

```sudo apt-get update && sudo apt-get upgrade -y```

As an optional next step, harden the instance some:

```curl https://gist.githubusercontent.com/bunchc/fa5787f9a398ee0c70e1/raw/265dc8a61ce63e53e0f97eb0099a7bd2fd0d71c8/user_data_hardening.sh | sudo bash```

**Note:** Go read that script first. Because:
![https://pbs.twimg.com/media/CHG5xIbWcAAzBPL.png](https://pbs.twimg.com/media/CHG5xIbWcAAzBPL.png)
HT @[nonapeptide](https://twitter.com/Nonapeptide/)
## Do the thing

The actual do things part here is two fold. First is to install the Pi-Hole bits onto your cloud instance. The second is to then set said cloud instance as your DNS server.

### Installing 'Pi-Hole'

This step is relatively straight forward, but involves that nasty curl | sudo bash bit again, so again I strongly **strongly** recommend reading the script. At a high level the script does a few things:

- Installs OS updates
- Installs dnsmasq (DNS server)
- Installs lighttpd (http server)
- Stops said services for modification
- Backs up the original config files
- Copies new ones into place
- Creates and runs gravity.sh
- Creates chrometer.sh

```gravity.sh``` is where the magic happens. This script, found [here](https://github.com/jacobsalmela/pi-hole/blob/master/gravity.sh). Said script grabs and parses a few rather large lists of ad serving domains, and adds them to your DNS blacklist.

For more, and very specific details, the Jacob, who created the Pi-Hole, has an [excellent write-up.](http://jacobsalmela.com/block-millions-ads-network-wide-with-a-raspberry-pi-hole-2-0/)

### Some additional niceties

Now that you've got Pi-Hole installed, there are a few things I do to keep things updated & humming along. That is, running ```gravity.sh``` once is a good start, but in the arms race of Internet advertising, one needs to keep moving. To do this, I create a cronjob to run gravity.sh weekly, make sure we're up to date:

```
sudo echo "47 6    * * 7   root    /usr/local/bin/gravity.sh" >> /etc/crontab
```

## Using the Pi-Hole

Setting your DNS server should be relatively straight forward. However, to ensure you do not forget a device or two, and capture all other devices as they come and go, I've found it preferential to configure this on your edge device (Say your Wifi router or so) .

## Summary

In this post we showed you how to use the Pi-Hole setup, but instead of on a Raspberry Pi, we built it on a generic Ubuntu 14.04 box "in the cloud" so it'll be available to you everywhere.