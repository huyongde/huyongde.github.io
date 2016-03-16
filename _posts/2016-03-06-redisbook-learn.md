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

###目录
* 内部数据结构
    * sds
    * 双端链表
    * 字典
    * 跳跃表
* 内存数据结构
    * intset，整数集合
    * 压缩列表 
* Redis数据类型
    * 字符串
    * 哈希表
    * 列表
    * 集合
    * 有序集合

## redisbook学习心得
>逼迫自己静下心来做自己抵触的事情，一定会有收获的。 


## redis内部数据结构

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
typedef struct dictEntry {
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

#### 字典的特征
1. 字典是由键值对构成的抽象的数据类型
2. Redis 中的数据库和哈希键都基于字典来实现
3. Redis 字典的底层实现为哈希表，每个字典有两个hash表，一般情况下只使用0号哈希表，只有在rehash进行时，才会同时使用0号和1号哈希表。
4. 哈希表是使用链地址的方式来解决键冲突的问题
5. Rehash 可以用于扩展或收缩哈希表
6. 对哈希表进行rehash，是分多次，渐进式完成的


### 3.跳跃表

#### 3.0 参考
    * [跳跃表](http://blog.sina.com.cn/s/blog_60707c0f0100wudj.html)

#### 3.1 跳跃表中查找元素时间复杂度O(logN):

在跳跃表中查找一个元素x，按照如下几个步骤进行：
> 从最上层的链的开头开始假设当前位置为p，它向右指向的节点为q（p与q不一定相邻），且q的值为y。
> 将y与x作比较: 如果x=y，输出查询成功，输出相关信息；
> 如果x大于y，从p向右移动到q的位置；
> 如果x小于y，从p向下移动一格， 如果当前位置在最底层的链S0中，且还要往下移动的话，则输出查询失败。 

#### 3.2跳跃表的结构定义

如下zskiplist和zskiplistNode定义都在redis.h头文件中.

{% highlight c linenos%}
typedef zskiplist struct {
    zskiplistNode *header, *tail;
    unsigned long length;
    int level;
}zskiplist;
typedef zskiplistNode struct {
    robj *robj;
    double score;
    struct zskiplistNode *backward;
    struct zkiplistlevel {
        struct zskiplistNode *forward;
        unsigned span; //跨越节点的数量
    } level[];
} zskiplistNode;
{% endhighlight %}

> 跳跃表中各节点是按照value来有序存储的，所以跳跃表的删除、插入、查找一个元素的时间复杂度都是O(logN).


## 内存数据结构
* 虽然内部数据结构足够强大，但是创建一套完整的数据结构本身就是一套非常费内存的工作， 当一个对象包含的元素并不多,或者元素体积并不大时，使用代价高昂的内部数据结构并不是最好的办法
* 为了解决如上问题，redis在特定条件下会使用内存数据结构。
* 内存映射数据结构是一系列特殊编码的字节序列，创建他们所需要的内存通常比左右类似的内部数据结构要少的多，使用得当可以节省大量内存。
* 但是内存数据结构的编码方式比内部数据结构的要复杂，所以内存数据结构所占用的CPU 时间会比作用类似的内部数据结构要多。

Redis中内存映射数据结构有两种1.intset, 2.压缩列表

### 1. 整数集合
> 整数集合用来保存有序、无重复的多个整数值，它会根据元素的值，自动想选择该用什么长度的整形类型来保存数据,是用最长类型元素的类型来保存所有的元素，
> 新元素的加入可能会改变整数集合的编码类型，可能要变成更大空间的类型来存储所有的元素

#####整数集合的数据结构

定义在`intset.h`中可以找到，详细如下：

{% highlight c linenos %}
typedef struct intset {
    uint32_t encoding; //保存元素所使用的类型的长度
    uint32_t length; //整数集合中元素个数
    int8_t contents[]; //保存元素的数组
}intset;
{% endhighlight %}

`encoding`的值可以是如下三种的一种：
{% highlight c linenos %}
\#define INTSET_ENC_INT16 (sizeof(int16_t))
\#define INTSET_ENC_INT32 (sizeof(int32_t))
\#define INTSET_ENC_INT64 (sizeof(int64_t))
{% endhighlight %}



`contents` 是实际存放元素的地方，数组中的元素有如下两个特性：
 
* 没有重复元素
* 元素在数组中从小到大排序。

#####intset 特征总结

* intset 用来存储有序、不重复的整形数据，它会根据元素的值，选择该用什么长度的整数类型来保存元素
* 当一个位长度更长的整数值添加到intset时，需要对intset进行升级,升级后的intset中的元素的位长度都等于新加元素的位长度,但元素值保持不变。
* 升级涉及对每个元素进行内存重分配和移动，时间复杂度是O(N)
* intset是有序的，使用二分法来查找元素，时间复杂度O(logN)
###2. 压缩列表
> ziplist是一系列特殊编码的内存块构成的列表，一个ziplist可以包括多个节点entry，每个节点可以保存一个长度受限制的 字符数据或者整数.

####ziplist的结构

![ziplist](/image/ziplist.png)

下面分别解释途中每个字段的含义：

* zlbytes: uint32_t 表示整个ziplist的字节数，用来重新分配内存或者计算末端使用
* zltail: uint32_t 到达整个ziplist表尾的偏移量，通过这个偏移量可以在不遍历整个列表的前提下，获得表尾节点
* zllen: uint16_t ziplist中节点(entry)的数量
* entryX : 节点，entry的结构见下图
* zlend: uint8_t 用来标记ziplist末端

![ziplistentry](/image/ziplistentry.png)

下面介绍些ziplist节点每个字段的含义：

* pre_entry_length: 上一个节点的长度，通过这个值，可以进行指针计算,从而跳到上一个节点
* encoding && length: encoding 和length共同决定了content中保存数据的数据类型以及长度。
* content: 是节点的实际数据内容

###ziplist总结
> 添加或者删除ziplist节点,可能会引起连锁更新，最坏时间复杂度是O(N^2), 不过连锁更新的概率不高，所以一般时间复杂度是O(N).


## Redis 数据类型







未完待续
