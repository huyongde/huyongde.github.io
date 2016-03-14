---
layout: post
title: mac 安装php7
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

这个问题解决后，又出现了一个GD相关的错误，搞了会没搞定。


***最后google出来一个安装方式[MAC OS 一行代码安装php7](http://php-osx.liip.ch/)

安装方式是`curl -s http://php-osx.liip.ch/install.sh | bash -s 7.0` ***

等待一段时间后，终于安装成功了，


```
[ 10:15上午 ]  [ huyongde@huyongde:/usr/local/php5/bin(master✔) ]
 $ ./php -v
PHP 7.0.4 (cli) (built: Mar 10 2016 14:34:46) ( NTS )
Copyright (c) 1997-2016 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies
    with Xdebug v2.4.0RC3, Copyright (c) 2002-2015, by Derick Rethans
```

建议先翻墙再执行命令。终于安装好了，迫不及待把玩一下php7.






