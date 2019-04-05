---
layout: post
title: 一次奇怪的刷题引发的惨案
tags: [jdk]
---

每当我觉得无聊的时候就喜欢刷一刷leetcode上的题目。

今天刷到[149.直线上最多的点数](https://leetcode-cn.com/problems/max-points-on-a-line/submissions/)这一题的时候一直有问题。

后来发现是在一个Map里面put key为0.0 和key为-0.0的时候这两个key居然不是同一个key。

```java
Map<Double, Integer> map = new HashMap<>();
map.put(0.0, 1);
map.put(-0.0, 2);
System.out.println(map);

输出{0.0=1, -0.0=2}
```
0.0 == -0.0这肯定是恒成立的。但是在map中就不是同一个key，想了一下，认为是因为0.0和-0.0在向上转型为Double的时候变成了两个对象。

于是猜测，0.0的hashCode和-0.0的hashCode是不一样。

验证:
```java
Double d = 0.0;
Double d2 = -0.0;
System.out.println(d.hashCode());
System.out.println(d2.hashCode());

输出：
0
-2147483648
```
果不其然，他俩的hashCode是不一样的，进一步，发现他俩的equals返回也是false,通过源码发现Double的equals和hashCode实现上都调用了同一个方法。

至于为什么要让0.0和-0.0的hashCode不一样，现在还不知道。这点在Float中也存在这种情况。

同时还发现，Double是没有缓存对象的。Integer默认是缓存-128到127之前的值。
```java
Double d1 = 0.0;
Double d2 = 0.0;
System.out.println(d1 == d2);
Integer i1 = 0;
Integer i2 = 0;
System.out.println(i1 == i2);
输出:
false
true
```
