---
title: "ZeroVM - Getting Started, Again"
date: 2014-10-17
categories: 
---

Ok, so now that we've covered a LOT of ZeroVM background & isolation details, let's actually start to get our hands dirty.

tl;dr - ```git clone https://github.com/bunchc/vagrant-zerovm.git && cd vagrant-zerovm && vagrant up```

## Getting Started

ZeroVM has packages for Ubuntu 12.04, so in order to get started you will either need hardware with 12.04 installed or a VM of the same. Once you have a VM up and running you are ready to install.

## Installing ZeroVM

To install ZeroVM, log into your Ubuntu 12.04 setup and run the following commands:

Install some needed packages:
```
sudo apt-get update
sudo apt-get install -y curl wget
```

Install the ZeroVM repository and key:
```
sudo su -c 'echo "deb http://packages.zerovm.org/apt/ubuntu/ precise main" > /etc/apt/sources.list.d/zerovm-precise.list' 
wget -O- http://packages.zerovm.org/apt/ubuntu/zerovm.pkg.key | sudo apt-key add - 
```

Next, refresh our repository and install ZeroVM.
```
sudo apt-get update
sudo apt-get install -y zerovm zerovm-cli
```

With that, you have ZeroVM installed.

By itself that is not interesting, so let's show off a the Python version of "Hello world" run inside ZeroVM:

```
wget http://packages.zerovm.org/zerovm-samples/python.tar 
echo 'print "Hello"' > hello.py

zvsh --zvm-image python.tar python @hello.py 
```

The first command fetches a cPython which has been cross compiled to work in ZeroVM. Next we set up our example file and run it. There are some even more interesting examples if you head over the [ZeroVM download site.](http://www.zerovm.org/download.html)

## Summary

In this post, you setup an Ubuntu 12.04 machine, installed the ZeroVM apt repositories, and installed ZeroVM. Finally, you created a Python of hello world and ran that within your newly installed ZeroVM.