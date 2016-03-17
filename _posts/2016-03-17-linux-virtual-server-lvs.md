---
layout: post
title: linux 服务器集群系统，LVS学习
tags: lvs load-balance
---

###参考
本文参考章文嵩博士的[LVS 服务器集群系统](http://www.linuxvirtualserver.org/zh/lvs1.html)

###LVS 简介
> LVS 是linux virtual server, 是通过一些手段，让服务可以承载大流量和大压力,  其中包括负载均衡技术和负载调度算法, 各负载均衡技术都是在网络层做一些升级。

###负载均衡技术
1. 网络地址转换实现虚拟服务器(virtual server via Network Address Translation, VS/NAT), 通过网络地址转换，
调度器重写请求报文的目的地址，根据预设的调度算法,讲去请求分配给后端真实的服务器。后端真实服务器的响应报文经过调度器是，
报文的源地址被重写，再返回给客户端，完成整个负载调度过程。
2. IP隧道实现虚拟服务器(virtual server via IP Tunneling, VS/TUN) ， 采用NAT技术时，由于请求和响应报文都需要经过调度器进行地址重写，
当客户端的请求越来越多时，调度器的处理能力和网卡都可能成为瓶颈;为了解决这个问题，调度器把请求报文通过ip隧道转发给后端真实服务器，
真实服务器的响应报文直接发给客户端.这样调度器只需要处理请求报文.一般情况下，请求报文比响应报文小的多，才有IP TUN技术后，可以大大提高
系统的吞吐量。
3. 直接路由实现虚拟服务器(Virtual Server Via Direct Routing, VS/DR), VS/DR 中，调度器通过修改请求报文的物理地址,将请求转发给真实服务器，
真实服务器的响应报文直接发给客户端。和IP 隧道技术一样DR可以大大提升系统吞吐量，但是要求调度器和真实服务器都有一个网卡连在同一个物理网段上。

> 如上简单介绍了三种IP层的负载均衡技术，详细的可以参考[LVS中的负载均衡技术](http://www.linuxvirtualserver.org/zh/lvs3.html)

###负载调度算法
1. 轮叫(round Robin)
2. 加权轮叫(Weighted Round Robin)
3. 最少链接(Least Connections)
4. 加权最少链接(Weighted Least Connections)
5. 局部最少链接(Locality-Based Least Connections)
6. 目标地址散列(destination hashing)
7. 源地址散列(source hashing)

> 负载调度算法详情可以参考[LVS的负载调度算法](http://www.linuxvirtualserver.org/zh/lvs4.html)


