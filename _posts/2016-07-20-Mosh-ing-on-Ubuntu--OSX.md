---
title: "Mosh-ing on Ubuntu & OSX"
date: 2016-07-20
layout: "post"
categories: SSH, Mobile, Mosh
---

Today? Today was a treat. Instead of staying holed up in the home office, I decided to head out and attempt to people for a few hours. While the results of my peopling varied (Coffee shops are strange) I also encountered less than good network connectivity. That is, if the firewall logs are to be believed, I wasn't the only one running a torrent or twelve.

Terrible network connections right? SSH is typically robust in the regard. Not this time, apparently. Enter Mosh.

# What is Mosh

Mosh, or mobile shell, comes to you from MIT ([here](https://mosh.mit.edu/)). Mosh uses SSH to establish the initial connection and authentication. This lets all your ssh-key files and such keep working. It then launches it's own udp mosh-server to handle the actual session.

It's a great little package, and worth looking at a bit more. I've been using it instead of SSH for most the boxes I manage for a while now.

# Installing and Using Mosh

As with all things, there are two components. That which you install on your laptop, and the bits that run on the box you are managing. This section assumes OSX on the desktop and Ubuntu on the server. That said, if you check the Digital Ocean link in the references section, you'll see the install covered for other platforms. I'd also recommend adding this to whatever scripts you use to bootstrap a server.

## Installing Mosh on OSX

```
brew install mosh
```

That's it!

Oh, you don't have homebrew installed? While I'd suggest fixing that, you can also download the Mosh package from [here](https://mosh.mit.edu/#getting), and install like any other OSX package.

## Installing Mosh on Ubuntu 14.04

This is also straight forward:

```
sudo apt-get install -y mosh
```

Additionally, I added an exception for it in IPTABLES:

```
sudo iptables -I INPUT 1 -p udp --dport 60000:61000 -j ACCEPT
```

## Logging in

All that setup, phew. Fortunately, from this point forward it works very much like ssh, in that:

```
ssh bunchc@codybunch.local
```

Becomes:

```
mosh bunchc@codybunch.local
```

If you need to pass port forwarding bits along, it is still straight forward, but a bit less so than standard ssh:

```
mosh --ssh"ssh -p 9000" bunchc@codybunch.local
```

# Resources

- [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-mosh-on-a-vps)
