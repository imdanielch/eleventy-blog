---
templateKey: blog-post
status: published
title: KVM setup and installing guest VMs CentOS 7
date: 2015-03-12T02:28:05.000Z
featuredpost: true
featuredimagealt:
featuredimage: 
description:
layout: layouts/post.njk
tags:
  - kvm
  - linux
  - system-administrator
  - vm
  - bridge
  - network
  - centos-7
  - selinux
  - virsh
---
##Host OS:
```
# cat /etc/*-release
CentOS Linux release 7.0.1406 (Core)
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CentOS Linux release 7.0.1406 (Core)
CentOS Linux release 7.0.1406 (Core)

# uname -ri
3.10.0-123.20.1.el7.x86_64 x86_64
```
##Check if Host supports virtualization
###Support for Intel VT-x (code name Vanderpool) and AMD-V (code name Pacifica):
```
# egrep -c '(vmx|svm)' /proc/cpuinfo
2
```
0 = no virtualization
\>1 = virtualization available

###Check if your CPU is 64bit:
```
# egrep -c ' lm ' /proc/cpuinfo
2
```
0 = not 64bit
\>1 = Note: lm stands for Long Mode which equates to a 64-bit CPU.

###Check if kernel is also 64bit:
```
# uname -m
x86_64
```
All good.

##Install KVM and preparing Host
###YUM install:
```
# yum -y install qemu-kvm libvirt virt-install bridge-utils
...
```
###Make sure modules are loaded:

```
# lsmod | grep kvm
kvm_amd                59987  3
kvm                   441119  1 kvm_amd
```
###Enable virtualization with systemd:
```
# systemctl start libvirtd
# systemctl enable libvirtd
```

###Configure Bridge networking for KVM virtual machine:
Decided to stick to NetworkManager because it is default in CentOS 7. This might not work properly with older versions.
copy from: http://www.server-world.info/en/note?os=CentOS_7&p=kvm

