---
title: "Docker on Raspberry PI for Fan Control"
date: 2017-07-14
layout: "post"
categories: Docker, Python, Flask, GPIO, Raspberry PI, rPI, REST
---

My home office gets a bit stuffy in the afternoon / early evenings. That I have west facing windows, this is not unexpected. Now, one could have solved this by adding shades, or an additional AC vent, or you know, not working past a given time.

... but that is no fun. So, I over engineered a solution using Docker, Python/Flask, and a Raspberry PI. Overkill? Maybe. Entertaining? Yup.

# Build

To keep things readable, I've broken the build out into a few components:

* Physical prep
* Setup the Raspberry PI
* Docker
* Flask / Python
* Test & Run

## Physical Preparation

As we're doing something "IRL", there are some parts required, and some hookups needed. The pictures in this section are borrowed from the Adafruit page for the [Powerswitch Tail 2](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-13-power-control?view=all).

### Parts list

You will need the following parts for this project:

* [Raspberry PI 2](https://smile.amazon.com/Raspberry-Pi-Model-Desktop-Linux/dp/B00T2U7R7I/ref=sr_1_3?ie=UTF8&qid=1500070934&sr=8-3&keywords=raspberry+pi+2)
* [Adafruit Powerswitch tail 2](https://www.adafruit.com/product/268)
* [Breadboard](https://smile.amazon.com/Elegoo-6PCS-tie-points-Breadboard-Arduino/dp/B01EV6SBXQ/ref=sr_1_3?ie=UTF8&qid=1500071001&sr=8-3&keywords=mini+breadboard)

### Assembly

First  up, connect the Powerswitc Tail to the fan. Then use jumper wires to connect it to the breadboard like this:

![Powerswitch Tail to Breadboard](https://cdn-learn.adafruit.com/assets/assets/000/004/166/original/learn_raspberry_pi_breadboard.png?1396809370)

If that is hard to read, you are connecting the +in on the Powerswitch Tail to a numbered GPIO pin. You are then connecting the -in to a GND pin. Once you have done this, you can plug the Powerswitch Tail into 110V power.

## Setup the Raspberry PI

Install HypriotOS and ensure you can login. To do this, I use the [flash](https://github.com/hypriot/flash) utility:

```
./flash --hostname node-00 --ssid lolwifi --password lolinternets \
    ~/Downloads/hypriotos-rpi-v1.4.0.img.zip
```

Once that finishes, plug the SD card into the rPI, power it on, and ensure it's up to date:

```
sshpass -p hypriot /usr/local/bin/ssh-copy-id pirate@node-01.local

ssh pirate@node01.local \
    "sudo apt-get update && \
    sudo apt-get install -y git wget curl vim mosh byobu && \
    sudo apt-get dist-upgrade -y"
```

## Docker

Next up, we need to create a Dockerfile for the container that will control the fan. Here is one to start from:

<script src="https://gist.github.com/bunchc/af38bbde1e6d38844ea7249a26858d16.js"></script>

* Line 1 - Tells Docker we want to start with the Arm version of Alpine Linux
* Line 3 - Installs python, pip, build utilities, and curl so we can test.
* Line 4 - Uses pip to install python modules needed to control the fan
* Line 5 - Copies our fan control program into our Docker container. We break this script down next.
* Line 7 - Exposes port 80 so we can control the fan remotely

## Flask / Python

Phew, that was a lot of work, no? Why don't you turn on the fan and sip a cold drink? Oh... we still haven't told python how to control the fan just yet. Place the following python into a file called fan.py.

As you'll see in both the script and the breakdown that follows, we use the RPi GPIO library to interface with the Powerswitch Tail to turn the fan on and off. We also use Flask to provide a simple REST interface to control the fan remotely.

Here's the script, followed by a breakdown of what it is doing:

<script src="https://gist.github.com/bunchc/a148027524674e100dfb00902c17f964.js"></script>

* Line 1: Imports the GPIO library
* Lines 2&3: Import the parts of Flask we are going to use.
* Lines 5-12: Initialize the GPIO pin we are going to use
* Lines 14-16: Create a new Flask app
* Lines 18-20: Define the /status endpoint.
    - This allows us to request fan status. 0 = off; 1 = on
* Lines 22-29: Defines the power control endpoint /power/[on|off]
    - Line 25 turns the fan on
    - Line 27 turns the fan off
    - Line 29 returns the new status of the fan
* Lines 31&32: Tells python to use Flask to serve our app on all interfaces on poer 80

## Running

Phew, all that hard work I broke a sweat. Let's actually turn the fan on!

To do that, once all your files are saved, you should have a directory structure that looks like this on the Raspberry PI:

```
$ tree
.
├── Dockerfile
└── fan.py
```

Now, let's have Docker build our image:

```
$ pwd
/home/pirate/projects/fan-control

$ docker build -t fan-control .
```

Next, run the image:

```
$ docker run -d -p 80:80 --restart=always --privileged fan-control python /fan.py
```

What this does, is tells Docker to run our container in the background, to keep it running, and to allow it access to the hardware (Needed for GPIO control). Finally, it tells Docker to start our fan controller in the container.

### Controlling the fan

Now, you can control the fan. From the RPi you can do this as follows:

```
# Get status
$ docker exec keen_neumann curl -s http://localhost/status
0

# Turn it on
$ docker exec keen_neumann curl -s http://localhost/power/on
1
```

You can also do this remotely using the same curl commands, changing localhost to the IP or hostname of the RPi:

```
$ curl -s http://node-01.local/power/off
0

$ curl -s http://node-01.local/status
0
```

# Resources
* [http://mattrichardson.com/Raspberry-Pi-Flask/](http://mattrichardson.com/Raspberry-Pi-Flask/)
* [https://learn.adafruit.com/adafruits-raspberry-pi-lesson-13-power-control?view=all](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-13-power-control?view=all)