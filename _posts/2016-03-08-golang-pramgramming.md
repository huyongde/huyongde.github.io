---
layout: post
title: go编程中自己遇到的问题
tags: go golang
---

####总结遇到的问题

1. 在定义结构体的时候尽量把结构体元素的key的首字母大写，不大写是不能导出的，不能导出就导致你在对一个结构体进行json的时候获得的json串是空的。

2.