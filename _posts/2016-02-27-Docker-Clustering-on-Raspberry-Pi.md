---
title: "Docker Clustering on Raspberry Pi"
date: 2016-02-27
layout: "post"
categories: IoT, Raspberry Pi, Docker, Containers
---

The folks behind [Hypriot](http://blog.hypriot.com/) have made getting Docker up and going on the Raspberry Pi a near on trivial task: Download the image, power it on. They've also done the same thing for Docker clustering, [here](http://blog.hypriot.com/post/introducing-hypriot-cluster-lab-docker-clustering-as-easy-as-it-gets/). Which is great, until it isn't.

That is, their clustering lab has a number of hard requirements around VLAN networking support. If you've got a plethora of dumb switches at home, this just wont do!

Have no fear, however, the combination of their image, and their clustering packages seem to be the magical combination.

## Hardware bits to have

- Some number of Raspberry Pi's (I've got 3x rPI 2 for this, I think anything b+ will work)
- At least one wifi dongle
- An Ethernet switch

## Let's do this!

This process roughly breaks down like this:

Pull the image down from [here.](http://blog.hypriot.com/downloads/) At the time of writing it was "Hector" 0.6.1

Use the Hypriot ['flash' script](https://github.com/hypriot/flash) to load your SD card:

```flash --hostname Drekkar --ssid WiFi-Iron --password "$DAVE" /path/to/theimage.img.zip
```

Do this for each card you have. Once your cards are imaged, plug them in, boot the Pi's, and [find them on the network.](https://github.com/adafruit/Adafruit-Pi-Finder)

For this next trick, I used ssh and broadcast input in iTerm (TMUX works well too), to issue the following commands across all the nodes in my cluster:

```
sudo apt-get update && sudo apt-get install hypriot-cluster-lab
```

The prior commands will take a couple of minutes to complete. Once they do, pick a node to be 'master', and direct all input to it (rather than all the nodes). On that node, run the following:

```
sudo systemctl start cluster-start
```

Again, this will take a few minutes. Once finished, run the prior command on *the rest* of your cluster nodes. At this point, fetch a coffee, it'll be a while.

Cool! You've now got a Docker Cluster running on Raspberry PI's

To read more about what the Hypriot group has going on, and some exercises to do with said cluster, check out their git page [here.](https://github.com/hypriot/cluster-lab)
