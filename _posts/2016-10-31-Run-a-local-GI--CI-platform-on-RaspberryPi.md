---
title: "Git at home with Hypriot & docker-compose"
date: 2016-10-31
layout: "post"
categories: docker, devops, git
---

The folks at Hypriot put out a [post](http://blog.hypriot.com/post/run-your-own-github-like-service-with-docker/) recently on running your own git server using Gogs in Docker on the Raspberry Pi. While this was a great start, I wanted to mix and match this with their [DockerUI container](http://blog.hypriot.com/post/new-docker-ui-portainer/).

Thing is, I didn't want to start them each separately using a mix of docker commands. To solve this I set up following in my docker-compose.yml:

```
gogs:
  image: hypriot/rpi-gogs-raspbian
  restart: always
  volumes:
    - '/home/pirate/gogs-data/:/data'
  expose:
    - 22
    - 3000
  ports:
    - '8022:22'
    - '3000:3000'


dockerui:
  image: hypriot/rpi-dockerui
  restart: always
  volumes:
    - '/var/run/docker.sock:/var/run/docker.sock'
  expose:
    - 9000
  ports:
    - '80:9000'
```

Then fired up the environment using:

```
docker-compose run -d
```

And there you go. DockerUI and Gogs for local git repos.
