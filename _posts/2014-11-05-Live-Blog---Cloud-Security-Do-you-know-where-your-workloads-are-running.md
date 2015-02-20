---
title: "Live Blog - Cloud Security: Do you know where your workloads are running?"
date: 2014-11-05
categories: 
---

Session background can be found [here](https://openstacksummitnovember2014paris.sched.org/event/98e55a657e2e78e8beacaaacde714a0f).

Speaker: Raghu Yeluri, Intel Corporation

> As an Enterprise and/or a Cloud service provider, you would have to ensure that all regulatory requirements for workload and data sovereignty are met.   You have to answer the questions from your customers like:

>where is my workload running? ,  Are my workloads running in a compliant location? ,  How can I trust the Integrity of the host servers on which my workloads are running , can you prove to me that my workloads and data have not violated policies? , How can I control via policy where my workload can and cannot migrate and run .

## Live Blog

Geo Tag / Asset tags are set in a write once area. Can be almost anything, user provided names, gps coordinates, actual asset tags, and importantly, the certificate of attestation from the TPM.

Today we're using Glance Image Registry to set the launch properties / policies: e.g. Only runs in France. The "Trust and Launch" scheduler filter runs last against the list of servers left. It then runs a variant of Open Attestation to say, "Which are trusted?". From there, the scheduler will deploy. This is all automatic. Only setting the tags is manual. This same attestation happens during migrations as well.

This is how we enable boundary control. We added Horizon plugins to support launch policies. Extended Nova scheduler for location filtering, and glance for policies. Finally we provide a number of tools that work in conjunction (OAT, etc). The end-to-end bits can be done entirely in OSS however.

This should be upstream in Kilo, however, we (Intel) provide scripts to make this work in Icehouse & Juno.

Live Demo - Lol WiFi!

There is no maximum number of tags that can be set, things like GEO, PCI-DSS, etc can be set, these are then used to select the servers from there.

Magic number of policies: 5. This was in conversation with NIST and MSP's

Looking forward:

- Extend geo-tagging for volumes
- Tenant-controlled encryption / decryption under controlled circumstances

Extend geo-tagging for volumes - Basically the above but for Cinder. The scheduler is pluggable, so we should be able to make this happen. The assumption being x86 storage host. This will be more difficult with traditional SAN/NAS due to not being TXT enabled... *yet*.