---
title: "Disabling / Enabling OSX Finder Tabs"
date: 2017-01-18
layout: "post"
categories: osx, finder, tabs
---

Here's to hoping Google picks this up, I couldn't find it elsewhere:

To tell Finder you don't want tabs when you open a folder:

```
$ defaults write com.apple.finder FinderSpawnTab -bool false
```

To put things back to normal:
```
$ defaults write com.apple.finder FinderSpawnTab -bool true
```

To figure out what a particular setting is:

```
$ defaults read com.apple.finder > ~/before.list

# Make the change

$ defaults read com.apple.finder > ~/before.list
$ diff ~/before.list ~/after.list
```

