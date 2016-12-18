---
title: "Seti@Home with Docker on Raspberry Pi"
date: 2016-12-18
layout: "post"
categories: docker, seti, citizen science, raspberry pi
---

A little while back Tim from CERN commented related to my HomeLab. Specifically he was wanting me to run LHC@Home workloads. While I promise those are coming, they do not currently support ARM CPU's. There is an @home citizen science project that does however. Seti. You know, the one that started it all.

![Aliens](https://futurism.com/wp-content/uploads/2014/01/aliens-meme.jpeg)

To get started with this on docker on rPI you need to do a few things:

* [Flash the hypriot image onto your rPI](http://blog.hypriot.com/getting-started-with-docker-on-your-arm-device/)
* [Sign up for a Seti@home account](https://setiathome.berkeley.edu/join.php)
* On your rPI, run the container:

Don't worry about any of the values the first time around, we're only getting it going enough to get our authenticator code

```
docker run -d -t --env BOINC_CONFIG_CONTENTS="<account>
<master_url>http://setiathome.berkeley.edu/</master_url>
<authenticator>your_authenticator_code_here_get_it_from_setiathome</authenticator>
</account>" --env BOINC_CONFIG_FILENAME=account_setiathome.berkeley.edu.xml -i boinc
```

Next attach to the container and get your authenticator code:

```
docker exec -i -t /bin/bash <id of container>
boinccmd --lookup_account http://setiathome.berkeley.edu <your_email> <your_password>
```

Copy the authenticator code it gives you, and kill the container:

```
docker kill <id of container>
```

Finally restart it with the correct auth code:
![Science!](http://pre15.deviantart.net/0f82/th/pre/f/2015/330/a/8/stand_back_i_m_going_to_try_science_by_motorcycle_hero-d9i34zm.png)

```
docker run -d -t --env BOINC_CONFIG_CONTENTS="<account>
<master_url>http://setiathome.berkeley.edu/</master_url>
<authenticator>your_authenticator_code_here_get_it_from_setiathome</authenticator>
</account>" --env BOINC_CONFIG_FILENAME=account_setiathome.berkeley.edu.xml -i boinc
```

Once done, you can fire up htop and confirm all your CPU's have maxed out.

## Resources

* [Boinc Docker Image for rPI](https://hub.docker.com/r/bunchc/rpi-boinc/)
* [Boinc for Docker](https://github.com/browning/boinc-docker)
* [Seti@Home on Raspberry PI](https://pimylifeup.com/raspberry-pi-boinc/)
