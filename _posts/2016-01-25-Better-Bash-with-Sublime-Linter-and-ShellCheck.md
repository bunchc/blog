---
title: "Better Bash with Sublime Linter and ShellCheck"
date: 2016-01-25
layout: "post"
categories: scripting, code, devops, bash
---

As I stated in my [Markdown workflow](http://blog.codybunch.com/2014/08/14/Updated-Blog--Markdown-workflow/) post. I use Sublime Text. A lot. Besides iTerm2, it is one of the few always running apps on my laptop. Just so super useful. Beyond markdown, I've started using Sublime for *Everything*. Python, shell scripts, etc.

Which, brings us to the subject of said post. That is, recently, it was pointed out that most of my code, scripts and such, while workable, is ugly. Thing is, ugly code isn't just a me problem, whole communities suffer from this. All manner of projects have coding standards listed, and will outright reject or at least gently suggest you comply to said standards before submitting code.

# Sublime Linter

Enter [SublimeLinter](https://sublimelinter.readthedocs.org/en/latest/). It's sort of a meta-plugin for Sublime, in that each of it's "Linters" are plugins. Each of these "Linters" provides real time 'background' checks of the code you write against a number of standards. For Python, it enforces two spaces after a function, and will let you know if you are doing anything particularly teribad with imports, variable names, and the like.

The preferred method to install SublimeLinter is Package Control for Sublime. Failing that, use the install instructions from their [site](https://sublimelinter.readthedocs.org/en/latest/). Each additional linter you want to add is also installed via Package Control. On installation, most of these require the installation of a 3rd party library or binary to preform the actual linting.

# Shell Check

So, how does this work in bash? One uses Package Control to install [SublimeLinter-shellcheck](https://github.com/SublimeLinter/SublimeLinter-shellcheck). Additionally, if you are on OSX and using HomeBrew, `brew install shellcheck` or, an `apt-get install shellcheck` on Apt managed systems.

[Shellcheck](https://github.com/koalaman/shellcheck) is a handy enough tool, that well, helps you out of bad situations in [bash](https://github.com/koalaman/shellcheck/blob/master/README.md#user-content-gallery-of-bad-code).

# Summary

Hope this helps you clean up your code / act, or at least, helps make it cleaner for those who come next. Enjoy!
