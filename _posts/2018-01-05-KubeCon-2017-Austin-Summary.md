---
title: "KubeCon 2017 Austin Summary"
date: 2018-01-05
layout: "post"
categories: kubernetes, k8s, kubecon, containers, docker
---

This post is a bit belated, and will honestly still be more raw than I would like given the time elapsed. That said, I'd still like to share the keynote & session notes from one of the best put on tech conferences I've been to in a long while.

Before we get started, I'd like to make some comments about KubeCon itself.

First, it was overall one of the better organized shows I've been to, ever. Registration & badge pickup was more or less seamless. One person at the head of the line to direct you to an open station, along with a friendly and detailed explanation of what and where things are... just spot on.

Right next to registration was a small table with what looked like flair of some manner, however, on closer inspection contained what might be the most awesome bit of the conference: communication stickers.

From the event site:

>Communication Stickers
>
>Near registration, attendees can pick up communication stickers to add to their badges. Communication stickers indicate an attendee’s requested level of interaction with other attendees and press.
>
>Green = open to communicate
>
>Yellow = only if you know me please
>
>Red = I’m not interested in communicating at this time
>
>Please be respectful of attendee communication preferences.

Communication stickers were just one of the many things the event organizers did around diversity & inclusion: [http://events17.linuxfoundation.org/events/kubecon-and-cloudnativecon-north-america/attend/diversity](http://events17.linuxfoundation.org/events/kubecon-and-cloudnativecon-north-america/attend/diversity)

## Keynote Summaries

What follows are my raw notes from the keynotes. You'll note, that these are from day 1's afternoon keynotes. This isn't because the rest weren't interesting, these were just the ones I found most interesting.

### Ben Sigelman - Lightstep

Creator of OpenTracking, CEO LightStep

Service Mesh - Making better sense of it's architecture. Durability & operationally sane.

Microservices: Independent, coordinated, elegant.

Build security in with service mesh.

Microservice issues are like murder mysteries. Distributed tracing, telling stories, about transactions across service boundaries. OpenTrace allows introspection, lots of things are instrumented for it, different front ends possible.

Service Mesh == L7 proxy, and allows for better hooks into tracing / visibility. Observing everything is great, need mroe than edges. Open Tracing helps bring all the bits together.

Donut zone. Move fast and bake things.

### Brendan Burns - This job is too hard

"Empowering folks to build things they might not have otherwise been able to."

Distributed systems a CS101 exercise.

Rule of 3's. It's hard because of too many tools for each step. Info is scattered too, all over the place. Why is this hard? - Principals: Everything in one place. Build libraries. Encourage re-use.

Cloud will become a language feature. It's already here, but will need more work. - Metaparticle, stdlib for cloud. Really neat, but still needs to grow.

metaparticle.io

## Session Notes

Next up, are raw notes from the sessions I made it to. If you're playing along at home, the CNCF has published a playlist of all the KubeCon 2017 sessions here: [https://www.youtube.com/watch?v=Z3aBWkNXnhw&list=PLj6h78yzYM2P-3-xqvmWaZbbI1sW-ulZb](https://www.youtube.com/watch?v=Z3aBWkNXnhw&list=PLj6h78yzYM2P-3-xqvmWaZbbI1sW-ulZb)

### Microservices, Service Mesh, & CI/CD

<iframe width="560" height="315" src="https://www.youtube.com/embed/UbLG_qUyCgM" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>

Concepts for the talk:

- CI
- CD
- Blue/Green
- Canary Testing
- Pipelines as code
- Service Mesh

CI/CD Pipeline is like robots, does the same thing over and over really well
Canary testing - App => proxy => filter some % of traffic to the app.

Canary w/ Microservices ecosystem is complex at best.

Sample pipeline:

__Dev__: build service => build container => deploy to dev => unit testing
__Prod__: deploy canary => modify routing => score it => merge PR => update prod

istio - a new service mesh tech. well adopted.

sidecar - envoy, istio control plane, creates route rule that controls side cars

Brigade - ci/cd tool, event driven scripting for k8s

- functions as containers
- run in parallel or serial
- trigger workflows (github, docker, etc)
- javascript
- project config as secrets
- fits into pipelins,

Kashti - dashboard for brigade

- easyy viewing & constructing pipelines
- waterfall view
- log output

### Vault and Secret Management in K8s

<iframe width="560" height="315" src="https://www.youtube.com/embed/FhUJYwM_xy0" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>

Why does vault exist? - Secret management for infrastructure as code. Hide the creds for your interconnects and such into vault, not github.

What is a secret? All the IAM stuff.

Current State:

* Scatter / sprawl everywhere
* Decentralized keys
* etc

Goals:

* Single source of secrets truth
* Programmatic application access (automation)
* Operations access (CLI/UI)
* Practical sec
* Modern Datacenter Friendly

Secure Secret Storage - Table stakes
Dynamic == harder

Security principles, same as they've always been:

* C,I,A
* Least Privilege (need to know)
* Separation (controls)
* Bracketing (time)
* no-repudiation (can't deny access)
* defense in depth

Dynamic secrets:

* Never provide "root" to clients
* provide limited access based on role
* granted on demand
* leases are enforceable
* audit trail

Vault supports a lot of the things, like, a lot of them. It's plugin based.

Apps don't use user / pass / 2fa
Users use ldap/ad and 2FA

Shamir Secret Sharing - Key levels to generate the master key, to generate the encryption key. See [here](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing).

"But I have LDAP" - When you have auth for k8s, and auth internal, and github and and and... There is no more single source of truth. Stronger Identity in Vault aims to help with this.

### Kernels v Distros

<iframe width="560" height="315" src="https://www.youtube.com/embed/fXBjA2hH-CQ" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>

Distro risks - fragmentation: conformance

Optionn 1 - Ignore it

- Doesn't work
- fragmentation
- etc

Distros have challenges tho, some options for k8s, but not without issues:

- Ignore it? Causes fragmentation
- Own it? Needs lots of support, marketing, etc.

Middle ground:

- Provide a 'standard' distro.
- Provide a known good starting point for others.

Formalize add-ons process: ```kubectl apt-get update```

Start with cluster/addons/?

k8s is a bit of both kernel & distro, releases every 6 weeks, good cadence. Maybe a break it / fix it tick tock.

__Manage the distro__

Fork all code into our namespace?

* Only ship what is under control
* Carry patches IFF needed

Push everything to one repo:

* No hunting
* managing randomly over the internet

Release every 6-12mo, distinguish component from package version. Base deprecation policies on distro releases.

Upgrade cadence with 1 quarter releases is too fast for upgrades. If you decouple 'kernel' from distro,

__New set of community roles__

* Different skills
* Different focus

We'll never have less users or more freedom than we do today, so, let's make the best decision.

Is the steering committee? - A dozen times this week. Every session on the dev day. Steering committee will delegate it, so let's have these discussions.

Model of cloud integrations ~= hardware drivers?

## Overall

It was refreshing to go to a conference as an attendee, and have this much great content over such a short period.