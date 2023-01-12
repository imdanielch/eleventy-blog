---
templateKey: blog-post
title: iptables for my minecraft server
date: 2020-04-01T15:55:16-0500
featuredpost: false
featuredimage: /static/img/blog/featured/minecraft.png
featuredimagealt:
description: My experience trying to run Virtual Machines in an Ubuntu 18.04 host with Docker.
tags:
  - kvm
  - docker
  - ubuntu
  - iptables
---
With a minecraft server set up, I want to make it so people can access the server without putting my home network in jeopardy.

Right now, the minecraft server runs on docker inside a VM with bridged network. In other words, it's in the same subnet as my home network. I thought about keeping it in another subnet but I don't have a smart switch and the server handles stuff within the home network too so it'd be preferrable that it is physically in the same subnet.

Long way to say, I can't think of a better way to set up the network itself.

![graph showing network](/static/img/blog/network-minecraft.png)

The setup is as shown on the image, everything within the home LAN. I plan on giving strangers access to the Ubuntu Server 18.04 guest. So, how do I do that without exposing my home network? My thought would be iptables.

Iptables is a widely used firewall in the linux world. It is fairly simple but seem complex due to its syntax.

I want the VM guest to be able to:

1. Access the internet through ports:
    - ssh
    - web
    - minecraft
2. Be blocked from accessing home network stations
3. (optional) can access host to create backups in a specific directory but nothing else

I have a router with iptables, and I think that's probably the best place to add rules. Another option would be to place rules on the VM host, so it can detect the destination and block if it's within the LAN.

There are two ways to define iptables rules. To `ACCEPT` connections by default and block the ones you don't want, or `DROP/REJECT` by default and then accept connections as needed. Generally, the 2nd options is better.

Another thing to keep in mind is that rules have a hierarchy by using chains. This means that it reads the rules from top to bottom, if a rule matches the packet, then it will use that rule and ignore the ones after.

### Rules on the router
I first tried adding the rules on the router. However, it didn't have any effect. My Network knowledge is weak but I think this is because when I try to ssh or ping to a local network (LAN), it doesn't need to go through the router if it doesn't have to, therefore if the rules are on the router, they never see the packets going across the LAN.

### Rules on the host

Well then, I guess host it is. In fact, my fix for [running KVM on Ubuntu 18.04 server with Docker](/blog/2020-03-18_kvm_docker/) gave me an idea of how things worked. My guest VM connects to the internet through a virtual bridge interface called `br0`. Due to docker modifying the iptables on the host with default `DROP` the packets, I have to append a rule `FORWARD -i br0 -j ACCEPT`. What this means is that any packets forwarded to/from `br0` will be let through. So what we would need to do is DROP the packets we don't want and if the packet don't match these rules, accept them.

1. I want to block packets from(source) guest VM to(destination) my LAN, except from(source) port 25565 because I want to be able to play from LAN too.

2. I want to block packets to(destination) guest VM from(source) my LAN, except to(destination) port 25565 (I also added 443 and 80 in case I want to have web interface later on).

3. accept any connection to `br0` which does not match the rules above.

After running tests without setting the IP range (so rule applies to even those from the internet), I would get an error that authentication servers were down. By using `tcpdump` I found that the minecraft server tries to connect to the internet through HTTPS(443) and without accepting RELATED,ESTABLISHED, I wasn't able to log into the server. So these are the rules I ended up with.

```
-A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -d 192.168.3.148/32 -i br0 -p tcp -m multiport ! --dports 25565,443,80 -m iprange --src-range 192.168.3.2-192.168.3.254 -j REJECT --reject-with icmp-port-unreachable
-A FORWARD -s 192.168.3.148/32 -i br0 -p tcp -m tcp ! --sport 25565,443,80 -m iprange --dst-range 192.168.3.2-192.168.3.254 -j REJECT --reject-with icmp-port-unreachable
-A FORWARD -i br0 -j ACCEPT
```

You can add these rules by typing:

```bash{promptUser: user}{promptHost: localhost}
sudo iptables <rule>
```

Quick explanation of one of the rules:

`-A FORWARD -d 192.168.3.148/32 -i br0 -p tcp -m multiport ! --dports 25565,443,80 -m iprange --src-range 192.168.3.2-192.168.3.254 -j REJECT --reject-with icmp-port-unreachable`

`-A`ppend to `FORWARD` chain `-d`estination IP `192.168.3.148/32`, at `-i`nterface `br0`, `-p`rotocol `tcp` `-m`atch `iprange`  from `--src-range 192.168.3.2-192.168.3.254` `-j`ump to `REJECT` and also `--reject-with icmp-port-unreachable` (reject ping)

You can also do INSERT with `-I` instead of `-A` and it will put the rule at the beginning of the rules list. You can also do `iptables -I CHAIN #` where `#` is the rule number where you want to insert into. You can get the rules list by
```bash{promptUser: user}{promptHost: localhost}
sudo iptables -L -v --line-numbers
```

