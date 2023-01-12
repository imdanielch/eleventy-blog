---
templateKey: blog-post
status: published
title: Installing Aseprite from GitHub
date: 2015-05-08T18:47:01.000Z
featuredpost: false
featuredimagealt:
featuredimage:
description:
tags:
  - ubuntu
  - aseprite
  - compile
  - makefile
  - install
  - cmake
  - github
---
Installing Aseprite from source can be a pain, which is why the developer offers a paid prepackaged solution [here](http://www.aseprite.org/download/). This is a tutorial for those who want to test new features and install it from source, and maybe provide feedback to the developer.


**Update 2015-05-07**: There's a GitHub release 22 days ago which compiles properly in Ubuntu 15.04 (have not tested in other distros). You can see it here [Aseprite v1.1-beta2](https://github.com/aseprite/aseprite/commit/849e40b0f950d5bb2519e42f1d9237d96c942c3d).

So during git checkout, use this stage:

```
cd aseprite && git checkout 849e40b0f950d5bb2519e42f1d9237d96c942c3d
```

---

![](/content/images/2015/05/snapshot2.png)

Installing Aseprite from the Ubuntu 14.10 repository gives you a version which has issues. Instead, we'll install it from GitHub.

Install git and prerequisites if not available:

```
$ sudo apt-get install git cmake libx11-dev && sudo apt-get build-dep aseprite
```

Clone the Aseprite master repository and its dependencies:

```
$ git clone --recursive https://github.com/aseprite/aseprite.git
```

Since the developer makes all his changes on the master branch, it can cause problems in linux therefore I chose a stage which contains fixes for linux compilation:

https://github.com/aseprite/aseprite/commit/85c569b2f16acc8b0f927e04d0f59f6915b80a6e

```
cd aseprite && git checkout 85c569b2f16acc8b0f927e04d0f59f6915b80a6e
```

Separate the compiled program into 'build' directory from the source code:

```
mkdir build && cd build
```

Specify generator to know what build it needs to make a makefile for, this will check available libraries on your computer:

```
cmake .. -G "Unix Makefiles"
```

Create makefile to build the software:

```
cmake .
```

compile it:

```
make install
```

That's it, now Aseprite will be installed under aseprite/build/bin/. Execute aseprite in that directory to open the program.

If you enjoy the software, please remember to support the developer [David Capello @davidcapello](https://twitter.com/davidcapello) by buying a license:

http://www.aseprite.org/download/

### Resources:

Official Aseprite GitHub install: https://github.com/aseprite/aseprite/blob/master/INSTALL.md
