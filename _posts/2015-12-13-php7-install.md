---
layout: post
title: php7 安装 install
tags: shell program
---

##简介

安装php,把玩PHP新特性

##下载源代码

php7的[**下载地址**](http://cn2.php.net/distributions/php-7.0.0.tar.gz) 点击下载即可

##安装

运行./configure 并设置自己的选项,我的选项如下：

```

```
需要安装的一些库:

* `brew install homebrew/dupes/libiconv`
* `brew install homebrew/dupes/zlib` , 之后配置 `--with-zlib-dir=/usr/local/Cellar/zlib/1.2.8 \`
* `brew install curl`,之后配置`--with-curl=/usr/local/Cellar/curl/7.46.0`
* `brew install homebrew/versions/libpng12` ， 之后配置 ``


**未完待续**
