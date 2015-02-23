---
title: "ZeroVM - Getting Started with ZeroCloud"
layout: post
date: 2014-10-22
categories: 
---

If you are getting up to speed on ZeroVM as I am, you will find the following useful:

- [Background](http://blog.codybunch.com/posts/2014-10-15-ZeroVM---Some-Background)
- [Isolation](http://blog.codybunch.com/posts/2014-10-16-ZeroVM---Security-Isolation-and-IO)
- [IO Operations](http://blog.codybunch.com/posts/2014-10-21-ZeroVM---IO--Channels/)
- [Getting Started with ZeroVM](http://blog.codybunch.com/posts/2014-10-17-ZeroVM---Getting-Started-Again/)
- [Lots of ZeroVM Resources](http://blog.codybunch.com/posts/2014-10-20-ZeroVM-Link-Dump)

Additionally, this [post](https://www.packtpub.com/books/content/mapreduce-openstack-swift-and-zerovm) does a great job of explaining how ZeroVM fits when you start to approach middling to large (Read: *Big Data*) problems. Go read it now.

## Getting Started with ZeroCloud

Now that you've read the [introduction by Lars](https://www.packtpub.com/books/content/mapreduce-openstack-swift-and-zerovm), and I trust that you have, we can start looking into the 'what' of ZeroCloud. In this post we'll touch on it's architecture and use case or two. In a subsequent post we'll show you how to build your own small-scale lab for such an environment.

**Note:** Also go here, watch [this](https://www.youtube.com/watch?v=VYZU_4w_dCA), around the 16:30 mark, pay attention to the 't = data / Rate' bits.

**Note:** This is where things in ZeroVM start to get really cool.

### ZeroCloud Architecture

The best way to describe the what and how of ZeroCloud is with a few diagrams which we will then dive into. These are borrowed form slides in past presentations.

![Highlevel ZeroCloud Architecture](http://openstack.prov12n.com/screens/1e1bd9b12a3230982c98-e2a0e10379dcd0e09ec354fba3ca6600.r72.cf1.rackcdn.comZeroCloud.pdf_2014-10-21_13-09-08.jpg)

This first diagram is a *hugely* simplified view of a swift environment, that is, if you have worked with OpenStack Swift it should look familiar enough. The "Proxy" nodes at the top are Swift Proxies, the Storage Nodes below are well, object storage nodes.

What changes for ZeroCloud is the addition of a few ZVM parts made available via Swift Middleware. In this case ZVM represents job brokers, some messaging, ZeroVM itself, as well as an 'executor' that handles the connecting of channels and the reading / writing of objects in Swift. This next diagram breaks that down further:

![ZeroCloud Dataflow Architecture](http://openstack.prov12n.com/screens/1e1bd9b12a3230982c98-e2a0e10379dcd0e09ec354fba3ca6600.r72.cf1.rackcdn.comZeroCloud.pdf_2014-10-21_13-10-12.jpg)

In this diagram, which admittedly is an eyechart, each step in the ZeroVM Swift Middleware is shown. Working left to right: 

**1.** The user initiates a job request via the Swift REST API by sending a post request with the code to be executed.

**2.** This hits the Swift Proxy node and is handed to the ZeroVM middleware, which, via a job scheduler finds where the closest replica of your data is, and sends the request along.

**3.** The job arrives at the executor who then fires up a net-new ZeroVM session for the job, opens the required IO Channels to your file* and if needed other ZeroVM sessions.

**4.** The ZeroVM job, finished with it's work, sends the response back up to the Proxy Server who then serves the response back to the user.

> *Note:* re: file in #3, in the default configuration this is a net new copy of the file rather than the actual object itself. This prevents accidental damage to the file.

### ZeroCloud Use Cases

So what do you do with a deterministic, computable, distributed data store? Some things that jump to mind are things like "A Giant MapReduce Cluster", similar to the one described [here](https://www.packtpub.com/books/content/mapreduce-openstack-swift-and-zerovm). As Lars describes, ZeroCloud can address one of the biggest MapReduce computational issues, that is data locality.

In addition to MapReduce type workloads, what else can one do? One could couple ElasticSeach and Log Stash, and indeed serve Kibana directly from Zero Cloud. Video trans-coding is another use case that I've seen. 

ZeroVM/ZeroCloud then, works well when you have a data heavy workload that would benefit being attacked in a distributed fashion.

## Summary

In this post we introduced you to ZeroCloud, the amalgamation of ZeroVM and OpenStack Swift. We dove into the architecture of ZeroCloud and introduced some use cases where having programmable distributed storage system is insanely useful.
