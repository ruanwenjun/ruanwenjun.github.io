---
layout: post
title: Kafka源码Idea启动
tags: [kafka]
---

最近准备深入学习一下Kafka，于是开始卷源码。

虽然已经给Kafka提交了几个简单的Minor PR，但是说来惭愧，至今还没有在本地Debug过源码，于是今天打算跑一下。

由于Kafka在最新的3.x（我不确定是什么版本）引入了Kraft模块，所以未来Zookeeper肯定是会慢慢被抛弃的，所以也是以最新的版本为主。

（此处省略clone代码，下载IDEA，安装JDK，安装scala等一系列步骤...）

当我们在IDEA中成功导入Kafka项目之后，通过查看kafka的启动脚本`bin/kafka-server-start.sh`可以发现Kafka的启动类是`kafka.Kafka`，这是一个由scala写的object，里面自带一个main函数，直接启动会发现报错，提示我们需要传入properties文件路径。

可以通过在args中添加`./config/kraft/server.properties`来传入properties文件路径，重新启动发现还是会报错，提示没有clusterId之类的（忘了）。

通过查看config/kafka/README.md可以发现，其实我们如果要使用kraft方式启动话，需要初始化集群。

按照提示使用` sh bin/kafka-storage.sh format -t 8HWDlq07Rp2FZQP2TOGWkg -c config/kraft/server.properties`初始化之后会在`/tmp/kraft-combined-logs`目录下看到我们的rafk元数据目录。

初始化之后再次启动`kafka.Kafka`, 会发现启动成功，但是此时控制台没有日志打印，原因是core模块是没有包含log4j模块，至于为什么可能需要以后深入了解之后才会明白，总而言之，在build.gradle中添加
```java
implementation libs.log4j
implementation libs.slf4jlog4j

```
同时设置环境变量`-Dkafka.logs.dir=logs -Dlog4j.configuration=file:/Users/ruanwenjun/Project/Github/kafka/config/log4j.properties`
即可。

至此就可以愉快的debug代码了。


