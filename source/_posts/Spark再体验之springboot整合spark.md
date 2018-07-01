---
title: Spark再体验之springboot整合spark
date: 2017-04-24 22:31:16
categories: 技术
tags:
  - Spark
  - SpringBoot
---

> 有说胎记是前世死的方式，偶肚子上有个，于是想，难不成上辈子是被人捅死的，谁那么狠。。。后来遇到个人，在同样的位置也有个类似的，忽然就平衡了。
 神回复：也可能你们俩上辈子是很烤串
<!--more-->



![spark](http://upload-images.jianshu.io/upload_images/3167229-2f410d11c2bb5d67.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 前言
　　[上一篇](http://www.jianshu.com/p/52d0f043cc04)主要讲的是spark环境的搭建和任务的提交，这一篇是将spark直接部署在springboot搭建的web服务里，一些数据逻辑交给spark去处理，至于原理等我对spark有了更深的理解再来一一讲述！
### 编码
　　使用springboot快速搭建一个web框架，之前对pom中的依赖配置不是怎么在意，进过spark和scala版本的坑之后，发现想配置一个完美的pom是多么的不容易，下面倾情奉送
```xml
 <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.3.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <scala.version>2.10.4</scala.version>
        <spark.version>1.6.2</spark.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>

        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>${scala.version}</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.4.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.10</artifactId>
            <version>${spark.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-launcher_2.10</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-mllib_2.10</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming_2.10</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.4</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.specs</groupId>
            <artifactId>specs</artifactId>
            <version>1.2.5</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.ansj</groupId>
            <artifactId>ansj_seg</artifactId>
            <version>5.1.1</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

        </plugins>
    </build>
```
这里包含了springboot和spark需要的依赖

然后再写一个计算单词个数的方法，这个程序跟以前的一样，只是`SparkConfig`的配置有所改变
```java
@Component
public class WordCountService implements Serializable {
    private static final Pattern SPACE = Pattern.compile(" ");

    @Autowired
    private transient JavaSparkContext sc;

    public Map<String, Integer> run() {
        Map<String, Integer> result = new HashMap<>();
        JavaRDD<String> lines = sc.textFile("C:\\Users\\bd2\\Downloads\\blsmy.txt").cache();

        lines.map(new Function<String, String>() {
            @Override
            public String call(String s) throws Exception {
                System.out.println(s);
                return s;
            }
        });
        
        System.out.println(lines.count());

        JavaRDD<String> words = lines.flatMap(new FlatMapFunction<String, String>() {


            @Override
            public Iterable<String> call(String s) throws Exception {
                return Arrays.asList(SPACE.split(s));
            }
        });

        JavaPairRDD<String, Integer> ones = words.mapToPair(new PairFunction<String, String, Integer>() {

            private static final long serialVersionUID = 1L;

            public Tuple2<String, Integer> call(String s) {
                return new Tuple2<String, Integer>(s, 1);
            }
        });

        JavaPairRDD<String, Integer> counts = ones.reduceByKey(new Function2<Integer, Integer, Integer>() {

            private static final long serialVersionUID = 1L;

            public Integer call(Integer i1, Integer i2) {
                return i1 + i2;
            }
        });

        List<Tuple2<String, Integer>> output = counts.collect();
        for (Tuple2<String, Integer> tuple : output) {
            result.put(tuple._1(),tuple._2());

        }

        return result;

    }
}
```
**注意** **注意** **注意**
上面两点写法需要注意
`implements Serializable`和`private transient JavaSparkContext sc`
`transient`为的是不让sc序列化，如果没有它做修饰，你会遇到这样错
```console
Task not serializable] with root cause
java.io.NotSerializableException: com.quick.spark.xxx
```
别说我怎么知道的，这个问题花了整整一下午一把血与泪啊，中文，，英文和日文的解答都尼玛看了。。。文本我用的是《巴黎圣母院》的英文版，下面是结果
### 结果

![字数统计](http://upload-images.jianshu.io/upload_images/3167229-141260bf7f6b7bac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
代码我放在了[GitHub](https://github.com/VBMHJQS/springbootquick/tree/spark)上,有兴趣的可以看一看。
### 后记

　　代码都放在了公司了，自己住的地方网速慢的要死，短短一篇文章写了半个多小时。。。
　　接触spark不到四天，通过demo对其有了更进一步的认识，前几天买的书《Spark快速大数据分析》今天刚到，值得去看一看。

### 后续
早上使用java8提供的lambda表达式改了以下代码，如下图

![lambda](http://upload-images.jianshu.io/upload_images/3167229-d843629a4c795e28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码量减少了一倍，据说效率还提高了。。。
