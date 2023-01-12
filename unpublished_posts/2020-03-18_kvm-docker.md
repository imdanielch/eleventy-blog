---
templateKey: blog-post
title: The ordeal of KVM on Ubuntu 18.04
date: 2020-03-18T13:58:07+1000
featuredpost: false
featuredimage:
featuredimagealt:
description: My experience trying to run Virtual Machines in an Ubuntu 18.04 host with Docker.
tags:
  - kvm
  - docker
  - ubuntu
  - centos
---

It actually wasn't meant to be this hard, only I didn't know what I was getting myself into. My particular issue was a host with docker installed. Docker modifies iptables and would cause the bridge connection to fail. This is a collection of the most relevant resources I've used to make it happen.

**Host:**
HP Proliant DL360 G7
- Ubuntu 18.04.4 server with **Docker**

**Guests:**
- Ubuntu 18.04.4
- CentOS 7

Setting up the Host
----
### Verify hardware support for Virtualization

```bash{promptUser: user}
sudo egrep -c '(vmx|svm)' /proc/cpuinfo
```
Has to return a number more than 0. If not, enable Virtualization in your BIOS.

```bash{promptUser: user}
ls /dev/kvm
```
The guide says to install `kvm-ok`, I figured just checking if its' listed should do for Ubuntu. If you are worried, [click here to go to the original guide](https://www.linuxtechi.com/install-configure-kvm-ubuntu-18-04-server/).



### Installing KVM

