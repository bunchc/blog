---
title: "Easy Conference Tunneling - OSX, Sidestep, and Cloud Servers"
date: 2016-08-25
layout: "post"
categories: vmworld, conferences, security
---

VMworld 2016 is upon us. Or was at the time of this writing. That doesn't change the message, however. When you are traveling for work, play or otherwise, who knows who else is on the WiFi with you. Who is snooping your traffic, and so on.

In this post, we'll cover setting up a Cloud server, ssh keys, and sidestep to provide you with traffic tunneling and encryption from where-ever you are.

Assumptions:

- An account with some cloud provider
- A recent version of OSX

## Set up the Cloud Server

THe instructions for this will vary some depending on the provider you use. What you are looking for however, is some flavor of Ubuntu 14.04 or higher. From there, apply [this](http://blog.codybunch.com/2015/05/12/Update-Userdata-Hardening-Script/), to provide a basic level of hardening.

You can either copy / paste it in as user data, or run it line by line (or as a script on the remote host).

## Set up SSH Keys

First, lets check to see if you have existing ssh keys:

```
ls -al ~/.ssh
```

You are looking for one of the following:

```
id_rsa.pub
id_dsa.pub
id_ecdsa.pub
id_ed25519.pub
```

Not there? Want a new one for this task? Let's make one. From the terminal:

```
ssh-keygen -t rsa -b 4096
```

When prompted just give it all the default answers (yes yes, passwordless keys are the devil, but, we're only using this key, for this server, for this conference, right?)

Finally we need to copy the new keys over to your server:

```
ssh-copy-id user@your.cloudserver.com
```

Finally use the key to log in to your cloud server, and disable password logins:

```
ssh user@your.cloudserver.com

sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_conf

cat /etc/ssh/sshd_conf | grep PasswordAuthentication

sudo service ssh restart
```

So what we just did was:
- Log into the cloud server
- Disable password auth
- Confirm that we disabled password auth (That is, there is no # in front of the line and that it reads 'no')
- Restarted SSH to enable the setting

## Sidestep

[Sidestep](http://chetansurpur.com/projects/sidestep/) is the glue that pulls all of this together. From their site:

    When Sidestep detects you connecting to an unprotected wireless network, it automatically encrypts all of your Internet traffic and reroutes it through a secure connection to a server of your choosing, which acts as your Internet proxy. And it does all this in the background so that you don’t even notice it.

So, first things first, download and install this the same as you would other OSX packages. Once installed you will need to configure it.

First, set up the actual proxy host & click test:
![](https://i.imgur.com/fv4oCR2.png)

Next, set sidestep up to work automagically:
![](https://i.imgur.com/My4g2LE.png)

# Summary

In this post we showed you how to setup a budget tunneling solution for when you are out and about conferencing, or otherwise on a network you do not trust.
