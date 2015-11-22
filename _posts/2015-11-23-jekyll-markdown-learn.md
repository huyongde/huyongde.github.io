---
layout: default
title: jekyll 和 markdown 入门
---
<h1> {{ page.title }} </h1>

##段落
通过空行来形成不同的段落

##标题
```
># The largest heading (an <h1> tag)

>## The second largest heading (an <h2> tag)

>.....

>###### The 6th largest heading (an <h6> tag)
```

##引用(blockquotes)
>这里是个引用

##文字加粗&&斜体
*这里是加粗* (文字前后都加上*,可以加粗文字）
**这里是斜体** （文字前后都加2个*,可以是文字变成斜体）


#参考

[*markdown basic*](https://help.github.com/articles/markdown-basics/)

[*markdown master*](https://guides.github.com/features/mastering-markdown/)

[markdown 入门2](http://www.jianshu.com/p/q81RER)

[jekyll 入门](http://trefoil.github.io/2013/10/05/jekyll.html)

*{{ page.date | date_to_string }}*
