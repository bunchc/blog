---
title: "Building a Lab AD with PowerShell"
date: 2017-04-11
layout: "post"
categories: windows, powershell, ad, ldap
---

Remember that [post](http://blog.codybunch.com/2017/04/10/Building-a-Windows-Domain-with-PowerShell/) where we installed and built an Active Directory domain with PowerShell and BoxStarter?

Well, Domains are cool and all, but are generally uninteresting all by themselves. To that end, you can adapt the script and CSV files found [here](https://www.dominikbritz.com/2015/09/21/automate-your-lab-part-4-ad-ous-groups-and-users/) to populate your domain with some more interesting artifacts.

For me, I incorporated this into my existing BoxStarter build, so all I have is the one command to run, the script is below:

<script src="https://gist.github.com/bunchc/b7783fd220b5602cffc46158bac3099e.js"></script>

You'll notice, as compared to before, we've added some lines. Specifically:

* Lines 24 & 25 pull down all the files and extract them
* Lines 28 - 30 allow us to run internet scripts, change our working directory, and finally run our script.

As before, to launch this on a fresh Windows install:

```
START http://boxstarter.org/package/nr/url?https://gist.githubusercontent.com/bunchc/b7783fd220b5602cffc46158bac3099e/raw/a3e6b58904efb06953130112f98a5382cff7dc20/build_and_populate_domain.ps1
```

Once completed your script window will look similar to:
![PowerShell script output](https://i.imgur.com/Rlv9zMF.png)

And AD will look like this:
![AD Users and Computers](https://i.imgur.com/CJVbRgD.png)