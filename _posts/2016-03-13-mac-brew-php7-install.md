---
layout: post
title: mac上通过homebrew 安装php7
tags: brew php
---

###homebrew

> The missing package manager for OS X

homebrew可以用来在mac上安装一些软件和库，比如mysql, nginx, redis等，它会把所需要的依赖给你自动安装。

homebrew有基本的几个命令

* brew install xxx
* brew search xxx
* brew update
* brew unintall xxx

homebrew具体请参考官网[homebrew home](http://brew.sh/)

###brew来安装php

安装命令是:`brew install homebrew/php/php70`

安装过程中会遇到`configure: error: Cannot find libz` 的错误，

google了把，发现了个解决方案[cannot find libz](http://codex16.com/mac-osx-brew-install-php56-cannot-find-libz/)




