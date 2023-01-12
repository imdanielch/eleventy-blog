---
templateKey: blog-post
status: published
title: Ionic framework 2015-06-20~2015-07-xx
date: 2015-08-16T05:56:40.744Z
featuredpost: false
featuredimagealt:
featuredimage:
description:
tags:
  - Ionic
---
## Tutorials

Songhop build:
https://thinkster.io/ionic-framework-tutorial/

Lynda's Ionic tutorial:

part1: https://www.youtube.com/watch?v=93K6xtYxMgg

part2: https://www.youtube.com/watch?v=uEziLWMN2VU

Angularjs Scope:

https://github.com/angular/angular.js/wiki/Understanding-Scopes

Angularjs Sharing data across controllers:

http://onehungrymind.com/angularjs-sticky-notes-pt-1-architecture/

Autogrowing textarea height:

http://forum.ionicframework.com/t/auto-growing-textarea-in-ionic/6213

##Troubleshooting:

Getting data from a service to the controller in order for states to access them:
http://stackoverflow.com/questions/31100796/sharing-services-variables-across-controllers-angular-ionic
Network error when emulating Android:
http://abou-kone.com/2015/04/25/ionic-there-was-a-network-error-when-running-on-genymotion/

when running "ionic build android" it claims "java -version" failed, but when you type it out, it works. This however doesn't mean java JDK is installed. ```sudo apt-get install default-jdk``` can fix the problem.

Also, if your function fails when testing in browser, make sure it's not cordova related. Otherwise it might only work on device or emulation.

Cordova not letting you use geo:, tel:, or any external links?
https://github.com/apache/cordova-plugin-whitelist

##Android Support:
http://stackoverflow.com/questions/24522921/how-do-you-build-and-deploy-to-an-older-version-of-android-for-ionic-cordova

Geo URI:
https://developer.android.com/guide/components/intents-common.html

AngularJS structure:
http://mcgivery.com/structure-of-an-ionic-app/
