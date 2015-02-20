---
title: "Changing Jenkins Home Folder"
date: 2014-10-27
categories: 
---

Everything Jenkins does happens in the context of the jenkins user by default. While that sounds obvious, it has some interesting implications if you aren't careful. 

## The Problem

In my homelab, I've installed 14.04 onto a 16gb usb key. This is a hold over from the days when it was an ESXi lab, but, it works for the most part. However, for things like downloading vagrant boxes and the like, one needs to mount external storage. Enter the problem: Jenkins sets itself up in /var/lib/jenkins by default, which lives on my very limited usb stick.

This in turn means anytime Jenkins tried to 'vagrant up' a thing, it would fill my root partition trying to store all the temporary files vagrant used. Whoops.

## Changing the Jenkins Home Directory

There are a number of posts out there on how to do this. None of them, however, addressed the vagrant issue in particular. To do that, one has to hit it with a larger bat.

### First Change /etc/defaults

The first thing to do, is change how Jenkins thinks of itself. The most straight forward way I found was to change that in /etc/defaults/jenkins. This will be around line 23 or 25:

```
# jenkins home location
JENKINS_HOME=/new/location
```

### Then Change /etc/passwd

This is the step that was needed to fix the problem of Jenkins running vagrant and vagrant subsequently filling /:

Change:
```jenkins:x:105:114:Jenkins,,,:/var/lib/jenkins:/bin/bash```

To this:
```jenkins:x:105:114:Jenkins,,,:/new/location:/bin/bash```

### Finally Restart Jenkins

Now to make sure all the things stick:

```sudo /etc/init.d/jenkins restart```

## Summary

In this post we showed you how to change the Jenkins home directory, and why it's also important to change it in /etc/passwd.