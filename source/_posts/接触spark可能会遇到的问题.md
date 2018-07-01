---
title: 接触spark可能会遇到的问题
date: 2017-04-24 22:31:16
categories: 技术
tags:
  - Spark
  - question
---

>说明: 以下是我自己遇到的问题，可能会对你有所帮助，也可能对你一点帮助也没有

<!--more-->


### No configuration setting found for key 'akka.version'
spark的jar包不能通过`java -jar xxx.jar`来执行，需通过`spark-submit`来执行

### Caused by: java.lang.ClassNotFoundException:com.fasterxml.jackson.databind.type.ReferenceType
maven添加相关依赖
```xml
<dependency>
    <groupId>com.madgag</groupId>
    <artifactId>bfg</artifactId>
    <version>1.12.5</version>
</dependency>
```

### scala.runtime.BooleanRef.create(Z)Lscala/runtime/BooleanRef
scala版本号的问题，我的是因为项目有个dl4j的依赖里使用了scala的2.11版本，而我本地和测试环境上使用的是2.10版本，所以报错，将版本号统一即可

