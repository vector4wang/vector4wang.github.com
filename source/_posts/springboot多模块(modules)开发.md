---
title: springboot多模块(modules)开发
date: 2017-04-24 22:31:16
categories: 技术
tags:
  - SpringBoot
  - 项目管理
  - modules
---

>历史上的今天，那是在我国古代的这一天，蒙古人铁木真中年得了一种因脱发导致变成秃头的不治之症，因为之前从为见过此病例，所以便以铁木真的名字来命名此病，也就是现在大家都知道的“老铁没毛病”。

<!--more-->

### 为何模块开发

　　先举个栗子，同一张数据表，可能要在多个项目中或功能中使用，所以就有可能在每个模块都要搞一个mybatis去配置。如果一开始规定说这张表一定不可以改字段属性，那么没毛病。但是事实上， 一张表从项目开始到结束，不知道被改了多少遍，所以，你有可能在多个项目中去改mybatis改到吐血！
在举一个栗子，一个web服务里包含了多个功能模块，比如其中一个功能可能会消耗大量资源和时间，当用户调用这个功能的时候，可能会影响到其他功能的正常使用，这个时候，如果把各个功能模块分出来单独部署，然后通过http请求去调用，至于性能和响应速度，再单独去优化，将会非常爽！这也有利于将来的
分布式集群。
　　根据当前的业务需求，我需要重构现有的web功能，多模块化，然后单独部署，基本架构示意图如下
![示意图.png](http://upload-images.jianshu.io/upload_images/3167229-52dbde4d30bc1eba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 怎样分模块
注意:下面配置的步骤是基于IntelliJ IDEA 2016.3.4(64)，不保证eclipse能成功。如果你还在使用eclipse，建议你删掉它，使用idea吧
1、创建maven主项目例如，springbootmodules，并删掉src文件
2、右键项目分别创建三个module，dao,service1,service2
3、将之前项目用到的依赖写在主项目的pom里，这里要注意
4、dao层主要提供实体类，CURD接口和xml映射文件
5、一定要在service1和service2配置数据库的相关信息，并添加spring的相关配置
6、编写接口测试

### 相关代码
父项目pom
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.boot.lean</groupId>
    <artifactId>springbootquick</artifactId>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>dao</module>
        <module>service1</module>
        <module>service2</module>
    </modules>


    <packaging>pom</packaging>
    <name>springbootquick</name>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>

        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <shadedClassifier>bin</shadedClassifier>
        <java.version>1.8</java.version>


        <mybatis-spring-boot>1.2.0</mybatis-spring-boot>
        <mysql-connector>5.1.39</mysql-connector>
    </properties>


    <dependencies>

        <!-- Spring Boot Web 依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring Boot Test 依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- Spring Boot Mybatis 依赖 -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis-spring-boot}</version>
        </dependency>

        <!-- MySQL 连接驱动依赖 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql-connector}</version>
        </dependency>

        <!-- Junit -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
            <version>2.4.2</version>
        </dependency>
        <dependency>
            <groupId>commons-beanutils</groupId>
            <artifactId>commons-beanutils</artifactId>
            <version>1.9.2</version>
        </dependency>

        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>

        <dependency>
            <groupId>commons-dbcp</groupId>
            <artifactId>commons-dbcp</artifactId>
            <version>1.4</version>
        </dependency>

        <dependency>
            <groupId>net.sf.json-lib</groupId>
            <artifactId>json-lib</artifactId>
            <version>2.4</version>
            <classifier>jdk15</classifier>
        </dependency>

    </dependencies>


    <build>
        <plugins>


            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>


            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.19.1</version>
                <configuration>
                    <skipTests>true</skipTests>    <!--默认关掉单元测试 -->
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.2</version>
                <configuration>
                    <verbose>true</verbose>
                    <overwrite>true</overwrite>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>5.1.30</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
</project>
```
dao模块的pom(里面配置了mybatis的逆向功能插件)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springbootquick</artifactId>
        <groupId>com.boot.lean</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>dao</artifactId>
    <packaging>jar</packaging>

    <build>
	<!-- 一定要声明如下配置-->
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
    </build>
</project>
```
service1和service2的pom一样 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springbootquick</artifactId>
        <groupId>com.boot.lean</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>service1</artifactId>
    <packaging>jar</packaging>
    <dependencies>
        <dependency>
            <groupId>com.boot.lean</groupId>
            <artifactId>dao</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
需要注意的是，service模块里我用的是注解配置，如图所示

![结构示意图](http://upload-images.jianshu.io/upload_images/3167229-b360e7b2756a0bdf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注意配置文件里的端口号
具体代码[请点我](https://github.com/VBMHJQS/springbootquick/tree/modules)

### 打包测试
在父项目下执行maven命令
```bash
mvn package
```
service1和service2目录下分别会产生target文件，里面包含可执行jar包，分别执行
```bash
java -jar service1-1.0-SNAPSHOT
java -jar service2-1.0-SNAPSHOT
```
如果一切顺利的话，你可以得出下面的操作结果

![结果图](http://upload-images.jianshu.io/upload_images/3167229-b68d434f68fcf6a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意端口号哦

有什么问题，自行解决，然后你会发现，跨过这个坑，还有无数个坑在等你~