Note: instead of "br0", I went with "vbr0" and this reflects after this block of commands.
```
Replace the interface name "eno16777736" for your own environment's one.
# add bridge "br0"
[root@dlp ~]# nmcli c add type bridge autoconnect yes con-name br0 ifname br0 
Connection 'br0' (0f4b7bc8-8c7a-461a-bff1-d516b941a6ec) successfully added.
# set IP for br0
[root@dlp ~]# nmcli c modify br0 ipv4.addresses "10.0.0.30/24 10.0.0.1" ipv4.method manual 
# set DNS for "br0"
[root@dlp ~]# nmcli c modify br0 ipv4.dns 10.0.0.1 
# remove the current setting
[root@dlp ~]# nmcli c delete eno16777736 
# add an interface again as a member of br0
[root@dlp ~]# nmcli c add type bridge-slave autoconnect yes con-name eno16777736 ifname eno16777736 master br0 
# stop and start NetworkManager
[root@dlp ~]# systemctl stop NetworkManager; systemctl start NetworkManager 
[root@dlp ~]# ip addr 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eno16777736: <BROADCAST,MULTICAST,UP,LOWER_UP> 
    mtu 1500 qdisc pfifo_fast master br0 state UP group default qlen 1000
    link/ether 00:0c:29:9f:9b:d3 brd ff:ff:ff:ff:ff:ff
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 22:f8:64:25:97:44 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
4: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 00:0c:29:9f:9b:d3 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.30/24 brd 10.0.0.255 scope global br0
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe9f:9bd3/64 scope link
       valid_lft forever preferred_lft forever
```
###Allow passthrough firewalld (not sure if necessary):
From: http://forums.fedoraforum.org/showpost.php?p=1710469&postcount=9
```
# firewall-cmd --permanent --direct --passthrough ipv4 -I FORWARD -m physdev --physdev-is-bridged -j ACCEPT
```
###Set folder permissions for VM location:
For this example, VM will be placed in /vm/*
otherwise replace "/vm(/.\*)?" and /vm with your location. e.g. "/home/vm(/.\*)" and /vm with /home/vm
```
# mkdir /vm
# semanage fcontext -a -t virt_image_t "/vm(/.*)?"; restorecon -R /vm 
```
semanage changes context setting for the directory.
restorecon -R updates/refreshes the context.

##Install Guest VM
###Installation of Ubuntu 14.04 64bit VM:
`virt-install --os-variant=list` to see if ubuntu trusty is supported, otherwise use `--os-variant=ubuntusaucy`
Extract from: http://www.server-world.info/en/note?os=Ubuntu_14.04&p=kvm&f=2
Download Ubuntu 14.04 ISO from http://ubuntu.com
```
Install on text mode via network, it's OK on Console or remote connection with Putty and so on.
# virt-install \
--name template \
--ram 2048 \
--disk path=/vm/template.img,size=30 \
--vcpus 2 \
--os-type linux \
--os-variant ubuntutrusty \
--network bridge=br0 \
--graphics none \
--console pty,target_type=serial \
--cdrom  \
--extra-args 'console=ttyS0,115200n8 serial'
Starting install...# installation starts
```
For options, make sure 'man virt-install', there are many options
--name
specify the name of Virtual Machine
--ram
specify the amount of memories of Virtual Machine
--disk path=xxx ,size=xxx
'path=' ⇒ specify the location of disks of Virtual Machine
'size=' ⇒ specify the amount of disks of Virtual Machine
--vcpus
specify the virtual CPUs
--os-type
specify the type of GuestOS
--os-variant
specify the kind of GuestOS
--network
specify network types of Virtual Machine
--graphics
specify the kind of graphics. if set 'none', it means nographics.
--console
specify the console type
--location
specify the location of installation where from
--extra-args
specify parameters that is set in kernel. `console=ttyS0,115200n8 serial` allows serial connection from the host. `115200n8` is the highest frequency common serial baud rate. This line might not be necessary but I had issues with CentOS 6.6 with this.

```
Escape character is ^]
```
Wait for a bit and installation should start soon.
if you hit ```ctrl + ]``` you can escape the guest VM and be back at the host. To enter back into the guest VM do ```# virsh console <vmname>``` press [Enter] for login prompt to show.



###Installation of CentOS 7 VM:
good read: http://www.server-world.info/en/note?os=CentOS_7&p=kvm&f=2
I haven't tried this yet.

###Installation of CentOS 6.x VM 64bit:
I had issues with this. When I installed this with standard procedure the login prompt or install prompt would not show.

```
# virt-install \
--name www \
--ram 2048 \
--disk path=/var/kvm/images/www.img,size=30 \
--vcpus 2 \
--os-type linux \
--os-variant rhel6 \
--network bridge=br0 \
--graphics none \
--console pty,target_type=serial \
--location 'http://centos.mirrors.tds.net/pub/linux/centos/6.6/os/x86_64/' \
--extra-args 'console=ttyS0,9600n8 serial'
```
What I have found is that the Serial Baud rate of ```112500``` didn't seem to print on my ssh client (Putty tray, not sure if client makes a difference). Trying ```9600``` seemed to fix it and allowed me to install.

###Installation of Learn Puppet VM:
VM information as of publish date 2015-03-11:
```
# sudo cat /etc/*-release
CentOS release 6.5 (Final)
```
```
# uname -a
Linux learning.puppetlabs.vm 2.6.32-431.el6.i686 #1 SMP Fri Nov 22 00:26:36 UTC 2013 i686 i686 i386 GNU/Linux
```
Download learn puppet VM from https://puppetlabs.com/learn

KVM isn't able to import VirtualBox format nor VMWare format.
I downloaded VirtualBox version and extracted the VMDK.
Convert VMDK to img or qcow2 which are compatible with KVM.
```
qemu-img convert -f vmdk -O qcow2 learn_puppet_centos-6.5-disk1.vmdk learn_puppet_centos-6.5-disk1.qcow2
```
Import qcow2 into kvm:
```
# sudo virt-install --name learnpuppet --ram 2048 --disk path=/home/vm/learn_puppet/learnpuppet.qcow2 --vcpus=1 --os-type linux --os-variant rhel6 --arch=i686 --network bridge=vbr0 --graphics none --console pty,target_type=serial --import
```
Here's where I found great issues. After weeks have progressed thinking for firewall issues, SELinux issues, possible corrupted vmdk, bad converting with ```qemu-img``` and some others I can't currently recall. After countless hours I managed to close in to the issue. Serial baud.

We need to edit grub.conf inside the guest VM. For this we need guestfish
```
# yum install guestfish
```
make sure learnpuppet VM is not running.
```
# virsh destroy learnpuppet
```
This powers off the VM as if you unplugged the power (not graceful, but you have no access to console so nothing else to do).

Use guestfish to access files in learnpuppet VM and edit grub.conf (read further to see what I changed).
```
# guestfish -a learnpuppet.qcow2

Welcome to guestfish, the guest filesystem shell for
editing virtual machine filesystems and disk images.

Type: 'help' for help on commands
      'man' to read the manual
      'quit' to quit the shell

><fs> run
><fs> list-filesystems
/dev/sda1: ext2
/dev/vg00/rootvol: ext4
/dev/vg00/swapvol: swap
><fs> mount /dev/sda1 /
><fs> edit /grub/grub.conf #This will open grub.conf with your default editor
><fs> exit
```
/boot/grub/grub.conf inside Guest VM:
```
# grub.conf generated by anaconda
#
# Note that you do not have to rerun grub after making changes to this file
# NOTICE:  You have a /boot partition.  This means that
#          all kernel and initrd paths are relative to /boot/, eg.
#          root (hd0,0)
#          kernel /vmlinuz-version ro root=/dev/mapper/vg00-rootvol
#          initrd /initrd-[generic-]version.img
#boot=/dev/sda
default=0
timeout=5
#splashimage=(hd0,0)/grub/splash.xpm.gz
hiddenmenu
serial --unit=1 --speed=9600
terminal --timeout=8 console serial
title CentOS (2.6.32-431.el6.i686)
    root (hd0,0)
    kernel /vmlinuz-2.6.32-431.el6.i686 ro root=/dev/mapper/vg00-rootvol rd_NO_LUKS LANG=en_US.UTF-8 rd_LVM_LV=vg00/rootvol rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto rd_LVM_LV=vg00/swapvol  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet console=tty0 console=ttyS0,9600n8
    initrd /initramfs-2.6.32-431.el6.i686.img
```
http://www.cyberciti.biz/faq/linux-serial-console-howto/
added
```
serial --unit=1 --speed=9600
terminal --timeout=8 console serial
```
under ```hiddenmenu``` and appended ```console=tty0 console=ttyS0,9600n8``` to the ```kernel``` line.
I also commented out ```#splashimage=(hd0,0)/grub/splash.xpm.gz```.

My VM XML for learnpuppet, I had to edit ```arch='x86_64'``` to ```arch='i686'``` I did not add ```--arch=i686``` during my installation and it defaulted to host arch, even though there is only 1 vcpu.
To edit the XML use ```# virsh edit learnpuppet```.
```
<domain type='kvm'>
  <name>learnpuppet</name>
  <uuid>8d633780-c731-42ce-9a07-672d5a10c680</uuid>
  <memory unit='KiB'>2097152</memory>
  <currentMemory unit='KiB'>2097152</currentMemory>
  <vcpu placement='static'>1</vcpu>
  <os>
    <type arch='i686' machine='pc-i440fx-rhel7.0.0'>hvm</type>
    <boot dev='hd'/>
  </os>
  <features>
    <acpi/>
    <apic/>
    <pae/>
  </features>
  <clock offset='utc'/>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>restart</on_crash>
  <devices>
    <emulator>/usr/libexec/qemu-kvm</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='raw' cache='none'/>
      <source file='/home/vm/learn_puppet/learnpuppet.qcow2'/>
      <target dev='vda' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
    </disk>
    <controller type='usb' index='0'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x2'/>
    </controller>
    <controller type='pci' index='0' model='pci-root'/>
    <interface type='bridge'>
      <mac address='52:54:00:79:ce:2e'/>
      <source bridge='vbr0'/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
    </interface>
    <serial type='pty'>
      <target type='isa-serial' port='0'/>
    </serial>
    <console type='pty'>
      <target type='serial' port='0'/>
    </console>
    <input type='tablet' bus='usb'/>
    <memballoon model='virtio'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0'/>
    </memballoon>
  </devices>
</domain>
```
Once these edits are done, start the vm again.
```
# virsh start learnpuppet
```
and open console
```
# virsh console learnpuppet
```
This will show you
```
# sudo virsh console learnpuppet
Connected to domain learnpuppet
Escape character is ^]
```
pressing enter usually brings you the login prompt. However, this doesn't happen, but wait for a while I think I waited about 5 minutes before anything showed up in console.
```
# virsh console learnpuppet
Connected to domain learnpuppet
Escape character is ^]
▒putfont: PIO_FONT trying ...
........             Welcome to CentOS
.Starting udev: .....[  OK  ]
.Setting hostname learning.puppetlabs.vm:  [  OK  ]
Setting up Logical Volume Management:   2 logical volume(s) in volume group "vg00" now active
[  OK  ]
Checking filesystems
...
```
For more information I enjoy http://www.server-world.info/en/
On the left you'll be able to choose host OS, under virtualization you can find most information about virt-install, virsh, and other kvm tools.
#resources:
Check you can virtualize:
https://help.ubuntu.com/community/KVM/Installation

Installation of packages required, setup bridge through nmcli (Network Manager CLI):
http://www.server-world.info/en/note?os=CentOS_7&p=kvm

SELinux management, semanage:
http://wiki.centos.org/HowTos/KVM

Taught me about guestfish:
http://lost-and-found-narihiro.blogspot.com/2013/11/deploy-puppet-lab-vm-within-kvm.html