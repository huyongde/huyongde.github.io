---
layout: post
title: 大数据行存储还是列存储？
tags: data shell
---

介绍数据行存储和列存储的优缺点

##简介

**面向行**的数据存储架构更适用于[OLTP](https://en.wikipedia.org/wiki/Online_transaction_processing)-频繁交互事务的场景;

**面向列**的数据存储架构更适用于[OLAP](https://en.wikipedia.org/wiki/Online_analytical_processing#Overview_of_OLAP_systems)-(如数据仓库)这样在海量数据（(可能达到 terabyte规模)）中进行有限复杂查询的场景。
 
###[OLTP和OLAP介绍](http://blog.csdn.net/zhangzheng0413/article/details/8271322/)

数据处理大致可以分成两大类：联机事务处理OLTP（on-line transaction processing）、联机分析处理OLAP（On-Line Analytical Processing）。

OLTP是传统的关系型数据库的主要应用，主要是基本的、日常的事务处理，例如银行交易。

OLAP是数据仓库系统的主要应用，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果。 

OLTP 系统强调数据库内存效率，强调内存各种指标的命令率，强调绑定变量，强调并发操作；

OLAP 系统则强调数据分析，强调SQL执行市场，强调磁盘I/O，强调分区等。 

##参考

[大数据存储选择-行存储还是列存储](http://www.infoq.com/cn/articles/bigdata-store-choose)

[行存储和列存储的比较](http://www.douapp.com/post/525203)


