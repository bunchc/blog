---
title: "Reverse SSH for Lab Access"
date: 2017-08-20
layout: "post"
categories: lab, ssh, reverse ssh
---

There are times, like when you're writing a book with [Kevin Jackson](https://about.me/kevjackson), when you need access to your homelab while traveling. Sure you could set up a VPN, and there are plenty of decent guides for that, but, there are times when SSH is just simpler, and just sort of works.

For this to work, without opening ports on your lab firewall, we will need a server hosted somewhere else. Doesn't have to be large, in fact a (t2.micro) instance will work just fine.

## Build the cloud server

Log into your cloud provider of choise, and choose an Ubuntu flavored instance. Update, install SSH, and lock it down sone. The script linked below will do a lot of that work for you.

> __Note:__ Now that you have a JumpBox setup, you will want to lock it down some, so that not just anyone can get in. This [script](https://gist.github.com/bunchc/88ff359882548573d5fa138654f27385), not included here to keep the post short, enables a number of protections against spoofing. Disables all access inboud except for SSH, then installs and configures Aide, Fail2Ban, psad, and logwatch which will email you a daily report.

## Build the JumpBox

In your lab, you will now need to configure the JumpBox to "call home". That is, to SSH out to the cloud server, and setup up a tunnel in the reverse direction. For this we use the following script:

<script src="https://gist.github.com/bunchc/d79da2c5e4e0d4a1a98c5c725b65c182.js"></script>

Once you've created said script, set it to executable:

```
chmod +x ./create_ssh_tunnel.sh
```

Finally, we configure it to run every minute, to bring the ssh session back up should it fail. To do that, add the following line to your crontab:

```
*/1 * * * * ~/create_ssh_tunnel.sh >> tunnel.log 2>&1
```

## Connecting

Now that all the parts are in place, test a connection from your cloud server to the JumpBox:

From your console:
```
ssh user@cloudserver.address
```

From the cloud server:
```
ssh -l pirate -p 2222 localhost
```

And there you go.