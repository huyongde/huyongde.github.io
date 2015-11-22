---
layout: default
title: jekyll 和 markdown 入门
---

主题
======

<h1> {{ page.title }} </h1>

##内容列表
* [主题](#主题)
* [参考](#参考)

##段落
通过空行来形成不同的段落

##标题
\# The largest heading (an <h1> tag)

\#\# The second largest heading (an <h2> tag)

.....

\#\#\#\#\#\# The 6th largest heading (an <h6> tag)

##引用(blockquotes)
文字前面加上\>，

>这里是个引用

##文字加粗&&斜体
*这里是斜体* (文字前后都加上\*,可以使文字变成斜体）

**这里是粗体** （文字前后都加2个\*,可以变为粗体）

##列表
###无序列表
文字前面机上\*或者\-来表示无须列表

\* item 

\* item

\* item 

效果如下:

* item 
* item
* item 


###有序列表
文字前面加上数字表示有序列表

\1.item1

\2.item2

效果如下:

1. item1
2. item2

参考
======

[*markdown basic*](https://help.github.com/articles/markdown-basics/)

[*markdown master*](https://guides.github.com/features/mastering-markdown/)

[markdown 入门2](http://www.jianshu.com/p/q81RER)

[**markdown 入门精选**](http://ibruce.info/2013/11/26/markdown/)

[**markdown 简体中文教程**](http://wowubuntu.com/markdown/)

[***markdown 编辑器 mou***](http://25.io/mou/)

[jekyll 入门](http://trefoil.github.io/2013/10/05/jekyll.html)

*{{ page.date | date_to_string }}*
