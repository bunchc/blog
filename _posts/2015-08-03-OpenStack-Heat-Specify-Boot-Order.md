---
title: "OpenStack Heat, Specify Boot Order"
date: 2015-08-03
layout: "post"
categories: 
---

Another post inspired by [ask.openstack.org](https://ask.openstack.org/en/question/79391/how-to-specify-the-boot-order-of-two-instances-in-yaml)

The question was:
> If there are two instances(os:nova:server), how can I specify the boot order?

The Heat Orchestration Template, or HOT, specification provides a 'depends_on' attribute, that works generically on Heat resources. That is, depends_on a network, depends_on another instance, database, Cinder volume, etc.

In the [OpenStack docs](http://docs.openstack.org/developer/heat/template_guide/hot_spec.html#hot-spec-resources-dependencies) they provide the following example:

```
resources:
  server1:
    type: OS::Nova::Server
    depends_on: server2

  server2:
    type: OS::Nova::Server
```

Or for multiple servers:
```
resources:
  server1:
    type: OS::Nova::Server
    depends_on: [ server2, server3 ]

  server2:
    type: OS::Nova::Server

  server3:
    type: OS::Nova::Server
```

