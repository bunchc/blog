---
title: "Basic Automated Windows 10 post-install"
date: 2017-03-27
layout: "post"
categories: Automation, Windows 10, DevOps
---

It has been forever and a day since I've used Windows systems on a daily basis. However, in that time, the tools to do post install setup has evolved quite a bit. There are now tools like boxstarter and chocolatey, that when coupled with PowerShell, give you something akin to homebrew and dotfiles on osx (or apt & dotfiles on ubuntu, etc).

The following, are the commands I use to configure a fresh Windows 10 system:

```
START http://boxstarter.org/package/url?https://gist.github.com/bunchc/44e380258384505758b6244e615e75ed/raw/d648fffc21cb3cc7df79e50be6c05b05d29c79cc/0-SystemConfiguration.txt

START http://boxstarter.org/package/url?https://gist.githubusercontent.com/bunchc/44e380258384505758b6244e615e75ed/raw/239e8f6ca240a0c365619f242c14017f2f0de43e/1-Base%2520apps%2520setup.txt

START http://boxstarter.org/package/url?https://gist.github.com/bunchc/44e380258384505758b6244e615e75ed/raw/bae2117eba4091a78428493c2c821996ea5e3615/2-Dev%2520apps.txt

START https://github.com/Nummer/Destroy-Windows-10-Spying/releases/download/1.6.722/DWS_Lite.exe
```

The first command configures Windows updates and various privacy settings. The second command, pulls down a number of packages to provide a basic working environment. The third command does much the same with some additional packages. The last command pulls down a utility that helps clean up how much Win10 collects and sends home.

The first three commands launch boxstarter and supply a script to it. I've included those here:

<script src="https://gist.github.com/bunchc/44e380258384505758b6244e615e75ed.js"></script>

# Resources

I didn't do this alone. In this case, my versions of the scripts are almost identical to the originals from GreyKarnival:

* [https://gist.github.com/GreyKarnival/763adf559c413e0109cac8fad3604038](https://gist.github.com/GreyKarnival/763adf559c413e0109cac8fad3604038)

Additionally, the privacy tool run at the end comes from [here.](https://privacytoolsio.github.io/privacytools.io/)