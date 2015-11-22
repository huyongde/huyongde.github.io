---
layout: post 
title: jekyll 入门
---

主题
======

<h1> {{ page.title }} </h1>

-----

##jekyll 安装

通过gem来安装，gem是`perl`语言各种扩展包得管理工具

安装命令如下：

```
sudo gem install jekyll
```

##初始化站点

```
mdkir jekyll_site
jekyll new jekyll_site
```
执行完 jekyll new jekyll_site

执行`cd jekyll_site; tree`可以看到站点的目录结构,如下:

![jekyll站点目录](/image/tree_re.png)

通过`jekyll server` 就可以启动本地服务查看站点

jekyll server 执行结果如下：

```
Configuration file: /Users/huyongde/tmp/jekyll_site/_config.yml
            Source: /Users/huyongde/tmp/jekyll_site
       Destination: /Users/huyongde/tmp/jekyll_site/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 0.259 seconds.
 Auto-regeneration: enabled for '/Users/huyongde/tmp/jekyll_site'
Configuration file: /Users/huyongde/tmp/jekyll_site/_config.yml
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```


##参考

[jekyll 安装和初始化站点](http://jobinson.ga/%E5%BB%BA%E7%AB%99%E4%B9%8B%E8%B7%AF/2014/04/27/%E4%BD%BF%E7%94%A8jekyll%E7%94%9F%E6%88%90%E9%9D%99%E6%80%81%E7%AB%99/)

[jekyll 入门](http://trefoil.github.io/2013/10/05/jekyll.html)

*{{ page.date | date_to_string }}*
