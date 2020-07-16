---
title: Hive上使用udf的问题
copyright: true
permalink: 1
top: 0
date: 2020-07-16 21:09:56
tags:
	- hive
	- udf
	- 技术
categories:
	- 技术
password:
---

#### 

这里记录一下自己用UDF遇到的问题，最后虽然解决了也知道是什么原因导致的，但是没有从代码或Hive层面去理解，全是靠自己意会出来的。



<!-- more -->



<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=558462973&auto=1&height=66"></iframe>



### 背景
一个业务需求，需要在hive上操作，逻辑比较复杂，写了两个udf，用的是一项目，对应的目录如下：
```
└─src
    ├─main
       └─java
			└─com
		      └─quick
              	└─udf
              	  └─fun1
              	  └─fun2
```
大致简化了一下，这样的话两个udf的class就是
```
fun1 ---> com.quick.udf.fun1
fun1 ---> com.quick.udf.fun2
```
打包后的jar为**fun_v1.0.jar**
最终的用法如下
```
DELETE jar hdfs:///tmp/udf/fun_v1.0.jar;
DROP TEMPORARY FUNCTION fun1;
ADD jar hdfs:///tmp/udf/fun_v1.0.jar;
CREATE TEMPORARY FUNCTION fun1 'com.quick.udf.fun1';

select fun1(1,2,3) from tablea;


DELETE jar hdfs:///tmp/udf/fun_v1.0.jar;
DROP TEMPORARY FUNCTION fun2;
ADD jar hdfs:///tmp/udf/fun_v1.0.jar;
CREATE TEMPORARY FUNCTION fun2 'com.quick.udf.fun2';

select fun2(1,2,3) from table b


select fun1(a,b,c) from tablec
```
我们公司有自己的Hive平台， udf函数申请的时候一定要一个函数名对应一个jar包和class路径，所以上面两个udf就申请了两次，使用的时候就要add两次。
这个上线后没有任何问题，最近有了新的需求，要对fun1进行调整，改动还比较大，最后的jar包版本为v2.0,用法如下
```
DELETE jar hdfs:///tmp/udf/fun_v2.0.jar;
DROP TEMPORARY FUNCTION fun1;
ADD jar hdfs:///tmp/udf/fun_v2.0.jar;
CREATE TEMPORARY FUNCTION fun1 'com.quick.udf.fun1';

select fun1(1,2,3,4,5) from tablea;


DELETE jar hdfs:///tmp/udf/fun_v1.0.jar;
DROP TEMPORARY FUNCTION fun2;
ADD jar hdfs:///tmp/udf/fun_v1.0.jar;
CREATE TEMPORARY FUNCTION fun2 'com.quick.udf.fun2';

select fun2(1,2,3) from table b



select fun1(a,b,c,d,e) from tablec
```
这里就用了两个jar包，虽然这两个中的fun2代码是一样的，看起来没问题，理论上来说也是没问题！

### 噩梦开始

开始测试执行的时候，就开始报错，报的错非常的奇怪
第一次报错
```
Exception in thread "main" org.apache.hive.com.esotericsoftware.kryo.KryoException: java.lang.UnsupportedOperationException
```
（具体的日志没有保存）
所有的焦点都被KryoException带走， 说是序列化的问题，然后就对自己的udf进行各种测试，发现并不会出现这个问题（期间也测出了一个小bug，业务上的）
当时找了自己测了半天，又找了半天问题，最终在这里找到了“解决方案”
https://issues.apache.org/jira/browse/HIVE-7711

然后就写了个`DoNothingSerializer`，再在fun1的类上添加`@DefaultSerializer(value = DoNothingSerializer.class)`的注解，由于已经到了晚上（晚上十点）方法没有被审核，就早早的睡觉了。

第二天赶紧催了下同事审核，审核过之后，满怀信心的去执行，发现还是不行，当场崩溃！
第二次报错
这次报错跟之前不一样，至少跟udf相关的出来了，如下
```
Serialization trace:
typeInfo (org.apache.hadoop.hive.ql.plan.ExprNodeGenericFuncDesc)
colExprMap (org.apache.hadoop.hive.ql.exec.SelectOperator)
childOperators (org.apache.hadoop.hive.ql.exec.JoinOperator)
reducer (org.apache.hadoop.hive.ql.plan.ReduceWork)
org.apache.hive.com.esotericsoftware.kryo.KryoException: java.lang.IllegalArgumentException: Illegal character in path at index 0: verifyPackTypeh
Serialization trace:
typeInfo (org.apache.hadoop.hive.ql.plan.ExprNodeGenericFuncDesc)
colExprMap (org.apache.hadoop.hive.ql.exec.SelectOperator)
childOperators (org.apache.hadoop.hive.ql.exec.JoinOperator)
reducer (org.apache.hadoop.hive.ql.plan.ReduceWork)
```
里面有个`verifyPackType`, 看到这个时候，心里一阵窃喜，因为这说明是跟udf有关的错，不是自己sql的问题,然后拿着这个单词去项目里搜，搜来搜去都没有，很郁闷，怎么会这样！突然，我想到这是不是上一个版本的问题，是不是我的udf的引入不对，于是我做了如下两次尝试：
第一次尝试：
```
DELETE jar hdfs:///tmp/udf/fun_v2.0.jar;
DROP TEMPORARY FUNCTION fun1;
ADD jar hdfs:///tmp/udf/fun_v2.0.jar;
CREATE TEMPORARY FUNCTION fun1 'com.quick.udf.fun1';

select fun1(1,2,3,4,5) from tablea;


DELETE jar hdfs:///tmp/udf/fun_v1.0.jar;
DROP TEMPORARY FUNCTION fun2;
ADD jar hdfs:///tmp/udf/fun_v1.0.jar;
CREATE TEMPORARY FUNCTION fun2 'com.quick.udf.fun2';

select fun2(1,2,3) from table b
```

