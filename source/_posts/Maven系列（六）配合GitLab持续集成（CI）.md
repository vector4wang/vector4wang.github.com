---
title: Maven系列（六）配合GitLab持续集成（CI）
date: 2017-04-24 22:31:16
categories: 技术
tags:
  - Maven
  - 项目管理
  - GitLab
  - CI
---

Maven相关的内容
<!--more-->


### 前言
想要完成接下来的操作，你要做的准备工作有以下几点：
- [Maven系列（一）Maven的简介与使用](http://blog.csdn.net/qqhjqs/article/details/46963495)
- [Maven系列（二）无Maven不项目---使用Eclipse快速搭建Maven项目 ](http://blog.csdn.net/qqhjqs/article/details/47045585)
- [Maven系列（三）Maven给不同的环境打包 ](http://blog.csdn.net/qqhjqs/article/details/53495535)
- [Maven系列（四）Maven热部署 ](http://blog.csdn.net/qqhjqs/article/details/51594583 )
- [Maven系列（五）CentOS7搭建最新GitLab ](http://blog.csdn.net/qqHJQS/article/details/52950147)

如果你没有接触过Maven，没关系，看看上面的五点，相信会让你对Maven有一个稍微深入的了解，并会让你迅速掌握其高大上的用法。
### 本节所需
先来看一下这张图：
![传统与CI](http://upload-images.jianshu.io/upload_images/3167229-eb185719b2acecf3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
传统的热部署是图中虚线所指，通过本地执行`mvn tomcat:deploy` 来将项目部署到Server1上，实线所指是借助GitLab的CI来部署，比较起来感觉没什么两样，但是GitLab在这里除了作为一个代码托管平台之外，还担当着自动执行预编写脚本的功能（下面有介绍）。
这里需要两台Server，我用两个虚拟机来代替。
### 安装GitLabRunner
可以再任意一台server安装
```bash
## 添加GitLab官方源
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash
## 安装
sudo yum install gitlab-ci-multi-runner
```
这里要在管理员的权限下执行~，过程可能有点慢。
安装完毕后，执行`gitlab-ci-multi-runner` 会有命令帮助提示，如下：
```bash
[root@localhost ~]# gitlab-ci-multi-runner
NAME:
   gitlab-ci-multi-runner - a GitLab Runner

USAGE:
   gitlab-ci-multi-runner [global options] command [command options] [arguments...]
   
VERSION:
   1.8.1 (a2efdd4)
   
AUTHOR(S):
   Kamil Trzciński <ayufan@ayufan.eu> 
   
COMMANDS:
   exec                 execute a build locally
   list                 List all configured runners
   run                  run multi runner service
   register             register a new runner
   install              install service
   uninstall            uninstall service
   start                start service
   stop                 stop service
   restart              restart service
   status               get status of a service
   run-single           start single runner
   unregister           unregister specific runner
   verify               verify all registered runners
   artifacts-downloader download and extract build artifacts (internal)
   artifacts-uploader   create and upload build artifacts (internal)
   cache-archiver       create and upload cache artifacts (internal)
   cache-extractor      download and extract cache artifacts (internal)
   help, h              Shows a list of commands or help for one command
   
GLOBAL OPTIONS:
   --debug                      debug mode [$DEBUG]
   --log-level, -l "info"       Log level (options: debug, info, warn, error, fatal, panic)
   --cpuprofile                 write cpu profile to file [$CPU_PROFILE]
   --help, -h                   show help
   --version, -v                print the version
```
### 注册Runner
Runner的成功安装预示着我们现在拥有了一台发动机，下面要将“发动机”装在“机器”（GitLab）上。
执行`gitlab-ci-multi-runner register` 开始注册
```bash
[root@localhost ~]# gitlab-ci-multi-runner register
Running in system-mode.                            
 ## gitlab服务器的域名                                                  
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://192.168.92.128
## runnerToken，下面图片有位置
Please enter the gitlab-ci token for this runner:
3kjzGzK4PZDA73HYPHrP
## 描述
Please enter the gitlab-ci description for this runner:
[localhost.localdomain]: 简书·test
Please enter the gitlab-ci tags for this runner (comma separated):
js
Registering runner... succeeded                     runner=3kjzGzK4
## 执行容器，我用的shell
Please enter the executor: virtualbox, docker-ssh+machine, docker, docker-ssh, parallels, shell, ssh, docker+machine, kubernetes:
shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```

![新版GitLab](http://upload-images.jianshu.io/upload_images/3167229-528d390e6b49b16e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
安装成功之后，gitlab的页面会多出一个
![Runner](http://upload-images.jianshu.io/upload_images/3167229-02eeafb56fb336d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 添加项目
点击Edit进入详细页面
![详细页面](http://upload-images.jianshu.io/upload_images/3167229-52817be2fdb4bb59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们要让“发动机”干活，就选中项目后面的enable，让项目的CI在这个Runner运行。
### 编写CI脚本
我们要在项目的根目录下添加一个文件`.gitlab-ci.yml` ,鬼知道怎么会是酱紫的文件名。
```bash
build:
    script: "pwd && mvn tomcat:deploy"
```
这里写的比较简单，直接热部署。根据项目需要，我们可以写更复杂的脚本，比如自动将打包的项目复制到指定位置，然后解压啦、配置服务器相关环境啦，等等等，这些统统交给CI来做~，是不是很方便啊！

项目提交上去后自动会跑CI脚本。有执行的记录和详细信息

![集成记录](http://upload-images.jianshu.io/upload_images/3167229-0ad768a679bbc709.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 可能会遇到的问题
安装Runner的服务器要安装Maven
maven安装。

    下载maven文件，官网下载对应版本。http://maven.apache.org/download.cgi；
    解压下载的包，将maven目录移动到自定义目录下，建议移动到/opt公共目录下，以便任何用户能够访问到该目录，例如/opt/apache-maven-3.3；
    配置maven环境变量，修改/etc/profile，在文件末尾增加export M2_HOME=/opt/apache-maven-3.3（你的maven所在目录）， 将该目录的bin目录添加到环境变量PATH中，export PATH=$PATH:$M2_HOME/bin;
    使配置生效，source /etc/profile;

    测试maven安装成功，mvn -v，出现配置信息如下则说明maven配置成功：

    Apache Maven 3.3.3 (7994120775791599e205a5524ec3e0dfe41d4a06; 2015-04-22T19:57:37+08:00)
    Maven home: /Users/turinblueice/Downloads/apache-maven-3.3.3
    Java version: 1.8.0_40, vendor: Oracle Corporation
    Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre
    Default locale: zh_CN, platform encoding: UTF-8
    OS name: "mac os x", version: "10.10.5", arch: "x86_64", family: "mac"

### 说在后面
这篇应该是昨天晚上放在CSDN的，前前后后搞了半个多小时，后来在看其他页面的时候，一不小心给退出了，然后当时就懵逼了，真是有苦说不出！！！CSDN及时保存的功能没有简书做的好，所以我决定，以后再简书上写完，然后在复制过去~~~
