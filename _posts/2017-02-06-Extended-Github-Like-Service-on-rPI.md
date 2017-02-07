---
title: "Extended Github Like Service on rPI"
date: 2017-02-06
layout: "post"
categories: docker, alpine, docker-compose, git, github
---

The good folks over at [Hypriot](https://blog.hypriot.com/post/run-your-own-github-like-service-with-docker/) have a wonderful tutorial on how to run your own GitHub like service. To do this, they use Gogs on a raspian image. A great starting point, but, well, what if you wanted to scale it some?

> Yes yes, we're running on a Raspberry PI, 'scale' in this case just means build it a bit more like you'd deploy on real hardware.

Forgiving my ascii, the following depicts what we will build:

```

+------------------------+
|                        |
|      nginx             |
|                        |
+-----------+------------+
            |
            |
+-----------v------------+
|                        |
|      gogs              |
|                        |
+-----------+------------+
            |
            |            
+-----------v------------+
|                        |
|     postgres           |
|                        |
+------------------------+

```

This build is involved, so, lets dive in.

## Before Starting

To complete the lab as described, you'll need at least one Raspberry Pi running Hypriot 1.x and a recent version of Docker and docker-compose.

> Everything that follows was adapted for rPI, meaning, with some work, could be run on an x86 Docker as well.

## What we're building

In this lab we will build 3 containers:

* nginx - A reverse proxy that handles incoming requests to our git server
* gogs - The GoGit service
* postgres - a SQL backend for gogs.

> Ok, maybe not entirely production, but, a few steps closer.

## Getting Started

To get started, we'll make the directory structure. The end result should look similar to this:

```
$ tree -d
.
├── gogs
│   ├── custom
│   │   └── conf
│   └── data
├── nginx
│   └── conf
└── postgres
    └── docker-entrypoint-initdb.d
```

You can create this with the following command:
```
mkdir gogs-project; cd gogs-project
mkdir -p gogs/custom/conf gogs/data \
    nginx/conf postgres/docker-entrypoint-initdb.d
```

Next, at the root of the project folder, create a file called 'env'. This will be the environment file in which we store info about our database.

```
cat > env <<EOF
DB_NAME=myproject_web
DB_USER=myproject_web
DB_PASS=shoov3Phezaimahsh7eb2Tii4ohkah8k
DB_SERVICE=postgres
DB_PORT=5432
EOF
```

## Define the hosts in docker-compose.yaml

Next up, we define all of the hosts in a docker-compose.yaml file. I'll explain each as we get into their respective sections:

```
nginx:
  restart: always
  build: ./nginx/
  ports:
    - "80:80"
  links:
    - gogs:gogs

gogs:
  restart: always
  build: ./gogs/
  expose:
    - "3000"
  links:
    - postgres:postgres
  volumes:
    - ./gogs/data:/data
  command: gogs/gogs web

postgres:
  restart: always
  image: rotschopf/rpi-postgres
  volumes:
    - ./postgres/docker-entrypoint-initdb.d:/docker-entrypoint-initdb.d
  env_file:
    - env
  expose:
    - "5432"
```

## Build the Reverse Proxy

Repeated here, is the nginx section of the docker-compose.yaml file.

```
nginx:
  restart: always
  build: ./nginx/
  ports:
    - "80:80"
  links:
    - gogs:gogs
```

* restart - This container should always be running. Docker will restart this container if it crashes.
* build - Specifies a directory where the Dockerfile for this container lives.
* ports - Tells docker to map an external port to this container
* links - Creates a link between the containers and creates an /etc/hosts entry for name resolution

You'll notice we told docker-compose we wanted to build this container rather than recycle one. To do that, you will need to place a Dockerfile into the ./nginx/ folder we created earlier. The Dockerfile should have the following contents:

```
FROM hypriot/rpi-alpine-scratch:v3.4
RUN apk add --update nginx \
    && rm -rf /var/cache/apk/*

COPY conf/nginx.conf /etc/nginx/nginx.conf
COPY conf/default.conf /etc/nginx/conf.d/default.conf

CMD ["nginx", "-g", "daemon off;"]
```

* FROM - sets our base image.
* RUN - installs nginx and cleans out the apk cache to save space.
* COPY - these two lines pull in nginx configurations
* CMD - sets nginx to run when the container gets launched

Finally, we need to provide the configurations specified in the copy commands. 
The contents of which follow. These should be placed into ./nginx/config/

```
$ cat nginx/conf/nginx.conf

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        off;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```
This first config provides a very basic setup for nginx, and then pulls additional configuration in using the include line.

```
$ cat nginx/conf/default.conf
server {

    listen 80;
    server_name git.isa.fuckingasshat.com;
    charset utf-8;

    location / {
        proxy_pass http://gogs:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

}
```
This file tells nginx to listen on port 80, and to serve all requests to / back to our gogs instance over port 3000.

Let's look at what we've got so far:
```
$ pwd
/home/pirate/rpi-gogs-docker-alpine/nginx

$ tree
.
├── conf
│   ├── default.conf
│   └── nginx.conf
└── Dockerfile
```

## Building the Go Git service

What follows is how we build our Go Git container. A reminder of what this looks like in docker-compose.yaml:

```
gogs:
  restart: always
  build: ./gogs/
  expose:
    - "3000"
  links:
    - postgres:postgres
  volumes:
    - ./gogs/data:/data
  command: gogs/gogs web
```

* restart - We'd like this running, all the time
* build - Build our gogs image from the Dockerfile in ./gogs/
* expose - Tells docker to expose port 3000 on the container
* links - Ties us in to our database server
* volumes - Tells docker we want to map ./gogs/data to /data in the container for our repo storage
* command - launches the gogs web service

Neat, right? Now let's take a look inside the ./gogs/Dockerfile:

```
FROM hypriot/rpi-alpine-scratch:v3.4

# Install the packages we need
RUN apk --update add \
    openssl \
    linux-pam-dev \
    build-base \
    coreutils \
    libc6-compat \
    git \
    && wget -O gogs_latest_raspi2.zip \
        https://cdn.gogs.io/gogs_v0.9.128_raspi2.zip \
    && unzip ./gogs_latest_raspi2.zip \
    && mkdir -p /gogs/custom/http /gogs/custom/conf \
    && /gogs/gogs cert \
    -ca=true \
    -duration=8760h0m0s \
    -host=git.isa.fuckingsshat.local \
    && mv *.pem /gogs/custom/http/ \
    && rm -f /*.zip \
    && rm -rf /var/cache/apk/*

COPY custom/conf/app.ini /gogs/custom/conf/

EXPOSE 22
EXPOSE 3000
CMD ["gogs/gogs" "web"]
```

Unlike the nginx file, there is a *LOT* going on here.

* FROM - Same alpine source image. Tiny, quick, and does what we need.
* RUN - This is actually a bunch of commands tied together into a single line to reduce the number of Docker build steps. This breaks down as follows:
    - apk --update add - install the required packages
    - wget - pull down the latest gogs binary for pi.
    - unzip - decompress it
    - mkdir - create a folder for our config and ssl certs
    - /gogs/gogs cert - creates the ssl certificates
    - mv *.pem - puts the certificates into the right spot
    - rm - these two clean up our image so we stay small
* COPY - This pulls our app.ini file into our container
* EXPOSE - tells docker to expose these ports
* CMD - launch the gogs web service

Still with me? We have one last file to create, the app.ini file. To do that, we will pull down a generic app.ini from the gogs project and then add our specific details.

First, pull in the file:
```
$ curl -L https://raw.githubusercontent.com/gogits/gogs/master/conf/app.ini \
    > ./gogs/custom/conf/app.ini
```

Next, find the database section, and provide the data from your env file:
```
[database]
; Either "mysql", "postgres" or "sqlite3", it's your choice
DB_TYPE = postgres
HOST = postgres:5432
NAME = myproject_web
USER = myproject_web
PASSWD = shoov3Phezaimahsh7eb2Tii4ohkah8k
; For "postgres" only, either "disable", "require" or "verify-full"
SSL_MODE = disable
; For "sqlite3" and "tidb", use absolute path when you start as service
PATH = data/gogs.db
```

That's it for this section. Which should now look like this:

```
$ pwd
/home/pirate/rpi-gogs-docker-alpine/gogs
HypriotOS/armv7: pirate@node-01 in ~/rpi-gogs-docker-alpine/gogs
$ tree
.
├── custom
│   └── conf
│       └── app.ini
├── data
└── Dockerfile
```

## Setting up Postgres

Unlike the last two, postgres is fairly simple to setup, as instead of building it from scratch, we're running a community supplied image. Let's take a look at the postgres section of docker-compose.yaml:

```
postgres:
  restart: always
  image: rotschopf/rpi-postgres
  volumes:
    - ./postgres/docker-entrypoint-initdb.d:/docker-entrypoint-initdb.d
  env_file:
    - env
  expose:
    - "5432"
```

* restart - like our other containers, we want this one to run, all the time.
* image - specifies the postgres image to use
* volumes - This is a special volume for our postgres container. The container will run script contained within. You'll see how we use this next.
* env_file - turns our env file into environment variables within the container
* expose - makes port 5432 available for connections

Now, in  your ./postgres/docker-entrypoint-initdb.d/ directory, we're going to place a script that will create our database user and database:

```
$ cat ./postgres/docker-entrypoint-initdb.d/postgres.sh
#!/bin/env sh
psql -U postgres -c "CREATE USER $DB_USER PASSWORD '$DB_PASS'"
psql -U postgres -c "CREATE DATABASE $DB_NAME OWNER $DB_USER
```

Cool, once you've done that, you have finished the prep work. Now we can make the magic!

## Make the Magic!

Ok, so, that was a lot of work. With that in place, you can now build and launch the gogs environment:

```
docker-compose build --no-cache
docker-compose up -d
```

Once that completes, point a web browser to your raspberry pi on port 80, and enjoy your own github service.

# Summary

Long post is long. We used docker compose to build two docker images and launch a third. All three of which are tied together to provide a github like service.

# Resources
* [Django Stack with docker-compose](http://capside.com/labs/deploying-full-django-stack-with-docker-compose/)
* [Hypriot Github Like Service](https://blog.hypriot.com/post/run-your-own-github-like-service-with-docker/)