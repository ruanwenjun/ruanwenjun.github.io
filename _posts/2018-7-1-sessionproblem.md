---
layout: post
title: 分布式Session问题
tags: [架构]
---

目录：

* [SessionSticky](#sessionsticky)
* [SessionReplication](#sessionreplication)
* [Session数据集中存储](#session数据集中存储)
* [CookieBased](#cookiebased)

Http协议本身是无状态的，需要基于Http协议支持会话状态的机制，而Session就是一种存在于服务器端的会话。但是当应用服务器由一台变为多台时，就会碰到Session问题。用户请求需要知道它的Session保存在哪个服务器。

例如：用户第一次访问服务器A，那么他的会话（例如登陆信息）就保存在服务器A上，那么它下次如果不能保证访问的是服务器A，那么就会出现需要重新登陆的情况。

![image](https://ruanwenjun.github.io/images/structure/sessionproblem.png)


解决Session问题的方案主要有如下四种：

## SessionSticky

![image](https://ruanwenjun.github.io/images/structure/sessionsticky.png)

通过负载均衡器实现，让对某一个Session的请求每次都发送到某一台服务器

## SessionReplication

![image](https://ruanwenjun.github.io/images/structure/sessionreplication.png)

不要求负载均衡器保证同一个会话的多次请求都发送到同一个Web服务器。而是在Web服务器之间加入会话数据的同步，通过同步来保证服务器之间的Session数据一致。

## Session数据集中存储

![image](https://ruanwenjun.github.io/images/structure/sessionstore.png)

把Session集中存储起来，而不是存储在服务器上，然后Web服务器从同样的地方来获取Session。

## CookieBased

![image](https://ruanwenjun.github.io/images/structure/cookiecontainsession.png)

通过Cookie来传递Session，即将Session保存在本地Cookie中。然后在Web服务器上通过获取Cookie来生成Session数据。
