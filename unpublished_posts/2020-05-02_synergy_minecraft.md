---
templateKey: blog-post
title: Stuttering in Minecraft when walking while using Synergy
date: 2020-05-02T18:15:25-0500
featuredpost: false
featuredimage: /static/img/blog/featured/minecraft.png
featuredimagealt: picture of minecraft login screen
description: While using Synergy keyboard/mouse sharing tool, I had stuttering in movement when walking around within Minecraft. Turns out it was keyboard repeat and can be solved by "xset r off".
tags:
  - minecraft
  - synergy
  - kmv
  - raspberry pi
  - keyboard
---

I have a raspberry pi setup with Synergy server, and both my laptop and desktop are clients for it. This allows me to turn off either laptop or desktop, and still be able to use the shared keyboard.

In the Windows 10 client, I didn't encounter any issues (well, my button 4/5 for back and forward aren't working in Windows, but there are fixes for it). However, when I play Minecraft on Linux, I noticed stuttering in my movements when I hit WASD, and even flickering when holding down the tab key.

Turning on Debug mode showed me that the key presses were being repeated as I was holding it, rather than `key down` on press and `key up` on release. A bit of googling and I find that keyboard repeat is a xorg feature. The solution:

```bash{promptUser: user}{promptHost: localhost}
xset r off
```

Run this command in a terminal on the synergy server and it should turn off keyboard repeat.
