---
title: "On Computerized Note Taking"
date: 2017-04-17
layout: "post"
categories: Notes, Electronic Notes, Note Taking, git, Markdown
---

I have blogged on and off about note taking, their importance, some random techniques and such. After a year or two of trying /LOTS/ of different tools, techniques, and the like, I thought I'd share the current method that has stuck for me.

This post is going to be rather long, so: tl;dr - Rakefile with templates, markdown formatted, auto-saved to an personal gogs server.

This post has four parts:

* How I got here
* The types of notes I take
* My current process
* The Tools

## How I got here

The long and short of this story, is I missed the days of [putting .LOG into Windows notepad and having it timestamp on each file open](https://support.microsoft.com/en-us/help/260563/how-to-use-notepad-to-create-a-log-file). Couple that with what is formerly Google Desktop search, and you had all of your notes, right there, indexed and with easy access.

This is not a knock on Evernote and other tools of the like. I tried just about every one of them. Along with easy searching, version control, and all the other good stuff, I gain a few extra benefits: easy transition to blog posts (commit to a different repo), easy export as static html or PDF for others consumption (using pandoc).

## The Types of Notes

When taking notes on the computer, there are three basic types I take:

* Case-Notes
* Call-Notes
* Draft Posts

### Case-Notes
These are analogous to project notes and are the most generic form of notes I take. The template for these looks like this:

```
Customer:    Orangutan Roasters
Project:     Burundi Roast
Author:      Your Name
Date:        2017-04-17
categories: 
---
  
# Orangutan-Roasters

Some background here
# Burundi Roast

Some background here
# Notes:
```

The information at the top is metadata, which wile searchable, does not get exported to PDF. It then includes two sections to provide background information about the task, customer, project, you name it. This is useful for context setting. Finally, you have a heading for freeform notes. Here I add timestamps as I go, so I have a running log of what I am doing and some context to return to.

### Call-Notes

Any time my phone rings, I take notes. This way I have a reference point of who I called, when, what about, etc. All the stuff you'd get from a recorded call, but you know, searchable. The template for that looks like this:

```
---
Subject:     Orangutan Roasters at Farmers Market
Customer:    Orangutan Roasters
Project:     Cottage Food Sales
Author:      Not Me
Date:        2017-04-17
categories: 
---
# Background

The context of the call goes here.

# Call details:
Organiser:   Not Me
Bridge:      1-800-867-5309,,112233#

On the call:
* 

# Unformatted notes:

Start taking notes here

```

Much like the more generic case-notes, the bits at the top are metadata that do not get exported, but are useful for finding this again later. Additionally, when using ```rake``` to create the template file, you can supply the call bridge for easy copy-paste later.

## The Tools

There are a few tools used here:

* Rake / Rakefile - Provides the template used for the different note / blog types
* Sublime Text - My editor of choice
* The Git and [GitAutoCommit](https://github.com/anjlab/sublime-text-git-autocommit) modules for Sublime - Version history
* pandoc module for Sublime - Easy export to Word, PDF, or HTML

### Organization

As you will see reflected in the Rakefile that follows, I keep my notes in a bit of a tree structure:

```
$ tree
.
├── Rakefile
├── call-notes
    └── 2017-04-13-pest-control.md
└── case-notes
    ├── 2017-03-02-home-scratch.md
    ├── 2017-03-02-work-scratch.md
    └── 2017-04-13-home-networking.md
```

The drafts for blog posts live in their own tree. As long as you specify the path in the Rakefile, you can store your notes anywhere.

### The Rakefile

The rakefile I use to create notes currently has three sections, one for each type of note I take most regularly: Blogs (:post), Project (:note), and Call (:call)

<script src="https://gist.github.com/bunchc/c91c0267999ddf8256fe60b954336aa0.js"></script>

To create a new note, from the command line one runs ```rake notetype parameter="thing"```. For example, if I wanted to open a new file for a coffee roast I would use a command like this:

```
rake note customer="Orangutan Roasters" project="Buruni Roast"
```

Which creates a file named similar to ```2017-04-17-Orangutan-Roasters-Burundai-Tasting.md``` and which contains the following template text:

```
Customer:    Orangutan Roasters
Project:     Burundi Roast
Author:      Your Name
Date:        2017-04-17
categories: 
---
  
# Orangutan-Roasters

Some background here
# Burundi Roast

Some background here
# Notes:
```

Now that you have the template file in place, you can open in an editor of your choice, and markdown being super close to plain text, you're off and going.

### Sublime Text

My editor of choice. Yes yes, I know I can do these things in vimacs or whatever and to each their own.

To install Sublime on OSX:
```
brew install Caskroom/cask/sublime-text3
```

This in turn links the ```subl``` command to ```/usr/local/bin/subl```. This allows you to open notes as follows:

```
subl case-notes/2017-04-17-Orangutan-Roasters-Burundai-Tasting.md
```

### Sublime Modules

As discussed above I use the Git, GitAutoCommit and pandoc modules. These can each be installed using their specific instructions. I've linked those here:

* [https://packagecontrol.io/packages/Pandoc](https://packagecontrol.io/packages/Pandoc)
* [https://packagecontrol.io/packages/Git](https://packagecontrol.io/packages/Git)
* [https://packagecontrol.io/packages/GitAutoCommit](https://packagecontrol.io/packages/GitAutoCommit)

Once you have those, these minor config tweeks will save you some heartache. For Pandoc, in your user-settings (Click sublime, Preferences, Package Settings, Pandoc, Settings - User), paste the following in:

```
    {
      "default": {

           "pandoc-path": "/usr/local/bin/pandoc",

        "transformations": {

          "HTML 5": {
            "scope": {
              "text.html.markdown": "markdown"
            },
            "syntax_file": "Packages/HTML/HTML.tmLanguage",
            "pandoc-arguments": [
              "-t", "html",
              "--filter", "/usr/local/bin/pandoc-citeproc",
              "--to=html5",
              "--no-highlight",
            ]
          },

          "PDF": {
            "scope": {
              "text.html": "html",
              "text.html.markdown": "markdown"
            },
            "pandoc-arguments": [
              "-t", "pdf",

              "--latex-engine=/Library/TeX/Root/bin/x86_64-darwin/pdflatex"
            ]
          },

          "Microsoft Word": {
            "scope": {
              "text.html": "html",
              "text.html.markdown": "markdown"
            },
            "pandoc-arguments": [
              "-t", "docx",
              "--filter", "/usr/local/bin/pandoc-citeproc"
            ]
          },

        },
        "pandoc-format-file": ["docx", "epub", "pdf", "odt", "html"]

      }

    }
```

To export using Pandoc, 'command + shift + p', then type 'pandoc', press enter and select how you want to export it.

## The Process

If you are still with me, thanks for sticking around. Now that all the scaffolding is in place, to create a new note, from the root of the notes folder:

```
$ rake call subject="Orangutan Roasters at Farmers Market" customer="Orangutan Roasters" project="Cottage Food Sales" bridge="1-800-867-5309,,112233#" owner="Not Me"

$ subl call-notes/2017-04-17-Orangutan-Roasters-Orangutan-Roasters-at-Farmers-Market.md
```

And you're off.

# Summary

Well, this ended up being **much** longer than I had expected, however, at the end of it all, you have notes that are versioned and backed up to git, and are searchable with grep or any tool of your choice.