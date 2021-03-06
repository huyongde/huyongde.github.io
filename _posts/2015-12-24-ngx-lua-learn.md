---
layout: post
title: nginx lua 学习
tags: shell program nginx lua
---

[**参考**](http://www.ttlsa.com/nginx/nginx-lua/)


### 1.1. 介绍

* ngx_lua – 把lua语言嵌入nginx中,使其支持lua来快速开发基于nginx下的业务逻辑
该模块不在nginx源码包中，需自行下载编译安装。使用lua 5.1（目前不支持lua 5.2） 或 luajit 2.0 。
添加lua支持后，开发复杂的模块，周期快，依然是100%异步非阻塞。
* ngx_lua 哪些人在用:
淘宝、腾讯财经、网易财经、360、去哪儿网等
CloudFlare, CNN, Wingify, Reblaze, Turner, Broadcasting System
* 该项目主要开发者：
chaoslawful Taobao, Alibaba Grp.
agentzh CloudFlare
https://github.com/chaoslawful/lua-nginx-module

### 1.2. 安装
#### 1.2.1. 安装JIT平台

通常，程序有两种运行方式：静态编译与动态直译。
静态编译的程序在执行前全部被翻译为机器码，而动态直译执行的则是一句一句边运行边翻译。
即时编译(Just-In-Time Compiler)则混合了这二者，一句一句编译源代码，但是会将翻译过的代码缓存起来以降低性能损耗。
JAVA、.NET 实现都使用即时编译以提供高速的代码执行。


注意：

*** 在nginx.conf中添加"lua_code_cache on/off"，来开启是否将代码缓存，所以每次变更".lua"文件时，必须reload nginx才可生效。仅针对"set_by_lua_file, content_by_lua_file, rewrite_by_lua_file, and access_by_lua_file"有效, 因为其他为写在配置文件中，更改代码也必须reload nginx。在生产环境下，不能禁用cache。同时在lua代码中使用"dofile" 或 "loadfie" 来加载外部lua脚本将不会对它进行缓存，应该使用"require"来代替。禁用cache，当且仅当在调式代码下。 ***

##### LuaJIT
luajit 是一个利用JIT编译技术把Lua脚本直接编译成机器码由CPU运行
版本：2.0 与 2.1
当前稳定版本为 2.0。
2.1为版本与ngx_lua将有较大性能提升，主要是CloudFlare公司对luajit的捐赠。
FFI库，是LuaJIT中最重要的一个扩展库。
1. 它允许从纯Lua代码调用外部C函数，使用C数据结构;
2. 就不用再像Lua标准math库一样，编写Lua扩展库;
3. 把开发者从开发Lua扩展C库（语言/功能绑定库）的繁重工作中释放出来;
下载编译

```
wget -c http://luajit.org/download/LuaJIT-2.0.2.tar.gz
tar xzvf LuaJIT-2.0.2.tar.gz
cd LuaJIT-2.0.2
make install PREFIX=/usr/local/luajit
echo "/usr/local/luajit/lib" > /etc/ld.so.conf.d/usr_local_luajit_lib.conf
ldconfig
#注意环境变量!
export LUAJIT_LIB=/usr/local/luajit/lib
export LUAJIT_INC=/usr/local/luajit/include/luajit-2.0

```
#### 1.2.2. NDK与Lua_module

NDK(Nginx Development Kit)模块是一个拓展Nginx服务器核心功能的模块
第三方模块开发可以基于它来快速实现
NDK提供函数和宏处理一些基本任务，减轻第三方模块开发的代码量。

```
wget -c https://github.com/simpl/ngx_devel_kit/archive/v0.2.18.tar.gz
wget -c https://github.com/chaoslawful/lua-nginx-module/archive/v0.8.6.tar.gz
tar xzvf v0.2.18
tar xzvf v0.8.6

```
#### 1.2.3. 编译安装Nginx

```
wget -c http://nginx.org/download/nginx-1.4.2.tar.gz
tar xzvf nginx-1.4.2.tar.gz
cd nginx-1.4.2
./configure --add-module=../ngx_devel_kit-0.2.18/ --add-module=../lua-nginx-module-0.8.6/
make
make install
```
### 1.3. 嵌入Lua后
#### 1.3.1. 检测版本
自己编译官方的 nginx 源码包，只需事前指定 LUAJIT_INC 和 LUAJIT_LIB 这两个环境变量。
验证你的 LuaJIT 是否生效，可以通过下面这个接口：

```
location = /lua-version {  
    content_by_lua ' 
            if jit then 
                    ngx.say(jit.version) 
                else 
                    ngx.say(_VERSION) 
            end 
        '; 
}
```
如果使用的是标准 Lua，访问 /lua-version 应当返回响应体 Lua 5.1
如果是 LuaJIT 则应当返回类似 LuaJIT 2.0.2 这样的输出。
不要使用标准lua，应当使用luajit, 后者的效率比前者高多了。
也可以直接用 ldd 命令验证是否链了 libluajit-5.1 这样的 .so 文件，例如：

```
[root@limq5 sbin]# ldd nginx | grep lua
libluajit-5.1.so.2 => /usr/local/luajit/lib/libluajit-5.1.so.2 (0x00007f48e408b000)
[root@limq5 sbin]#

```
#### 1.3.2. Hello,World
在nginx.conf中的service添加一个location。

```
location = /test {
       content_by_lua '
           ngx.say("Hello World")
       ngx.log(ngx.ERR, "err err err")
       ';
}
```

用户访问 `http://localhost/test` 将会打印出“Hello World”内容。
ngx.say 是 lua 显露給模块的接口。
类似的有 ngx.log(ngx.DEBUG, “”),可以在error.log输出调试信息。
另外也可以调用外部脚本，如同我们写php、jsp应用时,业务脚本单独组织在.php或.jsp文件中一样

```
location = /test2 {
       content_by_lua_file conf/lua/hello.lua;
}
```
文件hello.lua内容如下：

`ngx.say("Hello World")`

这里的脚本可以任意复杂,也可以使用Lua 自己的库

lua可用[模块列表](http://luarocks.org/repositories/rocks/)

安装类似yum，它也有一个仓库:

`luarocks install luafilesystem`

运行上面命令后，会编译一个 “lfs.so”, 文件，拷贝文件到nginx定义的LUA_PATH中，然后引用该
库，就可调用其中函数。

```
LUA_PATH:
lua_package_path ‘/opt/17173/nginx-ds/conf/lua/?.lua;;’
lua_package_cpath ‘/opt/17173/nginx-ds/conf/lua/lib/?.so;/usr/local/lib/?.?;;’;
其中”;;”代表原先查找范围。
```
#### 1.3.3. 同步形式，异步执行
我们假定,同时要访问多个数据源，而且,查询是没有依赖关系,那我们就可以同时发出请求
这样我总的延时, 是我所有请求中最慢的一个所用时间,而不是原先的所有请求用时的叠加

```
location = /api {
       content_by_lua '
           local res1, res2, res3 =
               ngx.location.capture_multi{
                   {"/memc"}, {"/mysql"}, {"/postgres"}
               }
           ngx.say(res1.body, res2.body, res3.body)
       ';
}
```
`ngx.location.capture` 无法跨server进行处理, 只能在同一个server下的不同location。

### 1.4. Nginx与Lua执行顺序
#### 1.4.1. Nginx顺序
Nginx 处理每一个用户请求时，都是按照若干个不同阶段（phase）依次处理的，而不是根据配置文件上的顺序。
Nginx 处理请求的过程一共划分为 11 个阶段，按照执行顺序依次是

```
post-read、server-rewrite、find-config、rewrite、post-rewrite、 
preaccess、access、post-access、try-files、content、log.

```

* post-read:
读取请求内容阶段
Nginx读取并解析完请求头之后就立即开始运行
例如模块 ngx_realip 就在 post-read 阶段注册了处理程序，它的功能是迫使 Nginx 认为当前请求的来源地址是指定的某一个请求头的值。
* server-rewrite
Server请求地址重写阶段
当 ngx_rewrite 模块的set配置指令直接书写在 server 配置块中时，基本上都是运行在 server-rewrite 阶段
* find-config
配置查找阶段
这个阶段并不支持 Nginx 模块注册处理程序，而是由 Nginx 核心来完成当前请求与 location 配置块之间的配对工作。
* rewrite
Location请求地址重写阶段
当 ngx_rewrite 模块的指令用于 location 块中时，便是运行在这个 rewrite 阶段。
另外，ngx_set_misc(设置md5、encode_base64等) 模块的指令，还有 ngx_lua 模块的 set_by_lua 指令和 rewrite_by_lua 指令也在此阶段。
* post-rewrite
请求地址重写提交阶段
由 Nginx 核心完成 rewrite 阶段所要求的“内部跳转”操作,如果 rewrite 阶段有此要求的话。
* preaccess
访问权限检查准备阶段
标准模块 ngx_limit_req 和 ngx_limit_zone 就运行在此阶段，前者可以控制请求的访问频度，而后者可以限制访问的并发度。
* access
访问权限检查阶段
标准模块 ngx_access、第三方模块 ngx_auth_request 以及第三方模块 ngx_lua 的 access_by_lua 指令就运行在这个阶段。
配置指令多是执行访问控制性质的任务，比如检查用户的访问权限，检查用户的来源 IP 地址是否合法
* post-access
访问权限检查提交阶段
主要用于配合 access 阶段实现标准 ngx_http_core 模块提供的配置指令 satisfy 的功能。
satisfy all(与关系)
satisfy any(或关系)
* try-files
配置项try_files处理阶段
专门用于实现标准配置指令 try_files 的功能
如果前 N-1 个参数所对应的文件系统对象都不存在，try-files 阶段就会立即发起“内部跳转”到最后一个参数（即第 N 个参数）所指定的 URI.
* content
内容产生阶段
Nginx 的 content 阶段是所有请求处理阶段中最为重要的一个，因为运行在这个阶段的配置指令一般都肩负着生成“内容”并输出 HTTP 响应的使命。
* log
日志模块处理阶段
记录日志
淘宝有开放一个nginx开发手册，里面包含很多有用的资料
http://tengine.taobao.org/book/
作者的google论坛：
https://groups.google.com/forum/#!forum/openresty

#### 1.4.2. Lua顺序
Nginx下Lua处理阶段与使用范围：

```
init_by_lua            http
set_by_lua             server, server if, location, location if
rewrite_by_lua         http, server, location, location if
access_by_lua          http, server, location, location if
content_by_lua         location, location if
header_filter_by_lua   http, server, location, location if
body_filter_by_lua     http, server, location, location if
log_by_lua             http, server, location, location if
timer
```

* init_by_lua
在nginx重新加载配置文件时，运行里面lua脚本，常用于全局变量的申请。
例如lua_shared_dict共享内存的申请，只有当nginx重起后，共享内存数据才清空，这常用于统计。

* set_by_lua:
设置一个变量，常用与计算一个逻辑，然后返回结果
该阶段不能运行Output API、Control API、Subrequest API、Cosocket API
* rewrite_by_lua:
在access阶段前运行，主要用于rewrite
* access_by_lua:
主要用于访问控制，能收集到大部分变量，类似status需要在log阶段才有。
这条指令运行于nginx access阶段的末尾，因此总是在 allow 和 deny 这样的指令之后运行，虽然它们同属 access 阶段。
* content_by_lua:
阶段是所有请求处理阶段中最为重要的一个，运行在这个阶段的配置指令一般都肩负着生成内容（content）并输出HTTP响应。
* header_filter_by_lua:
一般只用于设置Cookie和Headers等
该阶段不能运行Output API、Control API、Subrequest API、Cosocket API
* body_filter_by_lua:
一般会在一次请求中被调用多次, 因为这是实现基于 HTTP 1.1 chunked 编码的所谓“流式输出”的。
该阶段不能运行Output API、Control API、Subrequest API、Cosocket API
* log_by_lua:
该阶段总是运行在请求结束的时候，用于请求的后续操作，如在共享内存中进行统计数据,如果要高精确的数据统计，应该使用body_filter_by_lua。
该阶段不能运行Output API、Control API、Subrequest API、Cosocket API
