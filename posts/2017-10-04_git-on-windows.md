---
templateKey: blog-post
status: published
title: Git on windows
date: 2017-10-04T17:50:54.619Z
featuredpost: false
featuredimagealt:
featuredimage:
description:
layout: layouts/post.njk
tags:
---
Git bash on windows allows you to generate an ssh key by using openssl.

`ssh-keygen -t rsa -b 4096 -C "your_email@example.com"` [1]

However if you try to run `git clone` it doesn't let you use the ssh key generated.

Instead you'll need to do:
`ssh-agent sh -c 'ssh-add ~/.ssh/id_rsa; git fetch user@host'` [2]

I had tried running the ssh-agent as stated by github's page but git bash would still ask me to provide password for git@host.com rather than asking for id_rsa password.

## Update:
Today I encountered this issue again and decided to look further into it.
When I installed git for Windows I chose to also have windows cmd/powershell be able to use git. I tried cloning from it and it worked.

This made me wonder why right clicking and opening Git Bash which opens a MINGW64 shell did not allow me to git clone git@host.com and still asking me for the password for git@host.com

The problem is that the URL copied from the browser is encoded in UTF-8. MINGW64 by default is not set to the right locale to take that string properly.
Right Click the title for Git Bash and select Options..., Click on Text and select en_US for the Locale and UTF8 for the Character Encoding. Apply and Save. This should fix encoding inconsistencies.




[1] https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/  
[2] https://superuser.com/a/868699

