---
title: "My First Django App"
date: 2017-01-07
layout: "post"
categories: python, django
---

Having had some time over the winter break, I took some time to watch the excellent Django Webcast on O'Reilly's Safari. What follows here are the commands used in said video cast used to get started with an app:

## Setting up a virtual environment

My first instinct to keep the development environment separate from my working world was to fire up a VM and go with it. Turns out, in Python you can still work locally without much fear of breaking your box. You do this with ```virtualenv```'s, and manage those with ```virtualenvwrapper```.

> *Note:* One can use virtual environments without virtualenvwrapper. virtualenvwrapper made things a bit easier for me.

### Install virtualenvwrapper on OSX

For this, I assume you have a working homebrew:

```
brew update
brew install pyenv-virtualenvwrapper
```

### Install virtualenvwrapper on Ubuntu 16.04

Thankfully, it's a happy little apt-package for us here:

```
sudo apt-get update
sudo apt-get install virtualenvwrapper
```

### Configuring virtualenvwrapper

Now that you have it installed on your system, the following .bash_profile settings set up some specific behaviors in virtualenvwrapper. These work on both OSX and Ubuntu:

```
echo "export WORKON_HOME=$HOME/.virtualenvs" >> ~/.bashrc
echo "export PROJECT_HOME=$HOME/projects" >> ~/.bashrc

source ~/.bashrc
```

The first line tells virtualenvwrapper where you would like it to store all of the files (python, pip, and all their parts) for said virtualenv. The second line tells virtualenvwrapper where your code lives. Finally, we pull said values into out working bash shell.

### Create and enter your virtual env

Now that's all sorted, let's make a virtual environment to work on:

```
mkvirtualenv -p /usr/bin/python3 newProject
```

Breaking this down, the ```-p /usr/bin/python3``` tells virtualenv to install python3 into our virtualenv. The name ```newProject``` is well, the new project name. This command will produce output like the following:

```
$ mkvirtualenv -p /usr/bin/python3 newProject
Already using interpreter /usr/bin/python3
Using base prefix '/usr'
New python executable in newProject/bin/python3
Also creating executable in newProject/bin/python
Installing setuptools, pip...done.
```

To enter your virtual environment and start working on things:

```
$ cd ~/projects/
$ mkdir newProject
$ cd newProject/
$ workon newProject
```

## Installing and Getting Started with Django

Ok, so that was a lot of setup to get to this point, but, here we are, it's time to install Django, create the structure of our application, and finally start Django's built in webserver to make sure it is all working.

To install Django inside your virtual environment:

```
$ pip install django
Downloading/unpacking django
  Downloading Django-1.10.5-py2.py3-none-any.whl (6.8MB): 6.8MB downloaded
Installing collected packages: django
Successfully installed django
Cleaning up...
```

Now let's install the skeleton of our app:

```
django-admin startproject newProject
```

This will create a directory structure like this:

```
$ tree
.
└── newProject
    ├── manage.py
    └── newProject
        ├── __init__.py
        ├── settings.py
        ├── urls.py
        └── wsgi.py

2 directories, 5 files
```

Next up, we will want to fire up django's built in server and validate our install:

```
$ cd newProject
$ python manage.py migrate

Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying sessions.0001_initial... OK

$ python manage.py runserver
Performing system checks...

System check identified no issues (0 silenced).
January 07, 2017 - 23:52:17
Django version 1.10.5, using settings 'newProject.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

Taking a look at what we did there:
* manage.py migrate - preforms the initial migration & population of a sqlite database so our shell application will work.
* manage.py runserver - started Django's built in webserver

While there is a lot more to actually writing a Django app, this is essentially how you get started on a net-new application.

# Summary

In this post we installed virtualenvwrapper and used it to crate a new virtual environment. We then installed Django, performed an initial database migration and run Django's built in server so we could browse to and test the shell application.
