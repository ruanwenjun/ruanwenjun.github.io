---
layout: post
title: 单例模式
tags: [大话设计模式读书笔记]
---
目录
* [问题引出](#问题引出)
* [初步解决方案](#初步解决方案)
* [多线程访问问题](#多线程访问问题)
* [解决方案](#解决方案)

# 问题引出
在有的时候，我们希望程序中的某一个类只有一个实例对象，例如DOTA当中的一个英雄，我们就不希望在程序里面同时出现多个。那么怎么才能做到这一点呢？
# 初步解决方案
可以使用一个私有静态成员变量+公有的静态方法来实现，同时使构造器私有（看下面的代码就一目了然）

例如：我们来实现SF英雄的单例

```java
//影魔英雄
public class SF{
    private static SF sf;
    //将构造器定义为私有保证外部不能实例化该类
    private SF(){}
    //获得影魔的方法
    public static SF getSF( ){
        if(sf == null){
            sf = new SF();
        }
        return sf;
    }
}
```
这样我们就只能通过SF.getSF()方法来得到影魔，但是这其实存在问题。
# 多线程访问问题
如果有多个线程同时调用SF.getSF()方法，其实是有可能得到多个SF对象的。

例如线程A,线程B同时调用SF.getSF（）

线程A : sf = null ;

线程B : sf = null ;

线程A ： sf = new SF();

线程B ： sf = new SF();

于是游戏就出现了BUG
# 解决方案
通常有三种方法解决多线程同时访问的单例问题
- 在加载类的时候就初始化

```java
public class SF{
    public static SF sf = new SF();
    //将构造器定义为私有保证外部不能实例化该类
    private SF(){}

}
```
这样就可以直接通过SF.sf来得到影魔对象

**缺点**：在类加载的时候就初始化会有一点浪费资源，如果影魔对象是在很久之后才被使用
- 将获得单例的方法上锁

```java
public class SF{
    private static SF sf;
    //将构造器定义为私有保证外部不能实例化该类
    private SF(){}
    //获得影魔的方法
    public static synchronized SF getSF( ){
        if(sf == null){
            sf = new SF();
        }
        return sf;
    }
}
```
这样当一个线程调用getSF方法的时候，其他线程就需要等待

**缺点**：如果getSF方法被频繁使用的话，会极大的影响性能（毕竟是加了个锁）
- 采用双重检查加锁

```java
public class SF{
    private static SF sf;
    //将构造器定义为私有保证外部不能实例化该类
    private SF(){}
    //获得影魔的方法
    public static SF getSF( ){
        if(sf == null){
            synchronized(SF.class){
                if(sf == null){
                    sf = new SF();
                }
            }
        }
        return sf;
    }
}
```
这是我觉得比较优秀的方案。

**缺点**：需要JAVA5以上

---
另外需要注意的是如果使用多个类加载器，可能导致单例失效而产生多个实例。

个人猜想：是由于静态成员变量在一个类加载器中只存在一个，如果是多个类加载器加载类，那么每个类加载器中都会有一个静态变量，此时需要自行指定类加载器，并使用同一个类加载器
