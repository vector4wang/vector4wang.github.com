---
title: springboot日志体系---log4j2
date: 2017-07-02 12:12:15
categories: 技术
tags:
	- 日志
	- springboot
	- log4j2
	- 颜色
	- 控制台
---

最近调试代码和运行代码，一些日志打印的乱七八槽，根据日志很难快速定位到问题，感觉自己是为了打印日志而打印日志，花了点时间把日志的相关整理了一下，意在让日志发挥最大的作用。

<!--more-->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=95871&auto=1&height=66"></iframe>


### 前言
 本文解决以下问题：
 - 为何使用log4j2
 - springboot下log4j2日志的使用
 - 控制台日志显示的级别和文件保存的日志不同
 - idea控制台颜色日志的输出
    

### 正文
### log4j2
目前有关日志的开源代码很多，如log4j、sl4j和log4j2,为什么我选择使用log4j2呢，看完下面两篇性能的对比，相信你也会选择log4j2
http://www.jianshu.com/p/483a9cf61c36
https://blog.souche.com/logback-log4j-log4j2shi-ce/?utm_source=tuicool&utm_medium=referral
### springboot集成Log4j2
需要将springboot内置的日志剃掉,然后引入log4j2，pom如下
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```
然后需要在resource下面添加log4j2.xml配置文件，当然了如果你不添加，springboo会提示你没有对应文件，并使用默认的配置文件，这个时候级别可以在application.properties中配置
`logging.level.root=error`
控制台打印结果

![图一.png](http://upload-images.jianshu.io/upload_images/3167229-92a529dd6e87f03b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当然了，使用配置文件，配置可以多样化,下面是默认的log4j2配置,log4j2支持xml、json、yml格式的配置
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<configuration status="OFF">  
  <appenders>  
    <Console name="Console" target="SYSTEM_OUT">  
      <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>  
    </Console>  
  </appenders>  
  <loggers>  
    <root level="error">  
      <appender-ref ref="Console"/>  
    </root>  
  </loggers>  
</configuration>  
```
主要结构，和我们用到的大致如下
![图二.png](http://upload-images.jianshu.io/upload_images/3167229-d79d1e496c110695.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
appenders里设置日志的输出方式、级别和格式
loggers里设置全局的级别和绑定appenders里的name

- File 日志输出到文件，可配置覆盖还是追加
- RollingFile “滚动文件”可作为按日输出日志的方式
- Console 控制台日志

- PatternLayout 格式化输出日志
- ThresholdFilter“阈值筛选器” 可单独设置appender的输出级别

- loggers里需要匹配每个appender的名称 name 

详细参见官网：https://logging.apache.org/log4j/2.x/manual/configuration.html#AutomaticConfiguration

### 稍复杂的需求
我的服务一般放在linux服务器上跑，可能要实时查看日志，现有这个需求“我要打印到控制台的日志级别为Error，日志文件里保存的是INFO级别的日志”这样在产生错误的时候，就不会被大量无用的代码干扰。
要使用ThresholdFilter，配置如下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="OFF">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="ERROR" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="%d{yyyy.MM.dd 'at' HH:mm:ss z} %-5level %class{36} %M() @%L - %msg%n"/>
        </Console>
        <File name="ERROR" fileName="logs/error.log" append="false">
            <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="%d{yyyy.MM.dd 'at' HH:mm:ss z} %-5level %class{36} %M() @%L - %msg%n"/>
        </File>
        <!--这个会打印出所有的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
        <RollingFile name="RollingFile" fileName="logs/app.log"
                     filePattern="log/$${date:yyyy-MM}/app-%d{MM-dd-yyyy}-%i.log.gz">
            <PatternLayout pattern="%d{yyyy.MM.dd 'at' HH:mm:ss z} %-5level %class{36} %M() @%L - %msg%n"/>
            <SizeBasedTriggeringPolicy size="5MB"/>
        </RollingFile>
    </Appenders>
    <Loggers>
        <Root level="INFO">
            <appender-ref ref="ERROR" />
            <appender-ref ref="RollingFile"/>
            <appender-ref ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```
不要被吓到了，按照我上面的思维导图分析一下就很清晰了：
三个appender：Console、File、RollingFile
- Console 通过ThresholdFilter过滤规则只输出ERROR级别的错误(onMatch="ACCEPT" onMismatch="DENY" 匹配到的接受，没有匹配的走人)
- File 也通过ThresholdFilter的方式输出到日志，当然了append="false" 会在服务每次启动的时候清空日志(覆盖)
- RollingFile 因为日志全局设置的为INFO，所以不需要ThresholdFilter,这里只需要指定filePattern和SizeBasedTriggeringPolicy就行了

执行代码，查看各文件和控制台
![图三.png](http://upload-images.jianshu.io/upload_images/3167229-eb9fa3afbe5ecdae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

完美~

### 控制台颜色输出日志
注：本人使用的是IDEA，没有使用Eclipse，可能Eclipse也有类似的插件。
本地开发与调试代码的时候，会不会感觉同样的颜色找日志头都大了，别担心，idea知道你头会大，所以提供了一个插件**Grep Console**,让你头慢慢的小下来~下载并重启后，这里需要注意将插件默认的配色关闭，当然了你可以通过自定义配色，我这里是在xml配置的
![图四.png](http://upload-images.jianshu.io/upload_images/3167229-2aa48b6a8f482564.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后log4j2.xml配置的如下
```xml
...
<Console name="Console" target="SYSTEM_OUT">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="%highlight{%d{yyyy.MM.dd 'at' HH:mm:ss z} %-5level %class{36} %M() @%L - %msg%n}{FATAL=Bright Red, ERROR=Bright Magenta, WARN=Bright Yellow, INFO=Bright Green, DEBUG=Bright Cyan, TRACE=Bright White}"/>
        </Console>
...
```
结果
![图五.png](http://upload-images.jianshu.io/upload_images/3167229-c8ccdee20104db2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

相当完美~

### 后记
相信有了明确的日志输出，能提高我们排错的效率，当别人看日志累的揉眼睛的时候，我们早已喝着茶水，唱着歌~

代码在[这里](https://github.com/vector4wang/spring-boot-quick/tree/master/quick-log)
