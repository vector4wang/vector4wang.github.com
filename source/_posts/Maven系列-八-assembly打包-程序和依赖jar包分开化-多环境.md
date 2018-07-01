---
title: Maven系列(八)assembly打包-程序和依赖jar包分开化+多环境
date: 2017-06-24 11:02:00
categories: 技术
tags:
	- maven
	- 部署
	- 项目管理
	- assembly
	- 多环境
---

会使用assembly插件对程序和依赖jar包分开化，下面是多环境的配置

<!--more-->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=27646205&auto=1&height=66"></iframe>

## 前言
上一篇介绍的是“assembly打包-程序和依赖jar包分开化”的配置方法， 这一篇就来介绍下如何多环境的配置，这里请看清楚，是“程序和依赖jar包分开化+多环境”跟之前的不太一样哦。

## 需要修改的配置

项目的目录结构
![工程结构.png](http://upload-images.jianshu.io/upload_images/3167229-b9316fad6897592f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### pom
添加`profile`配置，我这里同样配置了三种环境
```xml
<profiles>
        <profile>
            <id>local</id>
            <properties>
                <env>local</env>
            </properties>
            <!-- 如果不指定ID，默认是本地环境-->
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <profile>
            <id>test</id>
            <properties>
                <env>test</env>
            </properties>
        </profile>
        <profile>
            <id>product</id>
            <properties>
                <env>product</env>
            </properties>
        </profile>
    </profiles>
```

### package.xml
新增了两处
```xml
<fileSets>
        <!--需要包含的文件与输出的路径-->
        <fileSet>
            <directory>src/main/bin</directory>
            <outputDirectory>bin/</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>src/main/resources</directory>
            <outputDirectory>/</outputDirectory>
            <!-- 去除需要多环境配置的文件-->
            <excludes>
                <exclude>application.properties</exclude>
            </excludes>
        </fileSet>
        <!--多环境配置-->
        <fileSet>
            <!--${env} 可以获取打包命令里的参数-->
            <directory>src/main/resources/env/${env}/</directory>
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
```

## 多环境打包测试

环境|命令
---|---
本地|`mvn clean package -P local`
测试|`mvn clean package -P test`
生产|`mvn clean package -P product`

## 后记

- 关于assembly打包，mybatis的xml访问不了的问题已经解决了，注意配置`mybatis.mapperLocations=classpath:mapper/*.xml`
- maven的功能之强大到你无法想象，我之前的一系列文章对我所接触到的maven所有用法都有较详细的配置说明
- 以后有可能会开始尝试使用gradle打包

