---
title: Maven系列(七)assembly打包-程序和依赖jar包分开化
date: 2017-05-23 23:52:47
categories: 技术
tags:
	- maven
	- 部署
	- 项目管理
---

如果对maven不会用甚至不知道是什么的话，建议先看看下面几篇，看完，相信你会有所启发，并会对项目进行一个完整的依赖构建-打包测试-部署发布

<!--more-->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=299036&auto=1&height=66"></iframe>

如果对maven不会用甚至不知道是什么的话，建议先看看下面几篇，看完，相信你会有所启发，并会对项目进行一个完整的依赖构建-打包测试-部署发布

- [Maven系列（一）Maven的简介与使用](http://blog.csdn.net/qqhjqs/article/details/46963495)
- [Maven系列（二）无Maven不项目---使用Eclipse快速搭建Maven项目 ](http://blog.csdn.net/qqhjqs/article/details/47045585)
- [Maven系列（三）Maven给不同的环境打包 ](http://blog.csdn.net/qqhjqs/article/details/53495535)
- [Maven系列（四）Maven热部署 ](http://blog.csdn.net/qqhjqs/article/details/51594583 )
- [Maven系列（五）CentOS7搭建最新GitLab ](http://blog.csdn.net/qqHJQS/article/details/52950147)
- [Maven系列（六）配合GitLab持续集成（CI）](http://blog.csdn.net/qqHJQS/article/details/53561541)

开发过javaweb项目并发布过的兄dei知道，普通的war包放到tomcat里之后，随着tomcat的启动，war包会自动解压成文件，假如程序里有调用xml资源文件(不是spring相关的配置文件)，那么在程序里指定相对路径就行了，但是最近我有一个服务里，我把一些账户的用户名和密码放在xml里，然后启动的时候调用到文件再使用xpath去取就行了，这是比较常规的做法，idea直接运行咩有问题，但是当我打成jar包在服务器上启动的时候，报错说是找不到资源文件，即我存用户名与密码的xml文件，思来想去，明白了，因为xml文件在jar包里，程序在执行的时候根本就找不到文件，因为jar包没有被解压，根据相对路径也就找不到文件(我是这么理解的，不知道对不对)，这个时候我就明白了我以往的打包方式在这个服务是行不通的。这里说一下，我用的是springboot，打包的使用的是springboot提供的plugin，如下
```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```
一般来讲，使用这个插件，就自动将程序和那些依赖的jar包捆绑在一起，然后通过`java -jar xxx.jar`运行就OK了

再来说说maven打包的方式，整体来讲分为两大类

- 第一种就是上面说的将所有的依赖jar包、资源文件和程序捆绑在一起，比较常见；
- 第二种就是将程序和依赖的jar包和资源文件分开，比较灵活，可以自己编写一些shell脚本来启动或停止程序

各有各的好，具体的还需要视项目来定。
接下来说说第二种打包方式，我这里使用的是maven插件`assembly`,自己顺便写了个linux下执行和停止的脚本，解决了上面的问题，而且使用起来很方便，下面是步骤

### 去掉其他打包的插件

我将springboot的打包插件删掉，使用下面的插件
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <archive>
            <manifest>
                <!--指定main入口-->
                <mainClass>com.quick.Application</mainClass>
                <addClasspath>true</addClasspath>
                <classpathPrefix>lib/</classpathPrefix>
            </manifest>
            <manifestEntries>
                <Class-Path>./</Class-Path>
            </manifestEntries>
        </archive>
        <excludes>
            <exclude>config/**</exclude>
        </excludes>
    </configuration>
</plugin>
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <configuration>
        <!-- not append assembly id in release file name -->
        <appendAssemblyId>false</appendAssemblyId>
        <descriptors>
            <!--打包的详细描述，需要配置额外文件-->
            <descriptor>src/main/build/package.xml</descriptor>
        </descriptors>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
对应的package.xml文件内容为
```xml
<?xml version="1.0" encoding="UTF-8"?>
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3 http://maven.apache.org/xsd/assembly-1.1.3.xsd">
    <id>package</id>
    <formats>
        <!--压缩文件的类型-->
        <format>zip</format>
    </formats>
    <includeBaseDirectory>true</includeBaseDirectory>
    <fileSets>
        <!--需要包含的文件与输出的路径-->
        <fileSet>
            <directory>src/main/bin</directory>
            <outputDirectory>bin/</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>src/main/resources</directory>
            <outputDirectory>/</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>${project.build.directory}</directory>
            <outputDirectory>/</outputDirectory>
            <includes>
                <include>*.jar</include>
            </includes>
        </fileSet>
    </fileSets>
    <dependencySets>
        <dependencySet>
            <outputDirectory>lib</outputDirectory>
            <scope>runtime</scope>
            <excludes>
                <exclude>${groupId}:${artifactId}</exclude>
            </excludes>
        </dependencySet>
    </dependencySets>
</assembly>
```
这些可以直接拿来用，如果想看详细的解析[点我](http://maven.apache.org/plugins/maven-assembly-plugin/assembly.html#)
下面是项目的结构目录

项目里有脚本，在linux里可以执行，执行命令`mvn clean install`之后就会生成对应一个zip压缩包，解压后如图

这样程序和jar包、资源文件分开，就方便调用了，代码上传到了[github](https://github.com/vector4wang/springbootquick/tree/package-assembly)上