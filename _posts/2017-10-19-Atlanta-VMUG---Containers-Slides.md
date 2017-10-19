---
title: "Atlanta VMUG - Containers Slides"
date: 2017-10-19
layout: "post"
categories:  containers, docker, lxc, vmug, k8s
---

# Atlanta VMUG Usercon - Containers: Beyond the hype

Herin lie the slides, images, and Dockerfile to review this session as I presented it.

## Behind the scenes

Because this is a presentation on containers, I thought it only right that I use containers to present it. In that way, besides being a neat trick, it let me emphasize the change in workflow, as well as the usefulness of containers.

I used James Turnbull's ["Presenting with Docker"](https://kartar.net/2014/05/presenting-with-docker/) as the runtime to serve the slides. Then mounted three volumes to provide the slides, images, and a variant of index.html so I could live demo a change of theme.

## Viewing the presentation

Assuming you have Docker installed, use the following commands to build and launch the presentation:

```
# Clone the presentation
git clone https://github.com/bunchc/atlanta-vmug-2017-10-19.git
cd atlanta-vmug-2017-10-19/

# Pull the docker image
docker pull jamtur01/docker-presentation

# Run the preso
docker run -p 8000:8000 --name docker_presentation \
  -v $PWD/images:/opt/presentation/images \
  -v $PWD/slides:/opt/presentation/slides \
  -d jamtur01/docker-presentation
```

Then browse to [http://localhost:8000](http://localhost:8000). To view speaker notes, press S to view in speaker mode.