---
layout: post
title: "VIM PHP开发相关配置整理"
tags: vim php
---

### 0. 简介

vim配置弄过很多，关于编码的，缩进的，taglist, nerdtree， vunble插件管理器等的配置。假期有空重新整理学习学习

### 1. 相关配置
####1.1 设置保存vimrc自动生效

* `autocmd! bufwritepost .vimrc source ~/.vimrc` 设置之后，
* 当你修改~./vimrc并执行:w保存时可能会报 `E174: Command already exists: add ! to replace it`

superuser有关于这个问题的解决方案，[解决source ~/.vimrc E174](http://superuser.com/questions/830132/sourcing-the-vimrc-gives-e174-error)
主要就是你需要在command后面加上!来做vim的相关配置。

**:help E174**的结果是

```
:com[mand][!] [{attr}...] {cmd} {rep}
                        定义一个用户命令。命令的名字是 {cmd}，而替换的文本是
                        {rep}。该命令的属性 (参考下面) 是 {attr}。如果该命令已
                        存在，报错，除非已经指定了一个 !，这种情况下命令被重定
                        义。
```

> 设置完成上面两步之后，就可以在修改了~/.vimrc并保存的时候，自动执行`source ~/.vimrc` 来使配置生效了。

#### 1.2 vim配置help的中文文档

* 首先下载vimdoc,[下载链接](http://jaist.dl.sourceforge.net/project/vimcdoc/vimcdoc/vimcdoc-1.9.0.tar.gz) ,vimdoc官网[主页](http://vimcdoc.sourceforge.net/), 
下载后直接把压缩包放入`~/.vim/doc` 目录，不存在的话进行创建`mkdir -p ~/.vim/doc`
* 执行`:helptags ~/.vim/doc`
* 在`~/.vimrc ` 中设置 `set helplang=cn`

> 上面三步一步不少的执行完后，就可以愉快的看到中文版的vim help了， 可以help autocmd验证下


#### 1.3 设置php文件保存的时候自动进行php -l的php语法检查
* 第一种方法，不借助插件的配置`autocmd! BufWritePost *.php :!php -l %` 
是通过php -l来执行，做了一次自动命令的来实现的,当写php文件的时候，自动执行`:!php -l %`
* [vim 各语言的语法检查插件](https://github.com/scrooloose/syntastic)  [相关wiki](https://github.com/scrooloose/syntastic/wiki/Syntax-Checkers) 
* phplint 进行` Bundle 'nrocco/vim-phplint'` 和 `autocmd! BufWritePost *.php :phplint` 两个配置，第一个配置是安装phplint插件，第二个配置是在文件写入时自动执行phplint插件
[Phplint github](https://github.com/nrocco/vim-phplint)

> 最终我选择的Phplint，显示的结果更友好些。



### 参考
* [配置一个高效开发PHP的VIM](http://www.cnblogs.com/mo-beifeng/archive/2011/09/07/2169994.html)
