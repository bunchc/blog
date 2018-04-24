---
title: "Slack Night Mode with Rambox"
date: 2018-04-23
layout: "post"
categories: slack, rambox, tweetdeck, themes, tools
---

This is a quick post to remind me how I got around the eye-razors that the default bright colored Slack client is. First, the end result:

![NightMode Screenshot](https://i.imgur.com/zkiEhPV.png)

So, it's not quite perfect, but it's workable. The theme itself is CSS, and there are a few ways to get Slack to use said CSS, depending on how you consume Slack. The links in the resources section below discuss how to do it via browser or desktop client. What follows here, is how to apply said theme using Rambox.

> __Note:__ I feel like I'm a bit late to the party both theme wise and to Rambox. [Rambox](http://rambox.pro/) is everything Adium / Pidgin wanted to be when it grew up, and lets me pull in Slack, Tweetdeck, and others into one spot.

## Night Mode for Slack in Rambox

To "enable" night mode, open Rambox, and then select "configure" for the Slack service to change:

![configure slack](https://i.imgur.com/sQkfhHS.png)

In the resulting window, expand the "Advanced" section at the bottom:

![Advanced settings](https://i.imgur.com/fD7FmKy.png)

In the "Advanced" text field, copy and paste the code from [here](https://gist.github.com/bunchc/80317a5c5a24337c29927869cba57af1).

## Resources

The theme itself, along with how to force Rambox to load it came from here:

* [Pavel Makhov](http://pavelmakhov.com/2017/01/rambox-dark-theme) - JavaScript snippet that loads the theme
* [Sidebar Theme](https://gist.github.com/mgreensmith/098897288f580b964ef8)
* [Slack Dark Theme](https://raw.githubusercontent.com/angelsix/youtube/develop/Windows%2010%20Dark%20Theme/Slack/slack-dark.css) - A Dark / Black theme that covers most elements. I changed the color values to that of [Solarized CSS](https://github.com/nakedsushi/solarized-slack).