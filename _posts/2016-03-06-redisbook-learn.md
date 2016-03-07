---
layout: post
title: 学习redisbook, 了解redis设计和原理
tags: redis 
---

###redisbook介绍

> redisbook是huangjianhong大神写的，介绍redis设计和实现的一本，redis入门提高很好的一本书。

###相关学习资源

* [带注释的redis源码](https://github.com/huangz1990/redis-3.0-annotated)
* [redisbook网站](redisbook.com)
* [redis pdf 版本下载](http://pan.baidu.com/s/1jGXhgvs)  pdf版本已经大概看了一遍，确实对redis设计以及底层实现有了一些了解，**应对面试足够了。**

###redisbook学习心得

##reis内部数据结构

####0.简单动态字符串(sds)
* redis的字符串是使用sds来表示的，而不是c字符串
* 和 c字符串比较，sds有以下特性：
    * 高效地获得字符串长度,O(1)
    * 高效的执行追加操作
    * 二进制安全
* sds为追加操作进行追加优化，加快追加操作的速度，降低内存分配的次数，代价是多占用一些内存，而且这些内存不会主动释放

####1.双端列表

未完下周继续

