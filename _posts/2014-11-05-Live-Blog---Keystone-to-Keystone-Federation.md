---
title: "Live Blog - Keystone to Keystone Federation"
layout: post
date: 2014-11-05
categories: 
---

Session details [here.](https://openstacksummitnovember2014paris.sched.org/event/f7207fef7547319f322fa5cfee05cf49)

Speakers: 
- Marek Denis - Research Fellow, CERN 
- Steve Martinelli - Software Developer, IBM
- Joe Savak Sr. - Product Manager, Rackspace
- Brad Topol - Distinguished Engineer, IBM

>In this presentation, we describe the federated identity enhancements we have added to support Keystone to Keystone federation for enabling hybrid cloud functionality. We begin with an overview of key hybrid cloud use cases that have been identified by our stakeholders including those being encountered by OpenStack superuser CERN. We then discuss our SAML based approach for enabling Keystones to trust each other and provide authorization and  role support for resources in hybrid cloud environments.   

## Live Blog

Lots of different folks interested in identity federtion, Academia, companies, lots and lots of folks.

Use cases? - Easy to confiture, cloud bursting, central policy point, federating out, federating in. Keep the client small. No new protocols.

"Federate In" - You already have identity provider, SAML, etc Folks already have SSO / Identity. Federate allows for use of existing credentials to work with OpenStack MSP's.

"Federate Out" - That is, you setup a trust between on prem and off prem clouds.

Cern's Use-Case

Cern has 70,000 cores, they need more to process ALL the data they produce. This requires federation out allows folks to use pay-as-you-go to hire out additional resources as needed.

Cern also needs to be able to allow folks to federate in from others in science community.

Now an interlude for Keystone classic Auth.

Federated identity in Incehouse - Integrate existing tools, SAML, etc. There is a diagram, it has lots of arrows, the gist is you send SAML to keystone, keystone gives you a token, and things are good. This worked, but not as well as it could. Mapping engine, that is, groups in one system are not the same as groups in others. Woo Mapping: "IBM Regular Employees" --> "regular_canada" etc.

New diagram for Federation in Juno. A lot more arrows. This time around, Keystone is the provider, and will provide some level of attestation to the other Keystone in the trust relationship. Once the trust is in place, the user passes the token to either.

The SAML generator takes the token and goes backwards. Token --> SAML Generator --> SAML Assertion

Now we're at a slide covering all manner of config data. Important bits: Mapping is still a thing. You also need to 'prime' the SAML assertion pump.

keystone-manage now has a metadata generation thing.

Back to Cern: - 2 datacenters, OpenStack Cells, Cells not popular. 40k users in AD, and 12k more ADFS (Federation). Cern uses SAML2, and will be the first OpenStack in the world to allow federate-in to allow external entities to consume their resources.

Patches to the OpenStack and Keystone clients.

Looking forward:

- Auth-N
- Horizon Integration
- More & Better Mapping
- Fine Grained ACLs
- More protocols
