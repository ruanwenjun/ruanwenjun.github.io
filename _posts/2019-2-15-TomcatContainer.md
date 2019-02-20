---
layout: post
title: Tomcat初步学习
tags: [Tomcat]
---
目录：
- [Tomcat容器](#tomcat容器)
- [Tomcat启动](#tomcat启动)
- [思考](#思考)

Tomcat是一个web服务器，主要由连接器Connector和容器Container组成。

## Tomcat容器
Tomcat中的容器都实现自Container接口,Container主要包含一系列的get,set,add,remove方法。重点有获得父容器、子容器、和获得pipeline的方法。同时，
Container继承了Lifecycle接口，提供对容器生命周期的管理和统一的启动关闭。

![image](https://ruanwenjun.github.io/images/2019-02-15/container.png)
- Engine：表示整个Servlet引擎,可以有多个host
- Host：表示虚拟主机，可以有多个Context
- Context：表示一个web应用，可以有多个Wrapper
- Wrapper：表示一个独立的Servlet

他们的父子关系从上往下，Wrapper没有子容器。Engine没有父容器。
每个容器的基本实现类在core包下可以找到,命名为StandardEngine、StandardHost、StandardContext、StandardWrapper
同时每个容器还有pipeline实例。pipeline内含Value链。pipeline类似filterChain，Value类似filter，Value有一个next。
可以通过调用pipeline的Value的invoke来实现一些具体的操作。

例如：StandardWrapper的pipeline中有StandardWrapperValue,在StandardWrapperValue的invoke方法中会构造Servlet，和filterChain，
调用filterChain的doFilter方法，在doFilter方法中调用Servlet的service方法。

## Tomcat启动

Tomcat启动是通过BootStrap的main方法启动的。BootStrap会加载Catalina，通过反射调用Catalina的启动方法。

Catalina会构造Digester对象，加载配置文件,然后通过Server对象启动，Server主要用于提供容器start，close方法。

Server对象中包含Service数组,service启动会调用Engine启动,调用mapperListener启动，调用Connector启动。
Service用于连接Context和Connector,一个Context可以有多个Connector，一个Connector只能属于一个Context.

## 思考
在Tomcat中经常可以看到copy on write模式的使用，就是在多个线程操作数组的时候，当对数组修改的时候，复制一份，修改复制的那一份，修改完了在将修改的数组引用赋值给原数组。

这点在CopyOnWriteArrayList就看到过，但是CopyOnWriteArrayList中的数组使用Volatile修饰了，在Tomcat中Copy on write模式中就没有用volatile修改数组，

例如StandardService中就没有使用volatile修饰connectors数组，这里会不会存在问题，volatile是为了保证可见性，即读取的时候直接从内存中获取，而不从缓存中获取。
这里虽然使用加锁可以让修改之后的数组立刻刷新回主存，但是其他线程读取的时候可能会从处理器缓存中读取，即读到过去的历史数据。

TODO:但是作为一个顶级开源项目应该不会出现这种问题，应该还有我没有考虑到的地方。

