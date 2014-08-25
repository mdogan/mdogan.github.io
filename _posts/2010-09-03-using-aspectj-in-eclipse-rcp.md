---
layout: post
title:Using AspectJ in Ecplipse RCP
published: true
tags: java, eclipse, aspectj
comments: false
---

If you try to export an AspectJ enabled Eclipse RCP 3.6, you will notice that none of your pretty aspects is compiled.
Before Eclipse 3.6 there was an export option as **"Export Product with AspectJ"**.

But since version 3.6 you will not see that option. Instead you should just add AspectJ compiler adapter into your `build.properties` file:

```
compilerAdapter = org.eclipse.ajdt.core.ant.AJDT_AjcCompilerAdapter
sourceFileExtensions = *.java, *.aj
```
