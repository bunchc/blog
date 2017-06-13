---
title: "OSX Update All The Things"
date: 2017-06-13
layout: "post"
categories: osx, update, sysadmin, homebrew, bash
---

I have a little alias, yes I do. I have a little alias, how about you?

Specifically, this alias keeps my box up to date, and is derived from [@mathis](https://twitter.com/mathias)'s excellent dotfiles:
https://github.com/mathiasbynens/dotfiles/blob/master/.aliases#L56-L57

```
alias update='sudo softwareupdate -i -a; \
    brew update; \
    brew upgrade; \
    brew cleanup; \
    brew cask outdated | xargs brew cask reinstall; \
    npm install npm -g; \
    npm update -g; \
    sudo gem update --system; \
    sudo gem update; \
    sudo gem cleanup; \
    vagrant plugin update; \
    sudo purge'
```

Hope that helps someone other than me.