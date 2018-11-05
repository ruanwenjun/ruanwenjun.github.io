---
layout: post
title: HashMap中的Key能否为null
tags: [jdk]
---
目录：
- [HashMap的Key能否为null](#hashmap的key能否为null)
- [HashMap中value能否为null](#hashmap中value能否为null)
- [HashTable中的Key和Value能否为null](#hashtable中的key和value能否为null)
- [ConcurrentHashMap里的Key和Value能否为null](#concurrenthashmap里的key和value能否为null)
- [总结](#总结)


HashMap是用来存储键值对的一种数据结构，每个Key对应一个Value，那么HashMap的Key能否为null呢？

## HashMap的Key能否为null

直接上代码

```java
import java.util.HashMap;

/**
 * @Author RUANWENJUN
 * @Creat 2018-11-05 21:52
 */

public class HashDemo {
    public static void main(String[] args) {
        HashMap<Integer,Integer> map = new HashMap<>();
        map.put(null,1);
        Integer integer = map.get(null);
        System.out.println(integer);
    }
}

输出：1
```
根据结果看，HashMap的Key可以为Null，而根据HashMap中Key的唯一性（不知道这么说准不准确），可以得出HashMap中可以存在唯一的null key。

至于为什么，通过观察源码可以发现，
```java
 public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
}


static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

```
HashMap里面定义了一个hash（key）方法，当Key为null时，Hash值为0。

## HashMap中value能否为null
依然直接上代码

```java
import java.util.HashMap;

/**
 * @Author RUANWENJUN
 * @Creat 2018-11-05 21:52
 */

public class HashDemo {
    public static void main(String[] args) {
        HashMap<Integer,Integer> map = new HashMap<>();
        map.put(null,null);
        map.put(1,null);
        Integer integer = map.get(null);
        Integer integer1 = map.get(1);
        System.out.println(integer);
        System.out.println(integer1);
    }
}

输出：null
      null
```

根据结果可知，HashMap里的value可以为null，这也是早期HashMap设计不合理的一个地方，因为当key不存在的时候也会返回null（在JAVA8之后添加了getOrDefault方法）。

## HashTable中的Key和Value能否为null

上代码
```java
import java.util.HashMap;
import java.util.Hashtable;

/**
 * @Author RUANWENJUN
 * @Creat 2018-11-05 21:52
 */

public class HashDemo {
    public static void main(String[] args) {
        Hashtable<Integer,Integer> table = new Hashtable<>();
        table.put(null,1);

    }
}

输出：
Exception in thread "main" java.lang.NullPointerException
	at java.util.Hashtable.put(Hashtable.java:465)
	at HashDemo.main(HashDemo.java:12)

```
结构表明，HashTable中的Key不能为null，查看put源码如下：
```java
public synchronized V put(K key, V value) {
    // Make sure the value is not null
    if (value == null) {
        throw new NullPointerException();
    }

    // Makes sure the key is not already in the hashtable.
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }

    addEntry(hash, key, value, index);
    return null;
}
```
发现，HashTable的put方法与HashMap的put方法不同，HashTable是直接使用key的hashCode方法，那么当Key为null时，就会抛空指针异常，同时也可以发现当value为null时，会手动抛一个空指针异常。

## ConcurrentHashMap里的Key和Value能否为null

上代码
```java
import java.util.concurrent.ConcurrentHashMap;

/**
 * @Author RUANWENJUN
 * @Creat 2018-11-05 21:52
 */

public class HashDemo {
    public static void main(String[] args) {
        ConcurrentHashMap<Integer,Integer> concurrentHashMap = new ConcurrentHashMap<>();
        concurrentHashMap.put(null,1);
    }
}

输出：
Exception in thread "main" java.lang.NullPointerException
	at java.util.concurrent.ConcurrentHashMap.putVal(ConcurrentHashMap.java:1011)
	at java.util.concurrent.ConcurrentHashMap.put(ConcurrentHashMap.java:1006)
	at HashDemo.main(HashDemo.java:13)

```
结果表明，ConcurrentHashMap的Key并不能为null,查看put方法源码：
```java
public V put(K key, V value) {
    return putVal(key, value, false);
}

final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```
通过源码发现，当key或Value为null时，会手动抛一个空指针异常。

## 总结

- HashMap的Key和Value都可以为null
- HashTable的Key和Value都不能为null
- ConcurrentHashMap的Key和Value都不能为null

至于为什么这么设计，还有待日后挖掘
