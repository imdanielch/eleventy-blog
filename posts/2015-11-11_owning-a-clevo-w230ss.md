---
templateKey: blog-post
status: published
title: Owning a Clevo W230SS
date: 2015-11-11T22:13:01.563Z
featuredpost: false
featuredimagealt:
featuredimage:
description:
layout: layouts/post.njk
tags:
  - eurocom
  - m4
  - sound
  - troubleshoot
  - init
  - configure
---
Or depending on your reseller, Eurocom M4, which is what I have.

## Content

1. [Sound](#Sound)
2. [Network](#Network)
  a. [WiFi](#WiFi)
  b. [Bluetooth](#Bluetooth)

## Sound

### Headphones don't have sound after suspend or reboot:

https://bugs.launchpad.net/ubuntu/+source/alsa-driver/+bug/1313904
you can download the `init-headphone_0.2.0_all.deb` at the bottom of the comments.
run `sudo init-headphone`. If it tells you that your product is not supported (and you know for sure it's a w230ss), you can edit the executable.
`sudo vim /usr/sbin/init-headphone
`
and edit line 8:
`SUPPORTED_SYSTEM_PRODUCT_NAMES = ["W230SS"]
`
to fit your product, for me it was:
`SUPPORTED_SYSTEM_PRODUCT_NAMES = ["M4"]
`

**UPDATE**:

After upgrading to Kubuntu 15.04, the fix doesn't automatically work. I end up having to change the script again and remove the check for i2c_dev, this is because i2c_dev is incorporated into the kernel by default.

**UPDATE 2**:

Unrud commented underneath that the Eurocom M4 is now supported on his [GitHub repository](https://github.com/Unrud/init-headphone-ubuntu/releases) I haven't tested but could save you time.

Changes to script:

/usr/sbin/init-headphone line 86 to 95
```python
for line in modules_file.readlines():
# if "i2c_dev" == line.split()[0]:
#     module_i2c_dev_found = True
if "i2c_i801" == line.split()[0]:
module_i2c_i801_found = True
#    if not module_i2c_dev_found:
#        print("Warning: Module i2c_dev is not loaded", file=sys.stderr)
if not module_i2c_i801_found:
print("Warning: Module i2c_i801 is not loaded", file=sys.stderr)
return True
```

Then run these in terminal:

```
# modprobe i2c-dev
# modprobe i2c-i801
# init-headphone
```

If this doesn't work, you may need to install libi2c-dev:

```
# apt-get install libi2c-dev
```


## Network:

### WiFi

#### random disconnections:
This one is annoying, it randomly disconnects but the WiFi icon doesn't show it disconnects, you end up having to disconnect and reconnect manually to regain a connection, only to lose it again in 15mins~few hours.

Reference: http://ubuntuforums.org/showthread.php?t=2243978&page=3

_Side Note: This is a very nice script which would help troubleshoot wireless problems._
_Reference: http://ubuntuforums.org/showthread.php?t=370108_
_Mirror gist: [wireless script](https://gist.github.com/danielim/b9864f01d46c41880410)_

The command which seem to fix it for me:

Posted by [varunendra](http://ubuntuforums.org/showthread.php?t=2243978&p=13120817#post13120817)

```
# echo "options rtl8723be fwlps=N ips=N" | sudo tee /etc/modprobe.d/rtl8723be.conf
```

and reboot:

```
# reboot
```

hm, just got a hiccup, it seemed to disconnect for about 10~15 seconds and then reconnected, might be other network issues, so I'll try to monitor it.

### Bluetooth

Work in progress

http://askubuntu.com/questions/599893/bluetooth-is-not-working-on-14-04-with-bcm43142

http://askubuntu.com/questions/617260/bluetooth-does-not-detect-devices-on-lenovo-with-12-04

bluetooth audio:

http://askubuntu.com/questions/370254/how-can-i-get-the-a2dp-output-option-and-the-input-working-again

https://bugs.launchpad.net/ubuntu/+source/pulseaudio/+bug/1181106

Touchpad multitouch

xserver-xorg-input-multitouch
