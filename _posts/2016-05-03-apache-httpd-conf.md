---
title: apache配置
layout: post
tags: httpd apache
---

### 0. 简介
学习apache配置，主要是按照指令来学习，争取每天学习几个

### 1. 配置指令

#### 1.1 BrowserMatch
对不同的UA进行一些特殊的配置，包括
* nokeepalive 禁用keepalive; eg:BrowserMatch ^Mozilla nokeepalive
* force-response-1.0 对所有请求强制进行http1.0的响应
* downgrade-1.0 对所有的请求强制当作http 1.0 的请求来处理
 
例子:
* BrowserMatch "Mozilla/2" nokeepalive 
* BrowserMatch "MSIE 4\.0b2;" nokeepalive downgrade-1.0 force-response-1.0
* BrowserMatch "RealPlayer 4\.0" force-response-1.0 
* BrowserMatch "Java/1\.0" force-response-1.0 
* BrowserMatch "JDK/1\.0" force-response-1.0

#### 1.2 ErrorDocument

用来配置当出现特定的http code的时候提示文本或者转向的链接

例子：
* ErrorDocument 400 "400 了, please check"
* ErrorDocument 400 "/400.html"

#### 1.3 rewrite
rewirte主要的功能就是实现URL的跳转，它的正则表达式是基于Perl语言。
可基于服务器级的(httpd.conf)和目录级的 (.htaccess)两种方式。
使用rewrite模块前，需要先安装或加载rewrite模块。

基于服务器级的(httpd.conf)有两种方法，

* 一种是在httpd.conf的全局下直接利用RewriteEngine on来打开rewrite功能;
* 另一种是在局部里利用RewriteEngine on来打开rewrite功能

> 下面将会举例说明，需要注意的是,必须在每个virtualhost里用RewriteEngine on来打开rewrite功能。否则virtualhost里没有RewriteEngine on它里面的规则也不会生效。
> 基于目录级的(.htaccess),要注意一点那就是必须打开此目录的FollowSymLinks属性且在.htaccess里要声明RewriteEngine on。

相关配置的指令
* RewriteEngine 设置打开rewrite引擎, `RewriteEngine on`
* RewriteCond 为接下来的第一个RewriteRule，设置条件
* RewriteRule  为满足前面RewriteCond设定的条件的URL，设置跳转规则

完整的例子:

    RewriteCond %{REQUEST_URI} ^/test/
    RewriteCond %{REQUEST_URI} !^.*(.css|.js|.gif|.png|.jpg|.jpeg|.ico|.txt|.swf|.mp3|.mp4|.csv|.xls|.xlsx|.json)$
    RewriteRule /test/(.*) /test/index.php

    
#### 1.4 一些基本的设置
* `ServerRoot`: 之处服务器保存其配置、错误日志和行为日志的根目录， **路径的结尾不要加斜线**. `ServerRoot "/usr/local/apache2"`
* `PidFile`: 设置服务进程，进程号的存放文件， 相对于ServerRoot的路径，`pidFile logs/httpd.pid`
* `KeepAlive`: 设置是否开启链接复用，`KeepAlive On`
* `MaxKeepAliveRequests`: 设置单个链接可以处理的最大请求数,`MaxKeepAliveRequests 100`
* `Listen`: 设置监听得的主机以及端口号
* `ExtendedStatus`: 当调用`server-status`时，是否返回比较全面的信息`ExtendedStatus On`
* `LoadModule`: 设置一些模块加载一些DSO模式编译的模块(DSO是Dynamic Shared Objects（动态共享目标）的缩写)
* `ServerName`: 设定服务器的名字和端口号，和`Listen` 区别开
* `DocumentRoot`: 设定文档的根目录，默认情况下，所有请求都从这个目录进行应答, `DocumentRoot "/home/website/html"`
* `<Directory></Directory>`: 设置Apache可以存取目录的存取权限
* `DirectoryIndex`: 定义请求是一个目录时，Apache向用户提供服务的文件名, `DirectoryIndex index.php index.htm index.html index.html.var`
* `EnableSendfile`: 设置是否支持sendfile, sendfile介绍见[nginx sendfile配置](http://huyongde.github.io/2015/12/20/nginx-sendfile-tcp_nodelay-tcp_nopush.html)，`EnableSendfile On` 设置 支持sendfile
* 
* `ErrorLog`, `LogLevel`: 错误日志配置，`ErrorLog logs/error.log`, `LogLevel warn` 
* `CustomLog`, `LogFormat`: 指定access的记录文件，以及accessLog的格式`LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio`, `CustomLog logs/access_log combinedio`

* `ServerTokens` 设置http response header中server的信息，`Server:Apache`. 默认为“Full”， 这表示在回应头中将包含模块中的操作系统类型和编译信息。可以设为列各值中的一个： Full | OS | Minor | Minimal | Major | Prod. Full传达的信息最多，而Prod最少。



未完待续

#### 参考 
* [apache配置文件httpd.conf内容翻译](http://wiki.ubuntu.org.cn/index.php?title=Apache%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6httpd.conf%E5%86%85%E5%AE%B9%E7%BF%BB%E8%AF%91&variant=zh-hans)
