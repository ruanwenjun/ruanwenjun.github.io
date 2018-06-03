---
layout: post
title: Redis字典
tags: [Redis]
---
> Redis没有使用C语言传统的字符串表示，而是自己构建了一种名为简单动态字符串(simple dynamic string, SDS)的抽象类型，并将SDS作为Redis的默认字符串表示。

目录：
* [字典的结构](#字典的结构)
* [解决键的冲突](#解决键的冲突)
* [rehash](#rehash)

> 字典是一种用于保存键值对的抽象数据结构。字典中的每个键都是独一无二的，程序根据键查找与之关联的值，或者通过键来进行值的更新，又或者通过键来删除整个键值对。
> Redis中的数据库就是通过字典实现的，还有哈希键的底层也是通过字典实现。


### 字典的结构
(1) 哈希表节点

哈希表节点使用dictEntry结构表示

```c
typedef struct dictEntry{

    // 键
    void *key;

    // 值    
    union{
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;

    // 指向下一个哈希表节点
    struct dictEntry *next;
}

```
![image](https://ruanwenjun.github.io/images/redis/dictEntry.png)


key 属性保存键值对中的键，v属性保存键值对中的值，值可以是指针，next属性是一个指向下一个哈希表节点的指针，通过这个指针属性可以将多个哈希表节点构成链表，用这种方法来解决冲突。即当对key使用某种hash算法得到的哈希值上有节点了，那么就会将当前的节点使用头插法加到该链表中。

(2) 哈希表

```c
typedef struct dictht{

    // 哈希表数组
    dictEntry **table;

    // 哈希表大小
    unsigned long size;

    // 哈希表掩码，用于计算索引值
    // 总等于size - 1
    unsigned long sizemask;

    // 该哈希表已有节点的数目
    unsigned long used;
} dictht;
```

![image](https://ruanwenjun.github.io/images/redis/dictht.png)

table 是一个数组，数组中每一个元素都是指向一个dictEntry结构的指针

(3) 字典

```c
typedef struct dict{

    // 类型特定函数
    dictType *type;

    // 私有数据
    void *privdata;

    // 哈希表
    dictht ht[2];

    // rehash索引
    // 当没有进行rehash时，索引为1
    int trehashidx;
} dict;
```

![image](https://ruanwenjun.github.io/images/redis/dict.png)

type属性和privdata属性是针对不同类型的键值对，为创造多态字典而设置的。

ht属性是一个包含两个哈希表的数组，数组中每一项都是一个哈希表，一般只用ht[0]这个哈希表，另一个ht[1]只有当进行rehash时才使用

### 解决键的冲突

采用的是链地址法，上面介绍哈希表节点的时候提到的。

### rehash
随着操作的执行，哈希表保存的键值对会逐渐增多或者减少，为了让哈希表的负载因子保持在一个合理的范围，当哈希表的键值对太多或者太少时，会对哈希表进行rehash

rehash步骤：
1. 为字典的ht[1]哈希表分配空间(ht[1]的大小为第一个大于等于ht[0].used的2^n)
2. 将保存在ht[0]中的所有键值对rehash到ht[1]上，rehash是指重新计算键的哈希和索引值，然后将键值对放在ht[1]哈希表的指定位置上
3. 当ht[0]上所有键值对都迁移到ht[1]上后，释放ht[0]，将ht[1]设为ht[0],并且在ht[1]新创建一个哈希表。

渐进式rehash，指rehash过程不是一步完成的，而是渐进式的完成的。


除此之外，Redis字典中的哈希算法采用的是[murmurhash算法](http://xinklabi.iteye.com/blog/2195092)
