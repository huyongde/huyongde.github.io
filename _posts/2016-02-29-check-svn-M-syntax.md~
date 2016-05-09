---
layout: post
title: 检查当前svn路径下面修改的PHP文件是否有语法错误
tags: php svn syntax
---

######直接上代码

一行`shell`代码搞定，代码如下：

```
cd $1;for i in `svn st | awk '$1=="M" || $1=="A"{print $2}'`; do php -l $i;done
```


> 使用说明:$1是要检查的文件夹路径，借助php -l参数，来完成检查 
