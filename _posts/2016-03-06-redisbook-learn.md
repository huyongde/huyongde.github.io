---
layout: post
title: 学习redisbook, 了解redis设计和原理
tags: redis 
---

###redisbook介绍

> redisbook是huangjianhong大神写的，介绍redis设计和实现的一本，redis入门提高很好的一本书。

###相关学习资源

* [带注释的redis源码](https://github.com/huangz1990/redis-3.0-annotated)
* [redisbook网站](redisbook.com)
* [redis pdf 版本下载](http://pan.baidu.com/s/1jGXhgvs)  pdf版本已经大概看了一遍，确实对redis设计以及底层实现有了一些了解，**应对面试足够了。**

###redisbook学习心得

##reis内部数据结构

####0.简单动态字符串(simple dynamic string, sds)
##### redis sds 的实现如下：

{% highlight c++ linenos %}
struct sdshdr {
    int len; // buf已用长度
    int free; // buf可用的长度
    char buf[]; //实际存储字符串数据的地方
}
{% endhighlight %}

#####基于如上sds的实现，redis字符串有如下特征：

* redis的字符串是使用sds来表示的，而不是c字符串
* 和 c字符串比较，sds有以下特性：
    * 高效地获得字符串长度,O(1)
    * 高效的执行追加操作
    * 二进制安全
* sds为追加操作进行追加优化，加快追加操作的速度，降低内存分配的次数，代价是多占用一些内存，而且这些内存不会主动释放
* redis sds 的实现  


#### 1.双端列表

##### 双端链表的实现

{% highlight c++ linenos %}
//双端链表的节点
typedef struct listNode {
    //前驱节点
    struct listNode *prev;
    //后驱节点
    struct listNode *next;
    //值
    void *value; 
}listNode;

//双端链表的定义

typedef struct list {
    //表头指针
    listNode *head;
    //表尾指针
    listNode *tail;
    unsigned long len; // 节点数量
    //复制函数
    void *(*dup) (void *ptr);
    //释放函数
    void (*free) (void * ptr);
    //比较函数
    int (*match) (void *ptr, void *key);
} list; 
       
{% endhighlight %}

#####基于如上双端列表的实现，双端链表有如下特征：
* 链表带有前后驱的节点的指针，访问前后驱节点的负责度为O(1),并且链表可以进行两个方向上的迭代。
* 链表带有前后驱节点的指针，在头或者尾进行增加或者删除节点的复杂度为O(1).
* 链表带有记录链表长度的字段，可以O(1)获得链表的长度

### 2.字典
> 字典实现有多种方式，元素个数不多时，可以通过链表和数组来实现；元素个数达到一定数量级后可以考虑哈希表;还有一种更为复杂的平衡树实现。

#####字典的hash表实现


{% highlight c++ linenos %}
//每个字典有两个hash表，来实现渐进式rehash
typedef struct dict {
    //特定类型的处理函数
    dictType *type;
    //类型处理函数的私有数据
    void *privdata;
    //哈希表2个
    dictht ht[2];
    // 记录rehash进度
    int rehashidx;
    //迭代器的数量
    int iterators;
}dict;

//哈希表的实现
typedef struct dictht {
    //哈希表节点指针的数组
    dictEntry **table;
    //指针数组的大小
    unsigned long size;
    //指针数组的长度掩码,用来计算索引值
    unsigned long sizemark;
    //哈希表现有节点数量
    unsigned long used;
}dictht;
//table元素是个数组，数组中每个元素指向dictEntry结构体的指针。

//hash表节点的结构体
type struct dictEntry {
    //键
    void *key;
    //值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;    
    }v;
    //后继节点
    dictEntry *next;
}dictEntry;
//next属性指向另一个dictEntry节点，dictht是通过链地址来解决hash碰撞。
//当不同的键有相同的hash值时，dictht通过一个链表来链接起来这些key-value对的dictEntry。
{% endhighlight %}

####redis字典的实现可以用下图表示

![redis-dict](/image/redis-dict.png)


未完下周继续
