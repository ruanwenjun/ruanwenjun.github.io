---
layout: post
title: Tomcat容器初步学习
tags: [Tomcat]
---


Tomcat是一个web服务器，主要由连接器Connector和容器Container组成。

Tomcat中的容器都实现自Container接口,Container主要包含一系列的get,set,add,remove方法。
重点有获得父容器、子容器、和获得pipeline的方法。同时，Container继承了Lifecycle接口，提供对容器生命周期的管理。

![image](https://ruanwenjun.github.io/images/2019-02-15/container.png)
- Engine：表示整个Servlet引擎,可以有多个host
- Host：表示虚拟主机，可以有多个Context
- Context：表示一个web应用，可以有多个Wrapper
- Wrapper：表示一个独立的Servlet

他们的父子关系从上往下，Wrapper没有子容器。
每个容器的实现类在core包下可以找到,命名为StandardEngine、StandardHost、StandardContext、StandardWrapper
同时每个容器还有pipeline实例。pipeline内含Value链。pipeline类似filterChain，Value类似filter有一个next。
通过调用pipeline的Value的invoke来实现一些具体的操作。

例如：StandardWrapper的pipeline中有StandardWrapperValue,在StandardWrapperValue的invoke方法中包含了调用filter的doFilter方法，和servlet的service方法。



