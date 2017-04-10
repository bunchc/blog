---
title: "Building a Windows Domain with BoxStarter"
date: 2017-04-10
layout: "post"
categories: powershell, windows, devops
---

I had a need to create and recreate Windows domains for some lab work I've been up to. What follows here is adapted from [@davidstamen](https://twitter.com/davidstamen) who blogs [here](http://davidstamen.com). More specifically it is extracted from his Vagrant Windows lab, [here](https://github.com/dstamen/Vagrant/tree/master/windows).

>**Warning!** Boxstarter is sort of like curl | sudo bash for Windows.

First things first, look over what we're doing:
<script src="https://gist.github.com/bunchc/1d97b496aa1d6efe146f799b2fb34547.js"></script>

What is going on here?
* The first two lines contain the domain name you'd like configured.
* Lines 4 & 5 make Explorer a bit less annoying and enable remote desktop (if it's not already).
* Lines 7 - 15 install some useful packages
* Line 18 enables AD
* Line 22 installs the domain.

All of that is simple enough. The magic comes in when we use boxstarter to go from a new Windows Server to Domain Controller. From an admin command prompt on the Windows server, run the following:

```
START http://boxstarter.org/package/nr/url?https://gist.githubusercontent.com/bunchc/1d97b496aa1d6efe146f799b2fb34547/raw/51ebf18ca320c49c38e2f493e0ff4afad59bb0cd/domain_controller.ps1
```

>*Note:* You may need to add boxstarter.org to trusted sites.

Once executing, it should look a bit like this:

![BoxStarter Domain Installation](https://i.imgur.com/OveXd0d.png)