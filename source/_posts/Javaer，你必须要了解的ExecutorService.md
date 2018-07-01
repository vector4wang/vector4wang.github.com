
---
title: Javaer，你必须要了解的ExecutorService
date: 2017-04-24 22:31:16
categories: 技术
tags:
  - Java
  - Thread
  - 线程池
---

之前做的一个功能里有一个耗时操作：处理数据库里对应的记录，然后将每个处理后的结果做个排序。
恕本人小白，刚开始直接用单线程处理！你敢信？！

<!--more-->

### ExecutorService初接触
　　之前做的一个功能里有一个耗时操作：处理数据库里对应的记录，然后将每个处理后的结果做个排序。
　　 恕本人小白，刚开始直接用单线程处理！你敢信？！然后60多万条记录，跑了三分钟才出结果！当时我就震惊了，这尼玛要被“刁”的节奏啊。但我并没有什么好的解决方案，便去咨询老大，然后老大直接丢过来一段代码附带几个字
```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```
> 用`ExecutorService`弄个线程池跑


使用了`ExecutorService`之后，我便爱上了它！！！
### 简单介绍
　　什么是`ExecutorService`呢？它实现`Executor`中的方法，jdk中这样介绍它
> 
public interface Executor
执行已提交的 [Runnable
](http://tool.oschina.net/uploads/apidocs/jdk-zh/java/lang/Runnable.html) 任务的对象。此接口提供一种将任务提交与每个任务将如何运行的机制（包括线程使用的细节、调度等）分离开来的方法。
。。。
此包中提供的 Executor
 实现实现了 [ExecutorService
](http://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/concurrent/ExecutorService.html)，这是一个使用更广泛的接口。 [ThreadPoolExecutor
](http://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/concurrent/ThreadPoolExecutor.html) 类提供一个可扩展的线程池实现。 [Executors
](http://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/concurrent/Executors.html) 类为这些 Executor 提供了便捷的工厂方法。

### 使用场景
　　关于场景，就拿我最近做的功能举例
##### 无须等待结果
上一篇[采集领英信息](http://www.jianshu.com/p/d3533550b69c)最后提到采集慢的原因有一个是“使用单线程”，那好，现在使用`ExecutorService`创建线程池来加快采集速度(代码就不贴了，写一个大概的思路)：
- 创建一个线程池`ExecutorService executorService = Executors.newCachedThreadPool();`该方法创建一个数量没有上限的线程池

- 将采集代码中for循环的内容抽出来重构为一个集成`Thread`的类，因为这样可以传递一些参数，类似下面代码

```java
public class CacheThreadPool {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < 15; i++)
            exec.execute(new MyThread("张"+i));
        exec.shutdown();//并不是终止线程的运行，而是禁止在这个Executor中添加新的任务
    }
}

class MyThread extends Thread {

    private String name;

    public MyThread(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println("获取"+name+"的信息并保存");
    }
}
```
需要注意的是，我for循环中，我不需要等待线程执行结果，因为我不关心每一条是否能成功解析，是否能成功插入数据库。

##### 需要结果
　　这个就是开头提到的。现在要开线程跑，但是我不能保证我for循环结束了，每个线程都能返回给我结果，也就是说，我必须要等待线程将60万条数据都处理完，我拿到结果之后，程序才能继续下去。
　　jdk里有提供**invokeAll**方法
```java
 <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;
```
*执行给定的任务，当所有任务完成时，返回保持任务状态和结果的 Future 列表*
这难道就是所谓的“批量执行”？
于是我写了个demo测试了一下，for循环十次，每次处理耗时1s，单线程处理的话就是10s，无可厚非(排除其他一切因素，最理想状态)，然后用invokeAll方法批量执行，我预测结果是1s(傻子也猜到了)代码如下:
```java
package com.quick.thread;

import java.util.concurrent.Callable;

/**
 * Created by wangxc on 2017/3/29.
 */
public class TaskSleep implements Callable<Integer> {

    private int num;

    public TaskSleep(int num){
        this.num = num;
    }

    @Override
    public Integer call() throws Exception {
//        System.out.println(num + "--->" +i);
        Thread.sleep(1000);
        return  num;
    }
}
```
```java
package com.quick.thread;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

/**
 * Created by wangxc on 2017/3/29.
 */
public class ExecutorServiceDemo {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        long stat = System.currentTimeMillis();
        ExecutorService executorService = Executors.newCachedThreadPool();
        List<TaskSleep> callList = new ArrayList<TaskSleep>();
        for(int i=0;i<10;i++){
            callList.add(new TaskSleep(i));
        }
        List<Future<Integer>> futures = executorService.invokeAll(callList);
        executorService.shutdown();
        int sum=0;
        for(Future<Integer> item:futures){
            sum += item.get();
        }
        System.out.println("结果"+sum);
        long end = System.currentTimeMillis();
        System.out.println((double)(end-stat)/1000);
    }
}
```
结果
```console
结果45
1.045
```
嗯，这就是我想要的结果
### 最后
`ExecutorService`使用的方式非常之多，适合自己的业务场景才是最主要的，本文是根据自己的业务需求去使用`ExecutorService`,仅供参考~
