---
layout: post
title: HDFS写入文件是防止已存在
tags: [HDFS]
---

往HDFS写入文件的时候需要判断是否已经存在该文件，如果存在则需要删除再写，否则会出错

## 解决方案一：代码层面

```

val conf = new Configuration()
val HDFSFileSystem = FileSystem.get(conf)
// 输出路径
val filePath = "haha/filePath"
// 判断HDFS中是否存在该目录，如果存在则删除
if (HDFSFileSystem.exists(new Path(filePath))) {
  // 第二个参数代表循环删除
  HDFSFileSystem.delete(new Path(filePath), true)
}

```

## 解决方案二：脚本层面

直接利用Hadoop fs -rm -r -f haha/filePath 命令

在运行脚本前面加上该命令即可
