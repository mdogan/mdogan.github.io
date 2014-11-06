---
layout: post
title: Installing IBM JDK for newbies
published: true
tags: jdk ibm mailcap linux
comments: true
---

Installing [IBM JDK](http://www.ibm.com/developerworks/java/jdk/linux/download.html) on a linux box is a trivial task or *should be a trivial task*. But when you get a message like

> The installer cannot run on your configuration. It will now quit.

you find yourself staring at console without a clue.

<!--excerpt-->

When you search above *not-really-helping* message on your favourite search engine, you'll see that most of the answers are just saying to run installer as root.

If that solves your issue, please don't read after this point. We are done!

But... You already know how to install a package. You're not a newbie linux user anymore, right?

After some more search I saw an [IBM support ticket](http://www-01.ibm.com/support/docview.wss?uid=swg1IV45040), which just says installer requires `/etc/mailcap` and `/etc/mime.types`
files and either install [**Mailcap**](http://en.wikipedia.org/wiki/Mailcap) package or create dummy `/etc/mailcap` and `/etc/mime.types` files.


> #### Local fix
To make install operation successful either of  the following
options can be applied.

>* Install mailcap RPM package before attempting the JDK
installation.

>* Create dummy files /etc/mailcap and /etc/mime.types files
manually before installing the JDK

> #### Problem summary
The problem is caused when /etc/mailcap & /etc/mime.types file
are not present in the machine where the installer is being run.
These files are used to write the java web start entries during
jdk installation. The installer will fail silently if these
files are not present in the /etc directory of the system.
