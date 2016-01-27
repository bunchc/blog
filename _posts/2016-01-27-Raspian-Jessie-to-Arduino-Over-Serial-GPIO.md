---
title: "Raspian Jessie to Arduino With Python"
date: 2016-01-27
layout: "post"
categories: IoT, PittMasterFlex, Embedded, Arduino, Raspberry Pi
---

After having spent way too much time beating my head against getting the Raspberry PI to communicate with the Arduino via GPIO serial, I've come up with the following steps that work well for Raspian Jessie & a branded Arduino UNO (Spent a lot of time fighting a knockoff board too.).

# Hold up (aka prerequisites)

Before we get started, you'll need at least three things:

- A computer of some sort. I'm using OSX for this.
- An Arduino of some sort. As mentioned before, after fighting a knockoff board, I'm using a good and proper [Arduino UNO](http://www.amazon.com/Arduino-Board-Module-ATmega328P-Blue/dp/B008GRTSV6/ref=sr_1_1?s=electronics&ie=UTF8&qid=1453927131&sr=1-1&keywords=arduino+uno)
- A Raspberry PI. In this case I'm using the recently released [Raspberry PI 2 Model B](http://www.amazon.com/Raspberry-Pi-Model-Project-Board/dp/B00T2U7R7I/ref=sr_1_1?s=pc&ie=UTF8&qid=1453927219&sr=1-1&keywords=raspberry+pi+2+model+b)

Once you have these things, we proceed in three general steps:

1. Configure the Arduino
2. Configure the PI
3. Connect the Arduino and the PI

# Configure the Arduino

Start with the Arduino connected to the Computer / Laptop. From there, download the Nanpy firmware from [here](https://github.com/nanpy/nanpy). Then:

    cd nanpy-firmware
    ./configure.sh
    cd ../
    cp -R Nanpy/ /path/to/arduino/sketches

Once the files are copied into place, fire up the Arduino IDE and:

File > Sketches > Nanpy
Upload

**Note**: You'll want to make sure you've selected the right board, and usb / serial port to talk to the Arduino first.

Once the upload completes, unplug the Arduino from the computer and plug it into the Pi.

# Configure the Raspberry PI

This part took me the longest, here's roughly the steps I followed.

First run these commands:

    sudo sed -i 's/console=ttyAMA0,115200//' /boot/cmdline.txt
    sudo sed -i 's/kgbdoc=ttyAMA0,115200//' /boot/cmdline.txt
    sudo systemctl stop serial-getty@ttyAMA0.service
    sudo systemctl disable serial-getty@ttyAMA0.service
    sudo pip install nanpy
    sudo reboot

The first two lines remove the USB/Serial port from the '/boot/cmdline.txt' file and stops the Pi from talking on said ports at boot time. The next two lines stop the corresponding service and disable it, which turns off the login screen on said port.

Next we install the nanpy library, which will allow us to use Python to write actual Arduino programs.

Last, we reboot the thing.

# Connecting the Arduino and the Raspberry Pi

Now that you have them both, configure your Arduino for the ["Blink" example](https://www.arduino.cc/en/Tutorial/Blink), then on the Pi, create a file called blink.py with the following contents:

    #!/usr/bin/env python
    # Author: Andrea Stagi <stagi.andrea@gmail.com>
    # Description: keeps your led blinking
    # Dependencies: None
    from nanpy import (ArduinoApi, SerialManager)
    from time import sleep

    connection = SerialManager()
    a = ArduinoApi(connection=connection)
    a.pinMode(13, a.OUTPUT)

    for i in range(10000):
        a.digitalWrite(13, (i + 1) % 2)
        sleep(0.2)

*Borrowed from the nanpy project [here.](https://raw.githubusercontent.com/nanpy/nanpy/master/nanpy/examples/blink.py)*

Beginning after the comments lines (prefixed with #):

The first line imports two specific bits of the nanpy library. The Arduino API, which allows us to do things like set a pin to input or output, read data from it, and so on. The other is the SerialManager, or the bit that connects to the Arduino over the serial port and reads / writes data to it.

The next three lines form our connection to the Arduino, and set pin 13 as an output pin. The last three lines, are what do the actual work, and blink the LED for you.

# Summary

While not a fancy final example, what we did here, was pretty significant. That is, we configured the Arduino with a new firmware and then configured the Raspberry Pi to both talk to it, as well as supply the instructions in Python.

## Resources
- [Nanpy](https://github.com/nanpy/nanpy)
- [Arduino IDE](https://www.arduino.cc/en/Main/Software)
- [Doing it on Wheezy](http://www.akeric.com/blog/?p=2420)
- [Troubleshooting the thing](https://github.com/nanpy/nanpy/issues/15)
- [Stopping the tty service on Jessie](https://www.raspberrypi.org/forums/viewtopic.php?f=66&t=123081)
- 

## Disclaimer

Wherein parts are concerned, I'm using Amazon affiliates links.
