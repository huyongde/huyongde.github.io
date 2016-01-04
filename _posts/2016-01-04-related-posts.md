---
layout: post
title: 显示post的相关文章(有相同的tag) related_posts
tags: blog jekyll related_posts
---

##参考

* [jekyll related_posts](http://zhangwenli.com/blog/2014/07/15/jekyll-related-posts-without-plugin/)


##不用插件,显示文章相类似文章

直接把显示相似文章的代码放到了_layout/post.html中了，为每个post显示相关的posts。

代码如下

{% highlight html %}
{% raw %}
<div style="float:left;">
    {% assign hasSimilar = '' %}
{% for post in site.related_posts %}
    {% assign postHasSimilar = false %}
    {% for tag in post.tags %}
        {% for thisTag in page.tags %}
            {% if postHasSimilar == false and hasSimilar.size < 5 and post != page and tag == thisTag %}
                {% if hasSimilar.size == 0 %}
                <h2>Similar Posts</h2>
                <ul>
                {% endif %}
                    <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>

                    <h5> <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a> </h5>
                {% capture hasSimilar %}{{ hasSimilar }}*{% endcapture %}
                {% assign postHasSimilar = true %}
            {% endif %}
        {% endfor %}
    {% endfor %}
{% endfor %}
{% if hasSimilar.size > 0 %}
    </ul>
{% endif %}
</div>
{% endraw %}
{% endhighlight %}

我的blog就是左边显示相关的posts，右边显示最新更新的文章。

效果见本博客。