```bash{promptUser: user}
sudo apt update
sudo apt install qemu qemu-kvm libvirt-bin  bridge-utils  virt-manager
```
`virt-manager` is a GUI management software. I'm managing the server through SSH so I technically don't need it. However, it does depend on the packages I do need so this seems quicker to install. It also includes other tools which would be useful later on. [Click here to learn more](https://virt-manager.org/)

Add user to group
```bash{promptUser: user}
sudo adduser `id -un` libvirtd
```
You'll need to re-login for it to take effect. This allows your user to manage VMs

Turn on libvirtd

```bash{promptUser: user}
sudo systemctl start libvirtd
sudo systemctl enable libvirtd
```
You can check the status by issuing:
```bash{promptUser: user}
systemctl status libvirtd
```
Confirm installation was a success by checking with `virsh`

```bash{promptUser: user}
virsh list --all
```
this should return an empty list of virtual machines available.
If you get an error

```bash{outputLines:2-3}{promptUser: user}
virsh list --all
libvir: Remote error : Permission denied
error: failed to connect to the hypervisor
```
Then most likely you don't have permissions, try re-login or reboot. [More information on the ubuntu guide](https://help.ubuntu.com/community/KVM/Installation#Installation_of_KVM)
### Setting up network bridge

I wanted to set up a network bridge so the virtual machines would be in the same network as my LAN. After setting it up I realized it wasn't what I wanted but that's another story.

Ubuntu 18.04.4 server uses netplan to manage the network. Its a template configuration file is under `/etc/netplan/`.

The file could be named `50-cloud-init.yaml` or `01-netcfg.yaml`. Make a backup of the file just in case, eg:

```bash{outputLines:2}{promptUser: user}
ls /etc/netplan/
50-cloud-init.yaml
sudo cp /etc/netplan/50-cloud-init.yaml /etc/netplan/50-cloud-init.yaml-backup
```

Let's edit our network config
```yaml
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
  version: 2
  ethernets:
    enp3s0f0:
        dhcp4: true
        dhcp6: false
    enp3s0f1:
        dhcp4: true
    enp4s0f0:
        dhcp4: true
    enp4s0f1:
        dhcp4: true
```
My set up was simple, DHCP enabled on `enp3s0f0` and that's it. I don't use IPv6 so I didn't enable DHCP for it. This server has 4 Ethernet ports so that's why I have all these listed. You can ignore the comments in the file, they apply on  initial installation or when restoring from an image or something.

Let's make the edits:

For bridge with DHCP:
```yaml
network:
  version: 2
  ethernets:
    enp3s0f0:
        dhcp4: false
        dhcp6: false
    enp3s0f1:
        dhcp4: true
    enp4s0f0:
        dhcp4: true
    enp4s0f1:
        dhcp4: true
  bridges:
    br0:
      interfaces: [enp3s0f0] # The interface to use as a bridge
      dhcp4: true
```
For bridge with static IP:
```yaml
network:
  version: 2
  ethernets:
    enp3s0f0:
        dhcp4: false # Turn off DHCP
        dhcp6: false
    enp3s0f1:
        dhcp4: true
    enp4s0f0:
        dhcp4: true
    enp4s0f1:
        dhcp4: true
  bridges:
    br0: # the name of the bridge
      interfaces: [enp3s0f0] # The interface to use as a bridge
      addresses: [192.168.4.2/24] # IP for this bridge, /24 tells it the subnet. I'm not well versed with networking but I know this works.
      gateway4: 192.168.4.1 # Router's IP address
      mtu: 1500
      nameservers:
        addresses: [192.168.4.1,8.8.8.8] # My Router also does DNS, 8.8.8.8 is google's DNS server
      parameters:
        stp: false # set to true if you want it, and your router supports it.
        forward-delay: 0
      dhcp4: false
      dhcp6: false
```
Once this is configured, we should generate the config files which `networkd` will use, and apply them (WARNING: this will disconnect the network and reattach it as per configured. If there's an error, you may lose the ability to SSH into the server. Otherwise, it will disconnect briefly and then reconnect, may take a minute).
```bash{promptUser: user}
sudo netplan --debug generate
sudo netplan --debug apply
```
Confirm the changes applied properly
```bash{promptUser: user}
sudo networkctl status -a
```
and look for the entry `br0`, with:
```bash{outputLines: 2-10}{promptUser: user}
sudo networkctl status -a
[...]
State: routable, (configured)
Driver: bridge
Address: 192.168.4.2
Gateway: 192.168.4.1
DNS: 192.168.4.1
[...]
```
among other information.

[Linuxtechi.com does a better job showing you all this](https://www.linuxtechi.com/install-configure-kvm-ubuntu-18-04-server/)

See if you can ping out:
```bash
$ ping -I br0 -c 4 google.com
```
`-I br0` - specify interface to use. In this case, `br0`.
`-c 4` specifies 4 counts of ping.
we use `google.com` to also test DNS, otherwise use `8.8.8.8` for testing IP, and maybe `192.168.4.1` to see if you can even reach the local router (modify IP as per your settings).

If it pings, it works (we'll encounter an issue with docker later which took me 2 days to solve, but that was not related to this).

Note: Ubuntu community have a different approach to some things, [worth a read](https://help.ubuntu.com/community/KVM/Networking).

### Network bridge (Optional)

We'll add the bridge to the libvirt network. I originally did this, but after troubleshooting the docker issues I had reverted the changes and left it. It should be fine to do though.

create a file called `host-bridge.xml` and add in the contents here.
```xml
<network>
  <name>host-bridge</name>
  <forward mode="bridge"/>
  <bridge name="br0"/>
</network>
```
Then create the network
```bash{promptUser: user}
$ virsh net-define host-bridge.xml
$ virsh net-start host-bridge
$ virsh net-autostart host-bridge
```
Check it with:
```bash{outputLines: 2-5}{promptUser: user}
$ virsh net-list --all

 Name                 State      Autostart     Persistent
 ----------------------------------------------------------
 host-bridge          active     yes            yes
```
All that is stated by [Fabian Lee on this guide to create a bridged network with netplan on ubuntu bionic](https://fabianlee.org/2019/04/01/kvm-creating-a-bridged-network-with-netplan-on-ubuntu-bionic/).

Creating the VM
----

With that set, we are ready to create our VMs
BUT IT WILL FAIL BECAUSE OF DOCKER! If you don't have docker, you can continue. If you do, please scroll down to [troubleshooting](#Troubleshooting) first.

**Ubuntu:**

I created a shell script to keep the parameters clean in order to avoid mistakes.

```bash
#!/bin/bash
virt-install \
--name ubuntu18 \
--ram 24576 \
--disk path=/mnt/raid/vm/disks/ubuntu18.img,size=240 \
--vcpus 4 \
--virt-type kvm \
--os-type linux \
--os-variant ubuntu18.04 \
--graphics none \
--network bridge:br0 \
--location 'http://archive.ubuntu.com/ubuntu/dists/bionic/main/installer-amd64/' \
--extra-args "console=ttyS0 console=ttyS0,115200n8"
```

`--name` the name of the VM.

`--ram` how much RAM allocated for the VM in MegaBytes.

`--disk` the path where the virtual disk will be at. This creates one in `gcow2` format. On this line you also can specify the size in GigaBytes (in this case, `240GB`).

`--vcpus` How many virtual cores to allocate. Don't over allocate, don't under alocate. Meaning there's optimization for this but it's on a per use case.

`--virt-type kvm` available options listed by `$ virsh capabilities`. Choose as needed.

`--os-type linux` If your guest will be windows, windows.

`--os-variant` optimization based on the distribution you plan on installing.

`--graphics none` I don't need graphics, if you want to use VNC or of the sort, change this.

`--network bridge:br0` If you didn't set up the network into libvirt, you'd use this. Otherwise, `--network=host-bridge` should work.

`--location 'http://archive.ubuntu.com/ubuntu/dists/bionic/main/installer-amd64/'` Where to get install media. You can use a local ISO image too.

`--extra-args "console=ttyS0 console=ttyS0,115200n8"` Enable console (This didn't work for me with Ubuntu. There are additional steps coming up later in [Additional Ubuntu steps](#Additional-Ubuntu-Steps)).

[`virt-install` documentation](https://linux.die.net/man/1/virt-install) touches on all this.

I'll skip the installation since for the most part is pretty self explanatory. If not, there's lots of resources online to walk you through it.

### Additional Ubuntu Steps

(If you fixed the network, you can SSH to make these modifications, otherwise there's the method stated beneath)

I think there was some deprecation going on with Ubuntu so adding the extra-args didn't work. After installation Ubuntu would reboot and show a black console with a cursor. There was no way to access it because there was no network.

Note: Just found [this Ubuntu guide to define a console device for the Guest](https://help.ubuntu.com/community/KVM/Access), maybe it'll be helpful.

Otherwise, we use SSH.

edit `/etc/default/grub`:

```properties
# edit this line to add `console=ttyS0,115200n8 console=tty0`
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash console=ttyS0,115200n8 console=tty0"

# And uncomment this line to disable graphical terminal
GRUB_TERMINAL=console

```

update grub

```bash{promptUser: user}
sudo update-grub
```

Reboot the VM

```bash{promptUser: user}
virsh reboot ubuntu18
```

for it to take effect.

This will allow you to do `virsh console ubuntu18`. To get a terminal.

The `--extra-args` line added

```properties
GRUB_TERMINAL=serial
GRUB_SERIAL_COMMAND="serial --unit=0 --speed=115200 --word=8 --parity=no --stopp
=1"
```

but I think these are deprecated so they didn't work.

CentOS 7 steps
----

Before installing CentOS 7, I tried CentOS8. For some reason it wouldn't let me and it would fail into an initramfs terminal. CentOS 7 didn't give me as much trouble installing. Not sure why.

Regardless,

```bash{promptUser: user}
virt-install -n cent8 -r 4096 --vcpus=3 --disk path=/mnt/raid/vm/disks/cent7.imgbus=virtio,size=100 --os-variant rhel7 --nographics --location http://mirror.centos.org/centos/7/os/x86_64/ --extra-args console=ttyS0
```

These are the parameters I used. 4GB RAM, 3 vCPUs, 100GB disk

With internet working this was all it took. It actually installed even before I fixed the bridge so I used it to test connectivity from the VM.

Troubleshooting
====

Well, Docker modifies iptables for security reasons. This is great. However, it also blocks VM connection to our bridge. I spent 2 days on this.

If you try to install Ubuntu now, you'll fail at `Configuring the network with DHCP` with the error message `"Network authentication failed. Your network is probably not using the DHCP protocol. Alternatively, the DHCP server may be slow or some network hardware is not working correctly"`. Setting a static IP doesn't work either. It won't let you continue from this point.

CentOS 7 installed for me but then I wouldn't have network. This happened regardless of using bridge or the default NAT (with NAT, I could connect to local network but not ping out to the internet I think).

Docker iptables
----

This is because docker enables br_netfilter on for the bridge and drops connections by default. I don't understand the process well, but I assume it's so docker images don't automatically have open ports to the world. The answer on this [ServerFault post may answer the question better](https://serverfault.com/questions/963759/docker-breaks-libvirt-bridge-network) There are a few solutions that seem to have worked for others, I found [this one to be the least troublesome](https://bbs.archlinux.org/viewtopic.php?id=233727), albeit probably not a proper fix.

We'll add an entry to iptables which should free up our `br0` from Docker's clutches.

```bash{promptUser: user}
sudo systemctl edit docker
```

If you want to use vim as your editor

```bash{promptUser: user}
EDITOR=vim sudo -E systemctl edit docker
```

Add in the text:

```properties
[Service]
ExecStartPre=-/sbin/iptables -D FORWARD -p all -i br0 -j ACCEPT
ExecStartPre=/sbin/iptables -A FORWARD -p all -i br0 -j ACCEPT
```

First command removes the rule if it exists. The `-` at the beginning of the command tells it to return a success regardless of what it returns. This is because Docker will fail to start even when that command returns a mere warning. To learn more about [Special executable prefixes for systemd, click here](https://www.freedesktop.org/software/systemd/man/systemd.service.html).

Second command adds the rule back. This happens every time Docker starts because otherwise Docker will overwrite it.

Now we can apply, and restart docker.

```bash{promptUser: user}
sudo systemctl daemon-reload
sudo systemctl restart docker
```

From here we can go back to [Creating the VM](#Creating-the-VM).

Resources:
====

Many thanks to the community. Those who wrote guides, those who helped troubleshoot. The internet allows us to seek information for our problems where we can learn more about it and even find solutions. These are some of the resources I used, I'm missing many because I didn't log all the websites I looked up, just the ones I remember and still have tabs open.

- https://help.ubuntu.com/community/KVM/Installation
- https://www.linuxtechi.com/install-configure-kvm-ubuntu-18-04-server/
- https://fabianlee.org/2019/04/01/kvm-creating-a-bridged-network-with-netplan-on-ubuntu-bionic/
- https://bbs.archlinux.org/viewtopic.php?id=233727

