---
title: "ZeroVM - A ZeroCloud Lab"
date: 2014-10-23
categories: 
---

At this point we're pretty far down the ZeroVM rabbit hole. We've covered quite a bit of ground to get to this stage as well. Specific to this post, you will want to know more about ZeroCloud. You can do that [here](http://blog.codybunch.com/posts/2014-10-13-ZeroVM---Getting-Started-with-ZeroCloud).

Regarding getting up to speed on ZeroVM as I am, you will find the following useful:

- [Background](http://blog.codybunch.com/posts/2014-10-15-ZeroVM---Some-Background)
- [Isolation](http://blog.codybunch.com/posts/2014-10-16-ZeroVM---Security-Isolation-and-IO)
- [IO Operations](http://blog.codybunch.com/posts/2014-10-21-ZeroVM---IO--Channels/)
- [Getting Started with ZeroVM](http://blog.codybunch.com/posts/2014-10-17-ZeroVM---Getting-Started-Again/)
- [Lots of ZeroVM Resources](http://blog.codybunch.com/posts/2014-10-20-ZeroVM-Link-Dump)

## Building The ZeroCloud Lab

>There are a few ways of going about this, however before we get started, realize that this is still early days and if you find this post a few months or a year or two down the road, it will likely be super out of date.

To get working with ZeroCloud, the most straight forward path is to follow the directions in the ZeroVM Docs [here](http://docs.zerovm.org/zerocloud/devenv.html).

Once you've finished, you should have a functional ZeroCloud lab.

## Experimenting in the ZeroCloud Lab

Having a lab up in running is one thing, but it's rather boring thing by itself. So, let's do something with it! 

### ZeroCloud Hello World Example

The first thing we'll try to do, is a generic "Hello World" example, taken from [here](http://docs.zerovm.org/zerocloud/runningcode.html).

**1.** Log into the lab:

```
vagrant ssh

cd devstack/
./rejoin-stack.sh
source /vagrant/adminrc

zpm auth
export OS_AUTH_TOKEN=PKIZ_Zrz_Qa5NJm44FWeF7Wp...
export OS_STORAGE_URL=http://127.0.0.1:8080/v1/AUTH_7fbcd8784f8843a180cf187bbb12e49c
```

> **Note:** You will need to change the AUTH_TOKEN and STORAGE_URL values to match those returned by zpm auth.

**2.** Create our "Hello World" python file:

```
cat > example <<EOF
#!file://python2.7:python
import sys
print("Hello from ZeroVM!")
print("sys.platform is '%s'" % sys.platform)
EOF
```

**3.** Now we run the example:

```
curl -i -X POST -H "X-Auth-Token: $OS_AUTH_TOKEN" \
  -H "X-Zerovm-Execute: 1.0" -H "Content-Type: application/python" \
  --data-binary @example $OS_STORAGE_URL
```

Ok, so what did we do then? In the first step we logged in to our lab, sourced a file that contains credentials for our environment, and then set some additional environment variables to store our auth token and swift url.

Next we created the hello-world example file. In the absence of a text editor we just dumped everything between "<<EOF" and "EOF" into the file 'example'. Note the first line in the file begins with ```#!file://```. This is important as it lets the parser know what to do with said file, in this case, fire up python.

Finally, we use curl against our Swift install, letting it know to jump out to the ZeroVM middleware and that our application type is python. Additionally we specify our entire program as the payload in ```--data-binary```

## Summary

In this post, we showed you how to build a ZeroCloud lab and run an example "Hello World" application within it. If you would like to explore some more complex applications, be sure to explore the various talks and videos [here.](http://blog.codybunch.com/posts/2014-10-20-ZeroVM-Link-Dump)