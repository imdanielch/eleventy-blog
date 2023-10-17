---
templateKey: blog-post
status: published
title: Setting up a server architecture.
date: 2015-03-05T17:46:31.000Z
featuredpost: false
featuredimagealt:
featuredimage: 
description:
layout: layouts/post.njk
tags:
  - '2014'
---
As you may know, businesses tend to have multiple servers, one which is for production, which is accessed by the public and clients. One for development, where all the programmers and designers work on bringing new features or improving existing ones. I believe it would be nice to have another server which would provide a testing environment. The reason being to avoid possible accidents.

In this setup, IPtables would be used to restrict communication between the 3.

Production would accept In and Out packages from the public. Reject In and Out packages from Development.

Testing would Reject all public. Accept In from Production. Accept Out to Production.

Development would Reject All from Production. Accept Out to Testing.

Developers and designers would work in Development.

Integration Team would receive the developed features and try them in testing with mock or duplicates of real data.

Once Integration team deems the feature to be safe and free of bugs, they push to production.
