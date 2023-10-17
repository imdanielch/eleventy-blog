---
templateKey: blog-post
status: published
title: compiled nginx in ubuntu 16.04
date: 2018-06-09T19:09:08.051Z
featuredpost: false
featuredimagealt:
featuredimage: 
description:
layout: layouts/post.njk
tags:
---
Compiling nginx in ubuntu and want it to work with systemd?

Installed location was `/usr/local/nginx/` You'd need to change
This is the `/lib/systemd/system/nginx.service` file and then run
`# systemctl daemon-reload`

```ini
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
PIDFile will depend on what you put in `/usr/local/nginx/conf/nginx.conf` (default is `log/nginx.pid` so potentially you can just have `PIDFile=/usr/local/nginx/log/nginx.pid`)
Then `systemctl start nginx` and `systemctl status nginx` to confirm it running. If you are happy with the results, enable it to make it run on boot `systemctl enable nginx`
# References:
[1] https://github.com/arut/nginx-rtmp-module  
[2] https://github.com/arut/nginx-rtmp-module/wiki/Getting-started-with-nginx-rtmp  
[3] https://www.nginx.com/resources/wiki/start/topics/examples/systemd/