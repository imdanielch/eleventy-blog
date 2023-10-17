---
templateKey: blog-post
status: published
title: Windows 8 partition won't mount in Linux.
date: 2015-03-22T05:12:12.000Z
featuredpost: false
featuredimagealt:
featuredimage:
description:
layout: layouts/post.njk
tags:
  - linux
  - windows
  - partition
  - hibernation
  - mount
  - ntfs
---
Windows 8.x has issues with being mounted in Linux due to a configuration which makes it save a page file and allows it to boot faster at start up.

http://itsfoss.com/solve-ntfs-mount-problem-ubuntu-windows-8-dual-boot/

Will tell you how to disable fast startup.

If this doesn't work, you can try restarting from Windows rather than shutting down.

If this still isn't fixing the issue, you can just destroy the hibernation file which is created by Windows.

```
sudo ntfs-3g -o remove_hiberfile /dev/sdXY /media/windows
```
remove_hiberfile: remove hibernation file.

/dev/sdXY: replace XY for the partition you are trying to mount. e.g.: /dev/sda2

/media/windows: this is the destination directory you wish to mount to.
