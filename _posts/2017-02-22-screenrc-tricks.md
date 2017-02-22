---
title: ".screenrc tricks"
date: 2017-02-22
layout: "post"
categories: screen, screenrc
---

I'm not a tmux user.

There, I said it. I guess as tmux was becoming the thing to use to have lots of terminals open, I'd moved on to being Admin for a while. Who knows. screen 4lyfe.

With that said, here's a bit of my screenrc that makes life easier:

```
# Status bar
hardstatus always # activates window caption
hardstatus string '%{= wk}[ %{k}%H %{k}][%= %{= wk}%?%-Lw%?%{r}(%{r}%n*%f%t%?(%u)%?%{r})%{k}%?%+Lw%?%?%= %{k}][%{b} %Y-%m-%d %{k}%c %{k}]'

# Terminal options
term "xterm"
attrcolor b ".I"

# Turn off startup messaage
startup_message off

# Set the OSX term name to the current window
termcapinfo xterm* 'hs:ts=\\E]2;:fs=\\007:ds=\\E]2;screen\\007'

# In case of ssh disconnect or any weirdness, the screen will auto detach
autodetach on
```

The first line tells the status bar to always display. The second one, tells screen what this status should looklike. In this case, current user, windows with names, and date/time:

```
[ bunchc ][ 0$ irssi  (1*$vagrant)  2$ rbac-testing  3-$ docker-01 ][ 2017-02-22 15:35 ]
```

The next lines tell screen to:

* Set the term type to xterm for nested ssh
* Use bright colors for bold items
* Turn off the boiler plate when starting
* Set the OSX window title
* Autodetach if ssh breaks

## Getting remote hostnames as window names

This is not so much a screen thing as an ssh thing. First pull down [this script](https://gist.github.com/res0nat0r/221750) somewhere local. For me that's ```/home/bunchc/scripts```/

Then add these two lines to your .ssh/config:

```
# Screen prompts to the remote hostname
Host *
    PermitLocalCommand yes
    LocalCommand /home/bunchc/scripts/screen_ssh.sh
```

## Reloading the config from within screen

Now that you've got these settings, reload the screenrc file:
ctrl-a : source ~/.screenrc

## Resources
This post comes about after having collected these settings over a while. I'd love to give credit to all the original authors, finding posts from 2007 - 2009... well.

* [screen_ssh.sh](https://gist.github.com/res0nat0r/221750)
* [screen titles](http://aperiodic.net/screen/title_examples)
* [tenshu](http://www.tenshu.net/2007/06/screen-titles-from-ssh.html)