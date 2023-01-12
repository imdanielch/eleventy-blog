---
templateKey: blog-post
status: published
title: 2017-04-29 Setting samba shares
date: 2017-04-29T06:36:39.213Z
featuredpost: false
featuredimagealt:
featuredimage: 
description:
tags:
---
Samba shares have always been a little confusing. Their authentication system doesn't use the UNIX accounts.
This is a short guide for Ubuntu 16.04

1. Install samba:
    ```bash
    sudo apt update
    sudo apt install samba
    ```

2. Set a username and password:
`sudo smbpasswd -a <user_name>`

3. Figure out what directory you wish to share.
Mine was `/media/media5/`

4. Copy the samba configuration to make a backup
`sudo cp /etc/samba/smb.conf ~`
5. and edit the config file to fit your needs
`sudo vim /etc/samba/smb.conf`
    I wanted to make the share read only and write for my user. This is the settings I used.
    ```
    [share]
    path = /media/media5
    browseable = yes
    guest ok = yes
    read only = yes
    write list = daniel bob steve
    create mask = 755
    ```
    guest ok allows anonymous users.
    read only will make everyone have read only permissions.
    write list makes the users listed to be able to write too. However these users are those you set 
    up with `smbpasswd` and not the UNIX accounts.

6. restart samba
`sudo systemctl restart smbd`

you can use `testparm` to check for syntax errors.

### Resource:
[Ubuntu help documentation](https://help.ubuntu.com/community/How%20to%20Create%20a%20Network%20Share%20Via%20Samba%20Via%20CLI%20%28Command-line%20interface/Linux%20Terminal%29%20-%20Uncomplicated%2C%20Simple%20and%20Brief%20Way%21)
