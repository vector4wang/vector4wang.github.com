---
title: 面试系列之Integer缓存所引发的惨案(保证看完你就彻底明白)
copyright: true
permalink: 1
top: 0
date: 2019-08-28 20:52:16
tags:
	- 面试
	- Java
	- 缓存
	- 包装类型
categories:
	- 技术
password:
---



Integer缓存讲解，包！你！会！

<!-- more -->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=204524&auto=1&height=66"></iframe>



今天在整理代码的时候发现了一段程序，如下

```java
Integer integer1 = 3;
Integer integer2 = 3;

if (integer1 == integer2)
  System.out.println("integer1 == integer2");
else
  System.out.println("integer1 != integer2");

Integer integer3 = 129;
Integer integer4 = 129;

if (integer3 == integer4)
  System.out.println("integer3 == integer4");
else
  System.out.println("integer3 != integer4");

System.out.println(Integer.valueOf("127")==Integer.valueOf("127"));
System.out.println(Integer.valueOf("128")==Integer.valueOf("128"));
System.out.println(Integer.parseInt("128")==Integer.valueOf("128"));
```

我看了一眼，只记得答案是

```
integer1 == integer2
integer3 != integer4
true
false
true
```

我刚想把代码关闭，脑海里突然黑人问号？

> 为什么会是这样？

还别说，我竟然一时语噻，说不出个所以然来，考虑到这是一个面试题，而且也想去了解下设计原理，索性花了点时间去重新整理了一下，做个记录！



说明：本文使用的Java版本号如下

```bash
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
```





### 缓存

这里考察的主要是包装类型的缓存，我们点开`Integer`的源码，会发现如下代码

```java
private static class IntegerCache {
  // 最小值
  static final int low = -128;
  // 最大值，支持自定义
  static final int high;
  // 缓存数组
  static final Integer cache[];

  static {
    // 最大值可以通过属性配置来改变
    int h = 127;
    String integerCacheHighPropValue =
      sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
    // 如果设置了对应的属性，则使用该值
    if (integerCacheHighPropValue != null) {
      try {
        int i = parseInt(integerCacheHighPropValue);
        i = Math.max(i, 127);
        // 最大数组大小为Integer.MAX_VALUE
        h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
      } catch( NumberFormatException nfe) {
        // If the property cannot be parsed into an int, ignore it.
      }
    }
    high = h;
		
    cache = new Integer[(high - low) + 1];
    int j = low;
    // 将low-high范围内的值全部实例化并存入数组中当缓存使用
    for(int k = 0; k < cache.length; k++)
      cache[k] = new Integer(j++);

    // range [-128, 127] must be interned (JLS7 5.1.7)
    assert IntegerCache.high >= 127;
  }

  private IntegerCache() {}
}
```

这里就是`Integer`包装类型里的缓存声明:

`Integer`第一次使用的时候就会初始化缓存，其中范围最小值为-128，最大值默认是127，它可以通过`-Djava.lang.Integer.IntegerCache.high=xxx`或者`-XX:AutoBoxCacheMax=xxx`来设置，如下图：

![1.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pLmxvbGkubmV0LzIwMTkvMDgvMjgveDlvcThLWGtMQ3dHWk5ULnBuZw?x-oss-process=image/format,png)



![2.jpg](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pLmxvbGkubmV0LzIwMTkvMDgvMjgvcnBJUmNMTVhVN29QNGxHLmpwZw?x-oss-process=image/format,png)

接着会把low至high中所有的数据初始化存入数据中，默认就是将**-128~127**总共256个数循环实例化存入`cache`数组中。准确的说应该是将这256个对象在内存中的地址存进数组中。

再来看看其他包装类型中缓存的支持；

| 基本类型 | 大小  |  最小值   |           最大值           | 包装器类型 | 缓存范围 | 是否支持自定义 |
| :------: | :---: | :-------: | :------------------------: | :--------: | :------: | :------------: |
| boolean  |   -   |     -     |                            |  Bloolean  |          |                |
|   char   | 6bit  | Unicode 0 |      Unic ode 2(16)-1      | Character  |  0~127   |       否       |
|   byte   | 8bit  |   -128    |            +127            |    Byte    | -128~127 |       否       |
|  short   | 16bit |  -2(15)   |          2(15)-1           |   Short    | -128~127 |       否       |
|   int    | 32bit |  -2(31)   |          2(31)-1           |  Integer   | -128~127 |      支持      |
|   long   | 64bit |  -2(63)   |          2(63)-1           |    Long    | -128~127 |       否       |
|  float   | 32bit |  IEEE754  |          IEEE754           |   Float    |    -     |                |
|  double  | 64bit |  IEEE754  |          IEEE754           |   Double   |    -     |                |
|   void   |   -   |           | -                        - |    Void    |          |                |




这里又来了多个黑人问号

- -128~127怎么来的？
- 为什么是-128 ~ 127?怎么不是-200 ~ 200呢？
- 为什么需要缓存数据？

后面会讲到，我们先把程序走完~



### 对象的初始化

通过使用IDEA的**Show ByteCode**我们可以得到代码在`JVM`里的亚子，如图

![3.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pLmxvbGkubmV0LzIwMTkvMDgvMjgvWVNRWFVNN0NodFpCV3VuLnBuZw?x-oss-process=image/format,png)


我们可以看到`Integer integer1 = 3;` 实际上是通过`Integer.valueOf`返回一个`Integer`对象，我们再进入源码，它的最终现实如下:

```java
public static Integer valueOf(int i) {
  if (i >= IntegerCache.low && i <= IntegerCache.high)
    // int j = low;cache[k] = new Integer(j++);
    return IntegerCache.cache[i + (-IntegerCache.low)];
  return new Integer(i);
}
```

