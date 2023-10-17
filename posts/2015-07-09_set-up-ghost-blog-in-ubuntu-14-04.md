---
templateKey: blog-post
status: published
title: Set up Ghost blog in Ubuntu 14.04
date: 2015-07-09T06:41:37.449Z
featuredpost: false
featuredimagealt:
featuredimage:
description:
layout: layouts/post.njk
tags:
  - ghost-tag
  - ghost-blog
  - ubuntu
  - setup-tag
  - fresh
---
Here I'd like to walkthrough how to setup Ghost blog 0.5.10 with Apache2 and MySQL. Which is the setup I currently have. I may also write up how to set it up with NginX and SQLite later on.

# Setting up Ghost blog 0.5.10 with Apache2 and MySQL

System Info:

Fresh install of Ubuntu 14.04 64bit server.

```
$ cat /etc/*-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=14.04
DISTRIB_CODENAME=trusty
DISTRIB_DESCRIPTION="Ubuntu 14.04.2 LTS"
NAME="Ubuntu"
VERSION="14.04.2 LTS, Trusty Tahr"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 14.04.2 LTS"
VERSION_ID="14.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
```

```
$ uname -ri
3.16.0-31-generic x86_64
```

Install NVM to get NodeJS:

```
curl https://raw.githubusercontent.com/creationix/nvm/v0.24.0/install.sh | bash
```

Ghost Blog currently requires NodeJS version 0.10.x I have found 0.10.24 to work so I stuck to it.

Use NVM to install NodeJS.

```
$ nvm install 0.10.24
$ nvm alias default 0.10.24
```

Restart your terminal emulator to have nvm working.

update apt-get and install apache2 and mysql-server:

```
$ sudo apt-get update && sudo apt-get install apache2 mysql-server -y
```

Follow MySQL root password instructions to finalize installation.

Download Ghost blog from their website (this download is for 0.5.10, check [Ghost blog](https://ghost.org/) for the latest version:

```
$ curl -L https://ghost.org/zip/ghost-0.5.10.zip -o ghost-0.5.10.zip && unzip ghost-0.5.10.zip -d ./ghost-0.5.10 && cd ghost-0.5.10 && npm install --production
```

`npm install --production` will install all necessary packages for production. if you want to get all packages for developing ghost, just do `npm install`.

### Configure MySQL:

Login to mysql client as root

```
$ mysql -uroot -p
Enter password:
```

create user and database for ghost:

```
mysql> CREATE DATABASE ghostdb;
mysql> CREATE USER 'ghost'@'localhost' IDENTIFIED BY '<password>';
mysql> GRANT ALL on ghostdb.* to 'ghost'@'localhost';
mysql> FLUSH PRIVILEGES;
```
### Configure Ghost config file at ~/ghost-0.5.10/config.js:

File doesn't exist yet so we'll copy the example:

```
$ cp config.example.js config.js
```

and now edit, I use vim:

```
$ vim config.js
```

MySQL config: http://support.ghost.org/config/#database

Config file is separated into 3 sections: production, development, and testing. There's also testing-mysql and testing-pg which are used by Travis, an Automated testing run through GitHub.

```
production: {
  url: 'http://192.168.3.76/',
  mail: {},
  database: {
    client: 'mysql',
    connection: {
      host     : '127.0.0.1',
      user     : 'ghost',
      password : '<password>',
      database : 'ghostdb',
      charset  : 'utf8'
    }
  },
  server: {
  // Host to be passed to node's `net.Server#listen()`
  host: '127.0.0.1',
  // Port to be passed to node's `net.Server#listen()`, for iisnode set this to `process.env.PORT`
  port: '2368'
  }
}
```

`url` should point to the domain assigned to the IP. Since this is for an internal testing server I just used the LAN IP address.

`database` will need to be edited completely.

Under `'testing-mysql'` you can see an example of how to configure it, or follow the snippet above as a guide.

Note: if you wish to access ghost without apache2 from LAN and not rerouted from apache2 or nginx, change `host: '127.0.0.1'` to the IP assigned to the server. Otherwise, you can keep it as 127.0.0.1, requests will be from apache2 or nginx and that's localhost (127.0.0.1).

I believe production and testing are supposed to be set with different databases in case something gets corrupt during testing. Since this is all testing, I just used the same database.

### Configure apache2:

we need to make a config file for the blog in sites-available and enable it, along with other mods and optimize mpm-prefork.
enable proxy pass mod:

```
$ sudo a2enmod proxy && sudo a2enmod proxy_http
$ sudo service apache2 restart
```

disable the default site config:

```
$ sudo a2dissite 000-default.conf
```

make copy or create a new config file for ghost:

```
$ sudo cp 000-default.conf ghost.conf
$ sudo vim ghost.conf
```

```
<VirtualHost *:80>
  # The ServerName directive sets the request scheme, hostname and port that
  # the server uses to identify itself. This is used when creating
  # redirection URLs. In the context of virtual hosts, the ServerName
  # specifies what hostname must appear in the request's Host: header to
  # match this virtual host. For the default virtual host (this file) this
  # value is not decisive as it is used as a last resort host regardless.
  # However, you must set it for any further virtual host explicitly.
  # ServerName ghost.local

  ServerAdmin webmaster@localhost

  # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
  # error, crit, alert, emerg.
  # It is also possible to configure the loglevel for particular
  # modules, e.g.
  #LogLevel info ssl:warn

  <ifmodule mod_proxy.c>
    ProxyRequests Off
    <Location />
      ProxyPreserveHost on
      ProxyPass http://127.0.0.1:2368/
      ProxyPassReverse http://127.0.0.1:2368/
    </Location>
  </ifmodule>

  ErrorLog ${APACHE_LOG_DIR}/ghost_error.log
  CustomLog ${APACHE_LOG_DIR}/ghost_access.log combined

  # For most configuration files from conf-available/, which are
  # enabled or disabled at a global level, it is possible to
  # include a line for only one particular virtual host. For example the
  # following line enables the CGI configuration for this host only
  # after it has been globally disabled with "a2disconf".
  #Include conf-available/serve-cgi-bin.conf
