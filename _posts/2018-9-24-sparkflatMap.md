---
layout: post
title: 关于SparkRdd从1对多映射转为1对1映射
tags: [Spark]
---

flatMap跟Map的使用上的区别是，FlatMap会将Map之后的结果放在同一个集合中


例如：有一个这种需求
一个RDD里面保存着"小明,戏剧|爱情|动画"，这种格式的数据，每个人后面对应他喜欢的电影频道，需要统计或者得出每个电影频道下面的人，那么就需要先得出每个人跟他喜欢的频道的映射，这时就可以用flatMap来解决
```
package com.wenjun

import org.apache.spark.{SparkConf, SparkContext}

object FlatMapDemo {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("MySpark").setMaster("local")
    val sc = new SparkContext(conf)
    val list = new scala.collection.mutable.ListBuffer[String]()
    list += "小明,戏剧|爱情|动画"
    list += "小红,戏剧|爱情|动画"
    list += "小智,戏剧|爱情|动画|魔法|综艺"
    val rdd = sc.parallelize(list)
    val interestRdd = rdd.map(man => {
      val split = man.split(",")
      (split(0), split(1).split("\\|"))
    })
    val resRdd = interestRdd.flatMap(tuple => {
      for (i <- tuple._2) yield (i, tuple._1)
    }).groupByKey()
    resRdd.foreach(println)
  }

}
输出：
(爱情,CompactBuffer(小明, 小红, 小智))
(魔法,CompactBuffer(小智))
(综艺,CompactBuffer(小智))
(动画,CompactBuffer(小明, 小红, 小智))
(戏剧,CompactBuffer(小明, 小红, 小智))
```