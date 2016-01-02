---
layout: post
title: vim 插件管理 vundle
tags: shell program
---

##1. 简介

    vim 是程序员最受欢迎的coding神器，没有之一。 合理使用插件可以做到事半功倍的效果。 
vundle 把git整合到插件管理中，用户需要做的只是去Github上找到自己想要的插件的名字，安装，更新和卸载都可有vundle来完成了。
虽然去发现一个好的插件仍然是一个上下求索的过程，但是用户已经可以从安装配置的繁琐过程解脱了。 

* [Vundle git repo](https://github.com/VundleVim/Vundle.vim) 介绍了如何安装vundle和通过vundle来安装vim插件
* [vundle.txt](https://github.com/VundleVim/Vundle.vim/blob/master/doc/vundle.txt)

##2. 安装&&使用

* 下载vundle：

```
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```
* 下载完成之后，在vimrc中添加配置：

```
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'

" The following are examples of different formats supported.
" Keep Plugin commands between vundle#begin/end.
" plugin on GitHub repo
Plugin 'tpope/vim-fugitive'
" plugin from http://vim-scripts.org/vim/scripts.html
Plugin 'L9'
" Git plugin not hosted on GitHub
Plugin 'git://git.wincent.com/command-t.git'
" git repos on your local machine (i.e. when working on your own plugin)
Plugin 'file:///home/gmarik/path/to/plugin'
" The sparkup vim script is in a subdirectory of this repo called vim.
" Pass the path to set the runtimepath properly.
Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" Avoid a name conflict with L9
Plugin 'user/L9', {'name': 'newL9'}

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line

```
* vim 任意打开文件， 运行 `:PluginInstall` 就会安装vim配置中配置Plugin配置的需要安装的插件
    * Plugin 'VundleVim/Vundle.vim' 配置vundle
    * 另外配置了其他一些vim 插件， 如L9等。

* 对添加的vim 配置的一些解释

```
set nocompatible               " be iMproved
 filetype off                   " required!

 set rtp+=~/.vim/bundle/vundle/
 call vundle#rc()

 " let Vundle manage Vundle
 " required! 
 Bundle 'gmarik/vundle'

 " My Bundles here:
 #以后你想安装什么插件可以写在下面
 "
 " original repos on github 
#如果你的插件来自github，写在下方，只要作者名/项目名就行了
 Bundle 'tpope/vim-fugitive' #如这里就安装了vim-fugitive这个插件
 Bundle 'Lokaltog/vim-easymotion'
 Bundle 'rstacruz/sparkup', {'rtp': 'vim/'}
 Bundle 'tpope/vim-rails.git'
 " vim-scripts repos
#如果插件来自 vim-scripts，你直接写插件名就行了
 Bundle 'L9'
 Bundle 'FuzzyFinder'
 " non github repos
#如使用自己的git库的插件，像下面这样做
 Bundle 'git://git.wincent.com/command-t.git'
 " git repos on your local machine (ie. when working on your own plugin)
 Bundle 'file:///Users/gmarik/path/to/plugin'
 " ...

 filetype plugin indent on     " required!
#下面是 vundle的一些命令代会会用到
 "
 " Brief help
 " :BundleList          - list configured bundles
 " :BundleInstall(!)    - install(update) bundles
 " :BundleSearch(!) foo - search(or refresh cache first) for foo
 " :BundleClean(!)      - confirm(or auto-approve) removal of unused bundles
 "
 " see :h vundle for more details or wiki for FAQ
 " NOTE: comments after Bundle command are not allowed..
```  