</VirtualHost>
```

uncomment `ServerName` and enter your domain pointed to the blog.

add:

```
ProxyRequests Off
<ifmodule mod_proxy.c>
  <Location />
    ProxyPreserveHost on
    ProxyPass http://127.0.0.1:2368/
    ProxyPassReverse http://127.0.0.1:2368/
  </Location>
</ifmodule>
```

Disable `ProxyRequests` to disable forward proxy, this can be a security risk.
`ifmodule` means if the module mod_proxy is enabled, use these settings.

A reverse proxy is activated using the ProxyPass directive or the [P] flag to the RewriteRule directive. It is not necessary to turn ProxyRequests on in order to configure a reverse proxy.

ProxyPassReverse Adjusts the URL in HTTP response headers sent from a reverse proxied server.

For more information: [http://httpd.apache.org/docs/2.2/mod/mod_proxy.html#x-headers](http://httpd.apache.org/docs/2.2/mod/mod_proxy.html#x-headers)

prepend the log files with ghost for easier debugging later on:

```
ErrorLog ${APACHE_LOG_DIR}/ghost_error.log
CustomLog ${APACHE_LOG_DIR}/ghost_access.log combined
```

save and close, then enable the site:

```
$ sudo a2ensite ghost.conf && sudo service apache2 reload
```

### Start ghost:

`NODE_ENV=production npm start` or `npm start --production`
`Ctrl + C` to exit out.

### Automate starting Ghost blog:

We'll use pm2 to manage our ghost blog:

```
$ npm install pm2 -g
```

Start ghost blog with pm2 from ./ghost-0.5.10:

```
$ NODE_ENV=production pm2 start index.js --name ghost
```

to stop `pm2 stop ghost`

to restart `pm2 restart ghost`

### Resources:

NVM GitHub: https://github.com/creationix/nvm

MySQL config: http://support.ghost.org/config/#database

Apache2 Proxy info: [http://httpd.apache.org/docs/2.2/mod/mod_proxy.html#x-headers](http://httpd.apache.org/docs/2.2/mod/mod_proxy.html#x-headers)

# Making A Ghost theme from scratch.

This includes the use of schema.org, and other SEO optimizations.

http://webdesign.tutsplus.com/articles/adding-ghost-template-tags-and-markup--webdesign-15803
