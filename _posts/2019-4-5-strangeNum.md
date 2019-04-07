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

~~至于为什么要让0.0和-0.0的hashCode不一样，现在还不知道~~。这点在Float中也存在这种情况。

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
2019-4-7补充：关于equals方法和hashCode方法梳理

通常有个约定：当你重写equals方法之后，必须要重写hashCode方法。

equals方法是什么：equals方法是Object的一个方法,主要用来实现判断两个对象是否逻辑上”相等”.

默认实现是:判断两个对象是否是同一个对象
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

hashCode方法是什么：hashCode方法也是Object的一个方法，主要用于一些hash集合当中，例如HashMap,HashSet等.

默认实现是:
```java
public native int hashCode();
```
 
通常我们重写了equals方法的时候，需要重写hashCode方法，使得hashCode方法与equals方法保持一致。

主要是为了保证：
- if(a.equals(b) == true) then a.hashCode() == b.hashCode()。
- if(a.equals(b) == false) then a.hashCode() != b.hashCode()。

如果违反上述两条会出现以下两个问题

- 当两个对象equals一致的时候，如果他俩hashCode不一致，那么把这两个对象放到hash桶里，他们极大可能就不在一个桶（当然有可能会映射到同一个桶），当他们不在同一个桶的时候，那么这个时候就有问题了，比如：Set,这个时候就有个两个逻辑上相等的对象，就不符合Set的定义
- 当两个对象equals不一致的时候，他俩的hashCode方法如果一致了，那么这个时候也会有一点小问题，就是他俩必然会打到同一个桶了，这样不就增加了冲突了吗
所以，hashCode需要配合equals来实现，如果equals返回true，那么hashCode<a>必须</a>要返回一致，如果equals返回false,hashCode<a>最好</a>也要返回false。

所以0.0 和-0.0这两个equals返回false了，他们的hashCode自然也要返回不一致。

