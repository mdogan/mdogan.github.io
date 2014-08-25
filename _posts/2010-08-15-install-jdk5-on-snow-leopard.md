---
layout: post
title: Install JDK 5 on Snow Leopard
published: true
comments: false
---

Download Leoapard (Mac OS X 10.5) JDK from [here](http://tedwise.com/files/java.1.5.0-leopard.tar.gz)

Extract gzip archive.

```sh
tar -xvzf java.1.5.0-leopard.tar.gz
```

Move extracted folder to java framework installation folder. You need to do this as root user.

```sh
sudo mv 1.5.0 /System/Library/Frameworks/JavaVM.framework/Versions/1.5.0-leo
```

Go into java versions folder

```sh
cd /System/Library/Frameworks/JavaVM.framework/Versions/
```

Remove old java 5 sym links

```sh
sudo rm 1.5.0
sudo rm 1.5
```

Create new symlinks to 1.5.0-leo

```sh
sudo ln -s 1.5.0-leo 1.5.0
sudo ln -s 1.5.0 1.5
```
