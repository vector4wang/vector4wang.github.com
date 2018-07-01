---
title: Spark与Apache-Parquet
date: 2017-04-24 22:31:16
categories: 技术
tags:
  - Spark
  - Parquet
  - 大数据
---

> 七十年代时，有一长辈连练铁砂掌，功夫成了之后，可以掌断五砖，凌空碎砖，威风得不得了。时至八十年代，只能掌断三砖。到九十年代只能一砖一砖的断了。他说，一直以为功力退步了，后来才知道烧砖的配方改了。

<!--more-->


![数据压缩](http://upload-images.jianshu.io/upload_images/3167229-0ca5fe91bd99929e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 前言
　　前两篇将了spark的部署和一些简单的实例[Spark初体验(步骤超详细)](http://www.jianshu.com/p/52d0f043cc04)和[Spark再体验之springboot整合spark](http://www.jianshu.com/p/69d4547688c9)。我相信前两篇会对刚入门的sparker来说会有一些启发。
　　今天在使用spark去加载200万条数据的时候，服务器提示**内存分配不足**(服务器配置较低)，这里我就在想有没有什么方法将数据压缩压缩再压缩呢？网上查资料，问他人，最后看到并使用了`Apache Parquet`[官网](http://parquet.apache.org/)，这里简单的介绍一下parquet

### Parquet

>　　Parquet是面向分析型业务的**列式存储**格式,由Twitter和Cloudera合作开发，2015年5月从Apache的孵化器里毕业成为Apache顶级项目.
　　当时Twitter的日增数据量达到压缩之后的100TB+，存储在HDFS上，工程师会使用多种计算框架（例如MapReduce, Hive, Pig等）对这些数据做分析和挖掘；日志结构是复杂的嵌套数据类型，例如一个典型的日志的schema有87列，嵌套了7层。所以需要设计一种列式存储格式，既能支持关系型数据（简单数据类型），又能支持复杂的嵌套类型的数据，同时能够适配多种数据处理框架。

通过阅读[这篇文章](https://www.ibm.com/developerworks/cn/analytics/blog/5-reasons-to-choose-parquet-for-spark-sql/index.html)，相信你会对parquet的优点有所了解。
网上关于它的介绍和算法的讲解一大推，简单的讲使用parquet存储数据有以下优点：

- 压缩数据
- 不失真
- 减少IO吞吐量
- 高效的查询

更重要的是spark和parquet简直是绝配，犹如小葱拌豆腐、京酱肉丝陪面皮、手擀面配大蒜。。。

### 示例

老规矩，先上pom
```xml
...
<parquet.version>1.7.0</parquet.version>
...
<dependency>
            <groupId>org.apache.parquet</groupId>
            <artifactId>parquet-common</artifactId>
            <version>${parquet.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.parquet</groupId>
            <artifactId>parquet-encoding</artifactId>
            <version>${parquet.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.parquet</groupId>
            <artifactId>parquet-column</artifactId>
            <version>${parquet.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.parquet</groupId>
            <artifactId>parquet-hadoop</artifactId>
            <version>${parquet.version}</version>
        </dependency>
```

下面使用spark对mysql的一张表做parquet的读写操作
将整张表的数据存储为parquet格式的文件
```java
static String url = "jdbc:mysql://localhost:3306/world?" +
            "useUnicode=true&characterEncoding=UTF-8" +
            "&zeroDateTimeBehavior=convertToNull";
static String table = "jd_trainingdata";
static String username = "root";
static String passwd = "root";

private static void parquetWrite() {
        SparkConf conf = new SparkConf()
                .setAppName("test")
                .setMaster("local")
                .set("spark.executor.memory", "8g")
                .set("spark.rdd.compress true","true")
                .set("spark.testing.memory", "2147480000");
        JavaSparkContext sc = new JavaSparkContext(conf);
        SQLContext sqlContext = new SQLContext(sc);
        sqlContext.setConf("spark.sql.dialect", "sql");
        Properties connectionProperties = new Properties();
        connectionProperties.put("driver","com.mysql.jdbc.Driver");
        connectionProperties.put("user", username);
        connectionProperties.put("password", passwd);
        DataFrame vectorTable = sqlContext.read().jdbc(url, table,connectionProperties);
        vectorTable.saveAsParquetFile("jdData");// 在项目里生成文件，当然你也可以写绝对路径
    }
```
结果为
![parquet包含的文件](http://upload-images.jianshu.io/upload_images/3167229-7550885467f50a4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
关于内容我没有去深究（官网有详细的解释说明），但是能看得出文件是成功的生成了~
那么接下来看看如何去读parquet的文件内容
```java
private static void parquetRead() {
        SparkConf conf = new SparkConf()
                .setAppName("test")
                .setMaster("local[4]")
                .set("spark.executor.memory", "8g")
                .set("spark.rdd.compress true","true")
                .set("spark.testing.memory", "2147480000");
        JavaSparkContext sc = new JavaSparkContext(conf);
        SQLContext sqlContext = new SQLContext(sc);
        sqlContext.parquetFile("jdData").registerTempTable("jdData");
        DataFrame dbaClos = sqlContext.sql("select * from jdData where Title = 'DBA' and Title <> '' ");
        dbaClos.show();
    }
```
运行日志
![日志](http://upload-images.jianshu.io/upload_images/3167229-d37fea96d79e5e91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
当你拿到`sqlContext`的时候，你就可以使用spark提供的sql去查找你想要的数据，要提醒一下， spark支持的sql跟mysql，oracle的不太一样，详细参考[官网帮助文档](http://spark.apache.org/docs/latest/sql-programming-guide.html#supported-hive-features)

### 后续
　　这里说一下我为什么使用parquet，我现在测试服务器的内存是16g，项目在启动的时候需要加载其他的模型、数据等文件，而且数据虽然才有220多万条，但是存储的内容比较多，所以就会导致内存不足，使用了parquet之后，数据大小直接被压缩为原来的三分之一！！！而且spark对parquet文件的支持近乎完美，所以使用parquet之后，我完全可以不用考虑内存分配不足的问题了。