很明显了，首先判断`i`是否在已经被缓存，如果是的话直接从缓存中取出(地址)返回，否则就**重新实例化**一个。
**注意: 恶心的就在这里，有就从缓存拿，没有就实例化！这也就牵扯出`Integer Swap`的问题，后面文章会讲到。**


### 对象之间的比较

#### ==

- 基本数据类型：`byte`,`short`,`char`,`int`,`long`,`double`,`float`,`blooean`,它们之间的比较,比较是它们的值；
- 引用数据类型：使用`==`比较的时候，比较的则是它们在内存中的地址(heap上的地址)。

#### equals

这个是`Object`下的一个方法，对应代码如下：

```java
public boolean equals(Object obj) {
  return (this == obj);
}
```

默认也是比较两个对象之间的内存地址，而其他对象在继承`Object`的时候一般会去`Overwrite`此方法，所以在没有`Overwrite`的情况下用`equals`的结果跟`==`一样都是比较内存地址，否则则按照`Overwrite`规则来。

### 答案分析

再来看看本处的代码，使用`==`则认为是比较前后两者的内存地址(以下比较均使用默认的缓存大小即-128~127)

- 第一个比较`integer1 == integer2`前后都是3，通过上述的[对象初始化](#对象初始化)我们可以知道`integer1`和`integer2`**均在缓存数组**中，所以`Integer`直接从缓存数组中去取出地址返回，那边显而易见的，当使用`==`的时候则为`true`;
- 第二个比较`integer3 == integer4`前后都是129，不在**-128~127**范围内，则`Integer`需要**重新实例化**，所以`integer3`和`integer4`都是重新实例化的对象，对应的地址自然而然的也就不一样了，所以结果为`false`;
- 第三和和第四个同理；
- 第五个比较 `Integer.parseInt("128")==Integer.valueOf("128")`有意思，我们先看右边，128不在缓存内需要重新实例化一个，再看左边`Integer.parseInt("128")`返回值是`int`型，最终的比较变成了`128==Integer(128)`，再看`ByteCode`,如下图，我们会发现`Integer(128)`最终使用的是`Integer.intValue ()`方法，哈哈，竟做了**拆箱处理**，finally，比较就变成了`128==128`,结果当然就是`true`了。

![4.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pLmxvbGkubmV0LzIwMTkvMDgvMjgvVGp4T0RHdzRSMTVzSWVGLnBuZw?x-oss-process=image/format,png)

### 扩展

回到第二节的两个问题：

- -128~127怎么来的？
- 为什么是-128 ~ 127?怎么不是-200 ~ 200呢？
- 为什么需要缓存数据？

后来在网上查找，找到一个比较靠谱的[解释](https://javapapers.com/java/java-integer-cache/)：

> 实际上，在Java 5中首次引入此功能时，范围固定为-127到+127。 后来在Java 6中，范围的最大值映射到java.lang.Integer.IntegerCache.high，VM参数允许我们设置高位数。 根据我们的应用用例，它可以灵活地调整性能。 应该从-127到127选择这个数字范围的原因应该是什么。这被认为是广泛使用的整数范围。 在程序中首次使用Integer必须花费额外的时间来缓存实例。

[Java Language Specification](http://docs.oracle.com/javase/specs/jls/se7/html/jls-5.html#jls-5.1.7) 的说明如下：

> *Ideally, boxing a given primitive value* `p`*, would always yield an identical reference. In practice, this may not be feasible using existing implementation techniques. The rules above are a pragmatic compromise. The final clause above requires that certain common values always be boxed into indistinguishable objects. The implementation may cache these, lazily or eagerly. For other values, this formulation disallows any assumptions about the identity of the boxed values on the programmer's part. This would allow (but not require) sharing of some or all of these references.*

> *This ensures that in most common cases, the behavior will be the desired one, without imposing an undue performance penalty, especially on small devices. Less memory-limited implementations might, for example, cache all* `char` *and* `short` *values, as well as* `int` *and* `long` *values in the range of -32K to +32K.*

[API]( https://docs.oracle.com/javase/7/docs/api/java/lang/Integer.html#valueOf(int)) 上解释

> Returns an Integer instance representing the specified int value. If a new Integer instance is not required, this method should generally be used in preference to the constructor Integer(int), as this method is likely to yield significantly better space and time performance by caching frequently requested values. This method will always cache values in the range -128 to 127, inclusive, and may cache other values outside of this range



我的理解就是：

- -128~127范围内的数是比较频繁使用的，JDK就增加了这一默认的范围但并不是不可变的，毕竟JDK提供了两种方法供用户自定义范围；
- 事先做缓存还是为了提高空间和时间性能！
(大家参照上面官方与非官方的解释，自己意会哈~)


### 附加题

下面的输出是什么：

```java
Integer integer5 = new Integer(3);
Integer integer6 = new Integer(3);
if (integer5 == integer6)
  System.out.println("integer5 == integer6");
else
  System.out.println("integer5 != integer6");
```

本文的示例代码: [Github](https://github.com/vector4wang/java-learning-quick/blob/master/feature-learning/src/main/java/com/feature/learn/cache/JavaIntegerCache.java)

参考：

https://stackoverflow.com/questions/20897020/why-integer-class-caching-values-in-the-range-128-to-127

https://stackoverflow.com/questions/20877086/why-do-comparisons-with-integer-valueofstring-give-different-results-for-12

https://docs.oracle.com/javase/specs/jls/se7/html/jls-5.html#jls-5.1.7

https://javapapers.com/java/java-integer-cache/