```
DELETE jar hdfs:///tmp/udf/fun_v2.0.jar;
DROP TEMPORARY FUNCTION fun1;
ADD jar hdfs:///tmp/udf/fun_v2.0.jar;
CREATE TEMPORARY FUNCTION fun1 'com.quick.udf.fun1';
select fun1(a,b,c,d,e) from tablec
```
我把它们拆成了两次去执行，注意是两个窗口，结果是正常执行，没有报错！！！**看来是udf用的有问题**。
第二次尝试，每次用完udf就及时的drop掉

```
DELETE jar hdfs:///tmp/udf/fun_v2.0.jar;
DROP TEMPORARY FUNCTION fun1;
ADD jar hdfs:///tmp/udf/fun_v2.0.jar;
CREATE TEMPORARY FUNCTION fun1 'com.quick.udf.fun1';

select fun1(1,2,3,4,5)

DELETE jar hdfs:///tmp/udf/fun_v1.0.jar;
DROP TEMPORARY FUNCTION fun2;
ADD jar hdfs:///tmp/udf/fun_v1.0.jar;
CREATE TEMPORARY FUNCTION fun2 'com.quick.udf.fun2';

select fun2(1,2,3) from table b
DROP TEMPORARY FUNCTION fun2;
DELETE jar hdfs:///tmp/udf/fun_v1.0.jar;


DELETE jar hdfs:///tmp/udf/fun_v2.0.jar;
DROP TEMPORARY FUNCTION fun1;
ADD jar hdfs:///tmp/udf/fun_v2.0.jar;
CREATE TEMPORARY FUNCTION fun1 'com.quick.udf.fun1';
select fun1(a,b,c,d,e) from tablec
DROP TEMPORARY FUNCTION fun1;
DELETE jar hdfs:///tmp/udf/fun_v2.0.jar;
```
然后放在一个脚本里一起执行，竟然成功了！！！
当时感觉整个天都亮了，世界如此的美好！！！

冷静下来后，就准备了测试的脚本，每天都来跑一下，关注一下运行的情况，这样第二天就过去了！

第三天，早上慢悠悠的打开电脑，把测试脚本跑了起来，自己喝着茶慢慢的看结果，还哼着歌。。。结果又报错了！！！心里咯噔一下。。。报错如下：

![image-20200708105118033](D:\develop\code\fs_notes\image-20200708105118033.png)

提示`resultmap`的问题，突然醒悟过来，难道两个udf在使用的时候“错位”了？这样只能该项目了，时间紧迫，花了十分钟将目录结构改造如下

```
module1
└─src
    ├─main
       └─java
			└─com
		      └─quick
              	└─udf
              	  └─fun1
module2              	  
└─src
    ├─main
       └─java
			└─com
		      └─quick
              	└─seudf
              	  └─fun2
```

这样保证了两个udf不在同一个包下，也保证了一个jar只对应一个udf

最后重新打包

`fun1_v3.0.jar`和`fun2_v3.0.jar`

对应class为

`com.quick.udf.fun1`和`com.quick.seudf.fun1`

然后赶紧提交审核，最终的sql如下

```
DROP TEMPORARY FUNCTION fun1;
DELETE jar hdfs:///tmp/udf/fun_v3.0.jar;
ADD jar hdfs:///tmp/udf/fun_v3.0.jar;
CREATE TEMPORARY FUNCTION fun1 'com.quick.udf.fun1';

select fun1(1,2,3,4,5)

DELETE jar hdfs:///tmp/udf/fun_v3.0.jar;
DROP TEMPORARY FUNCTION fun2;
ADD jar hdfs:///tmp/udf/fun_v3.0.jar;
CREATE TEMPORARY FUNCTION fun2 'com.quick.seudf.fun2';

select fun2(1,2,3) from table b


select fun1(a,b,c,d,e) from tablec

```

然后执行，没有报错，后续我每天都会去执行，大约持续了一周，都没有报错，到此，问题得到解决！！！



### 后续

自己不是做hive开发的，也是第一次写udf，遇到这些稀奇古怪，真的是无助。。。。但是始终相信那句话

**大胆假设小心求证**