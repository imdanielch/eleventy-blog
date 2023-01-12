---
templateKey: blog-post
title: Compiling synergy 1.10.3 on a Raspberry Pi
date: 2020-05-02T18:37:21-0500
featuredpost: false
featuredimage:
featuredimagealt:
description: Compiling Synergy on the Raspberry Pi isn't too difficult. In this post I'll go over the packages necessary to do so.
tags:
  - synergy
  - raspberry pi
---

I have a Raspberry Pi(RPi) setup with Synergy server, and both my laptop and desktop are clients for it. This allows me to turn off either laptop or desktop, and still be able to use the shared keyboard.

I am currently running Synergy 1.11.0 on the RPi, 1.10.3 on the laptop with Manjaro, and I think 1.11.1 on Windows 10 desktop.

I was having issues with Synergy (which were fixed without the need of compiling a new version, but that's fine) so I wanted to try matching versions. Looking around, I didn't find a prepackaged version of Synergy 1.10.3 for the RPi so I had to compile it myself.

Note that these instructions should work for Synergy version 1.10.3, but might not for other versions. I noticed they seem to change dependencies or file structure a little fairly often.

First we update the system.

```bash{promptUser: user}{promptHost: localhost}
sudo apt update && sudo apt upgrade -y
```

Then we install the necessary dev tools

```bash{promptUser: user}{promptHost: localhost}
sudo apt install gcc cmake libx11-dev libxtst-dev libssl-dev libavahi-compat-libdnssd-dev
```

Download the [Synergy v1.10.3 source code](https://github.com/symless/synergy-core/releases/tag/v1.10.3-stable) from GitHub.

unzip or untar:

```bash{promptUser: user}{promptHost: localhost}
sudo tar -xzf synergy-1.10.3-Source.tar.gz
sudo unzip synergy-1.10.3-Source.tar.gz
```

Navigate into the directory

```bash{promptUser: user}{promptHost: localhost}
cd synergy-1.10.3-Source
```

Raspbian file structure is a bit different from Ubuntu so to fix a path issue, edit CMakeList.txt and find

```
set(CMAKE_INCLUDE_PATH "${CMAKE_INCLUDE_PATH}:/usr/local/include")
```

Then replace with

```
set(CMAKE_INCLUDE_PATH "${CMAKE_INCLUDE_PATH}:/usr/include")
```

Save and run

```bash{promptUser: user}{promptHost: localhost}
sudo cmake .
```

To configure the makefile. If there were no issues, we can now build the binaries.

```bash{promptUser: user}{promptHost: localhost}
sudo make
```

This will create the synergy binaries in the `./bin` directory.

To use it, make sure you have uninstalled any other existing versions of Synergy and copy the binaries to `/usr/bin`

```bash{promptUser: user}{promptHost: localhost}
sudo cp -a ./bin/. /usr/bin
```

Now you can run it from the command line

```bash{promptUser: user}{promptHost: localhost}
synergy
```

Or add it to startup scripts. This step will depend on your Window Manager so I 'll skip it for this tutorial.

## Resources

- https://learn.adafruit.com/synergy-on-raspberry-pi/compiling-synergy-for-raspbian
- https://stackoverflow.com/a/46008642
- https://stackoverflow.com/a/28536113
