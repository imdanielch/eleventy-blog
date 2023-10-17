---
templateKey: blog-post
status: published
title: Ubuntu issues.
date: 2015-11-02T02:21:52.374Z
featuredpost: false
featuredimagealt:
featuredimage:
description:
layout: layouts/post.njk
tags:
  - troubleshoot
  - kubuntu
  - kde
---
For some reason after running `apt-get upgrades`, and maybe a few other things happened in between, on my installation of Kubuntu 15.04, KDE had stopped working the way it should. It started with a crash and the screen turned black. Visible was only the mouse cursor which I could move, and IM Contacts which I could move too. Unfortunately I was unable to start any other program from within it, there was no panel, nor could I Alt+F2 to open anything through a command.

After looking around, and installing a whole bunch of other Desktop Environments (DE) which for the most part had other issues I didn't like, I managed to fix it.

By removing everything in `~/.cache/` and `~/.kde/`, you basically are cleaning up and starting fresh. A reboot and it worked for me.

Note: well, not completely, in between I also purged my kubuntu-desktop with `apt-get purge kubuntu-desktop` and reinstalled `apt-get install --no-install-recommends kubuntu-desktop`, but I question if this was necessary at all.

```
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=15.04
DISTRIB_CODENAME=vivid
DISTRIB_DESCRIPTION="Ubuntu 15.04"
NAME="Ubuntu"
VERSION="15.04 (Vivid Vervet)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 15.04"
VERSION_ID="15.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
```

## Ubuntu Unity Dash showing no applications:

http://askubuntu.com/questions/125843/dash-search-gives-no-result

Try wiping cache:

```
$ rm -rf ~/.cache
```

Try installing `unity-lens-applications` and `unity-lens-files` relog:

```
$ sudo apt-get install unity-lens-applications unity-lens-files
```

# Irssi or Weechat shortcut for switching windows `alt+#` does not work in Unity/Gnome
Issue is gnome-terminal has shortcuts which use the same combination to switch between terminal tabs. By disabling these, your commands will pass through to irssi/weechat and they will work properly. `Edit-> Preferences`, navigate to `shortcuts` tab and locate `Switch to Tab %d`, [1~10]. Clicking on the shortcut and hitting `backspace` will disable the shortcut.
