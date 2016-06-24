---
layout: post
title: sublime text 3  扩展包安装遇到的问题
tags: sublime package
---

### 简介
最近在用sublime text3, 通过package control 可以安装的扩展包真的太多了，很强大
但是在安装的时候遇到了一些问题

### 问题
1. 安装完成SublimeCodeIntel后，使用时不成功，通过`C+~` 调出来命名窗口，发现 **OSError: [WinError 6] 句柄无效** 的错误。 这个问题google了下，stackoverflow上面有个相关的解决方案 [链接](http://stackoverflow.com/questions/3028786/how-can-i-fix-error-6-the-handle-is-invalid-with-pyserial)，说是python安装的版本不对.
我的系统是win7 64, 需要安装64位的python, 下载了[安装包](https://www.python.org/ftp/python/2.7.12/python-2.7.12rc1.amd64.msi)
重新安装后，重新设置了环境变量PATH， 再次进行js编程时就可以自动提示并且补全了。
2. 安装了多个插件之后，sublime经常性卡顿，尝试设置了下gitgutter的user配置， 添加配置

```
{
    "non_blocking" : "true",
    "live_mode" : "false"
}
```

修改之后稍微有些改观，可能还有其他的插件导致这个问题。

> 后续有问题继续更新

### 配置前端开发的扩展包

#### [参考](http://www.cnblogs.com/hykun/p/sublimeText3.html)

上篇博文中包括了emmet,SublimeCodeIntel, SideBar等的扩展插件。
