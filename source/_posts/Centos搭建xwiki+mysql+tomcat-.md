---
title: Centos搭建xwiki+mysql+tomcat
date: 2017-04-24 22:31:16
categories: 技术
tags:
  - Linux
  - centos
  - xwiki
  - tomcat

---

XWiki是一个由Java编写的基于LGPL协议发布的开源wiki和应用平台。我在的公司，将学习文档、问题分享和一些技术文档都放在xwiki上管理，管理方便、发布简单，它有自己的xwiki语法，但是也支持markdown语法（不爽的是，不支持实时显示），下面就简单的介绍一下在linux下搭建xwiki的步骤。 

<!-- more -->

### 准备工作

+ 去官网下载xwiki的war包[xwiki](http://www.xwiki.org/xwiki/bin/view/Download/DownloadForm?downloadURL=http://download.forge.ow2.org/xwiki/xwiki-enterprise-web-8.3-milestone-1.war&projectVersion=8.3-milestone-1)
**注意:**需要输入邮箱，建议在windows下，下载好war包然后上传到linux里
+ 下载xwiki-enterprise-ui-mainwiki-all，之后导入xwiki项目里(可以安装好xwiki之后下载)
+ 安装mysql数据库
+ 安装tomcat
+ 安装jdk
　　因为本文主要是对xwiki的搭建，所以对后面三者的安装就不在一一阐述。

### 开始搭建

#### 解压war包
　
将下载好的war包移动到tomcat的webapp目录下，然后启动tomcat，启动成功后，war包也就解压完毕了，最后再删掉war包，防止下次启动将后面配置好的xwiki项目覆盖掉。用到的命令如下:
```bash
cp xxx/xwiki.war xxx/tomcat/webappsbin/startup.sh
rm -rf xwiki.war
```

#### 配置文件

在***tomcat/webapps/xwiki/WEB-INF/***目录下配置以下文件：
* hibernate.cfg.xml
* xwiki.cfg

 ##### 配置hibernate.cfg.xml
 
因为xwiki默认的是hsql数据库，所以我们要注释掉hsql的配置，放掉mysql的配置，注意mysql的用户名和密码如下：

```xml 
<!-- Configuration for the default database. Comment out this section and uncomment other sections below if you want to use another database. Note that the database tables will be created automatically if they don't already exist. If you want the main wiki database to be different than "xwiki" (or the default schema for schema based engines) you will also have to set the property xwiki.db in xwiki.cfg file <property name="connection.url">jdbc:hsqldb:file:${environment.permanentDirectory}/database/xwiki_db;shutdown=true</property>....-->
<!-- MySQL configuration. Uncomment if you want to use MySQL and comment out other database configurations. Notes: - if you want the main wiki database to be different than "xwiki" you will also have to set the property xwiki.db in xwiki.cfg file--> 
<property name="connection.url">jdbc:mysql://localhost/xwiki</property>
 <property name="connection.username">root</property> 
 <property name="connection.password"></property> 
 <property name="connection.driver_class">com.mysql.jdbc.Driver</property> 
 <property name="dialect">org.hibernate.dialect.MySQL5InnoDBDialect</property>
 <property name="dbcp.poolPreparedStatements">true</property> 
 <property name="dbcp.maxOpenPreparedStatements">20</property>
 <mapping resource="xwiki.hbm.xml"/> <mapping resource="feeds.hbm.xml"/> 
 <mapping resource="activitystream.hbm.xml"/> 
 <mapping resource="instance.hbm.xml"/> 
 <mapping resource="mailsender.hbm.xml"/>
```

##### 配置xwiki.cfg

这个是xwiki的主要配置文件，需要配置的比较多，找到如下代码并放掉注释

```xml
xwiki.store.main.hint=hibernate
xwiki.store.hibernate.path=/WEB-INF/hibernate.cfg.xml
xwiki.superadminpassword=system
xwiki.readonly=no 
xwiki.encoding=UTF-8 
```
之后可以通过用户名superadmin，密码system来登录。

##### 创建xwiki数据库

配置好了mysql的配置，当然要创建对应的xwiki的数据库了进入mysql的命令行，输入
```mysql
create database xwiki;
grant all privileges on xwiki.* to xwiki@127.0.0.1 identified by 'xwiki';
```
**注意：**这里需要下载mysql的驱动包放在**xwiki/WEB-INF/lib/**下，命令如下：
```bash
wget http://central.maven.org/maven2/mysql/mysql-connector-java/5.1.38/mysql-connector-java-5.1.38.jar
```

##### 配置tomcat内存大小

默认的tomcat内存比较小，运行xwiki有的时候回卡死在启动界面，在**tomcat/bin/**下修改** Catalina.sh**,在`cygwin=false`上面添加如下代码
```bash
JAVA_OPTS="-Xms256m -Xmx512m -Xss1024K -XX:PermSize=128m -XX:MaxPermSize=256m"
```
在windows下也是一样。如果没有意外情况的话，到此，xwiki的搭建就成功了。启动tomcat，打开xxx/xwiki,出现如下页面，就意味着你配置成功了
![第一次登陆界面](http://upload-images.jianshu.io/upload_images/3167229-3508d9319ba2904e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)直接next就ok进入主页面（Main）你会发现是一片空白页
![默认主页](http://upload-images.jianshu.io/upload_images/3167229-d34d2f21a9461768?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)点击右上角登录，使用superadmin，system登录
![导入配置页面](http://upload-images.jianshu.io/upload_images/3167229-16dc4e8d4e282acb?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)这个时候就要上传开头提到的ui配置包，点击界面上的链接，下载`org.xwiki.platform_xwiki-platform-administration-ui-8.2.1.xar`然后上传，点击安装就如下图的loding![这里写图片描述](http://upload-images.jianshu.io/upload_images/3167229-ae81d326d9bd29c4?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)稍等片刻如下![这里写图片描述](http://upload-images.jianshu.io/upload_images/3167229-26a308bd0f398184?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)这个时候刷新下页面，你会发现变成了这样![这里写图片描述](http://upload-images.jianshu.io/upload_images/3167229-a7f47f3892e258ab?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)在这里你可以配置用户组，用户，文章的语法，模板等等...

### 最后

*当团队在开发的过程中，遇到一些常见的问题、一些学习到的新技能，项目里用到的组件都可以在此上面分享，xwiki也有导入导出功能，方便资料的移植和维护*。
最最后have fun~~~

[CSDN传送门](http://blog.csdn.net/qqhjqs/article/details/52510587)
