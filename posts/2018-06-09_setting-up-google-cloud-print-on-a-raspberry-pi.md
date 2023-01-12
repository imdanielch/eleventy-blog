---
templateKey: blog-post
status: published
title: Setting up google cloud print on a raspberry pi
date: 2018-06-09T19:08:42.740Z
featuredpost: false
featuredimagealt:
featuredimage: 
description:
tags:
---
# Setup
raspberry pi 2 with Octopi (raspbian 8 jessie)
printer: Samsung ML-2510

# Install and configure CUPS
I won't go over installation of CUPS, this is well documented and will be dependent on your printers. I suggest making your raspberry pi's IP static, saves a bit of headaches from the CUPS server configuration.
## drivers:
Not sure which one is the one I needed, but these install drivers for Samsung and others.
```
sudo apt install splix
sudo apt isntall printer-driver-splix
```
Using the correct drivers is important. I tried using Samsung ML-2550 drivers but when google cloud print tried to print it would spin up but not print, and CUPS would think the print was successful. Switched to Samsung ML-2510 and it prints properly.

Resources:
https://github.com/google/cloud-print-connector/wiki/Install
https://github.com/google/cloud-print-connector/wiki/Build-from-source
https://github.com/google/cloud-print-connector/wiki/Installing-on-Raspberry-Pi-Raspbian-Jessie
https://packages.debian.org/search?keywords=splix