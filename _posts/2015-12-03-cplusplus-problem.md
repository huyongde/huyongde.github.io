---
layout: post
title: 写C++遇到的问题
tags: c++
---

**写C++测试程序遇到的问题**

##程序访问网络弹窗提示

弹窗内容如下:

![弹窗](/image/防火墙.png)

***解决办法***

    修改项目属性，链接器->清单文件->uac执行级别，设置为`requireAdministrator`

                  链接器->清单文件->uac绕过ui保护,设置为`是`

***修改后的问题***

第一次运行还是有弹窗，第二次运行就不会有弹窗了，**问题待解决**。

*端的开发可真是折腾人啊，快崩溃了。*
