---
title: Docker构建服务之部署和备份jekyll网站
copyright: true
permalink: 1
top: 0
date: 2019-01-15 21:45:18
tags:
    - Docker
    - 运维
    - 自动化
categories:
    - 技术
password:
---



来自《第一本Docker书》，我觉得很有趣，就记录一下

<!--more-->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=1320138&auto=1&height=66"></iframe>

## 准备国内ubuntu镜像
每次构建Ubuntu容器然后安装软件的时候，都异常的卡，那是因为没有使用国内镜像，所以我事先准备了sources.list文件，一定要确定对应的ubuntu的版本号，我用的是18.04，内容如下
```
vi sources.list
```
输入以下内容
```
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

## 创建对应的Dockerfile

### jekyll
```
mkdir jekyll
cd jekyll
vi Dockerfile
```
输入如下内容
```
FROM ubuntu:18.04
LABEL maintainer="vector4wang@qq.com"
ENV REFRESHED_AT 2019-01-14
## 更换镜像
RUN rm -rf /etc/apt/sources.list
ADD sources.list /etc/apt/

RUN apt-get -qq update
RUN apt-get -qq install ruby ruby-dev libffi-dev build-essential nodejs
RUN gem install --no-rdoc --no-ri jekyll -v 2.5.3

VOLUME /data
VOLUME /var/www/html
WORKDIR /data

ENTRYPOINT [ "jekyll", "build", "--destination=/var/www/html" ]
```
最后一句命令的意思就是每次启动的时候就将/data下的源文件编译成可发布的网站内容，并放在/var/www/html中供下面的apache使用

### apache
```
mkdir apache
cd apache
vi Dockerfile
```
输入以下内容
```
FROM ubuntu:18.04
LABEL maintainer="vector4wang@qq.com"

## 更换镜像
RUN rm -rf /etc/apt/sources.list
ADD sources.list /etc/apt/

RUN apt-get -qq update
RUN apt-get -qq install apache2

VOLUME [ "/var/www/html" ]
WORKDIR /var/www/html

ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid
ENV APACHE_RUN_DIR /var/run/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2

RUN mkdir -p $APACHE_RUN_DIR $APACHE_LOCK_DIR $APACHE_LOG_DIR

EXPOSE 80

ENTRYPOINT [ "/usr/sbin/apachectl" ]
CMD ["-D", "FOREGROUND"]

```

最终的目录结构为：
```
.
├── apache
│   ├── Dockerfile
│   └── sources.list
├── jekyll
    ├── Dockerfile
    └── sources.list
```

## 构建
分别构建 jekyll 和 apache
```
cd jekyll
docker build -t vector/jekyll .

cd apache
docker build -t vector/apache .
```

注意：一定不要忘记更换容器的镜像源...

执行`docker images`
[![2019-01-14 21-28-56.png](http://upload-images.jianshu.io/upload_images/3167229-1d1f25a9607b2b41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://i.loli.net/2019/01/14/5c3c8ea882065.png)

## 启动服务
### jekyll源文件
创建一个目录
```
mkdir jekyll-src
cd jekyll-src
```
从github上下载一个jekyll模板代码
```
git clone https://github.com/turnbullpress/james_blog.git
cd james_blog
```

### 启动jekyll
```
docker run -v /Users/wangxc/Develop/docker/jekyll-src/james_blog:/data/ --name vector_blog vector/jekyll
```
结果为
```
Configuration file: none
            Source: /data
       Destination: /var/www/html
      Generating...
                    done.
```

>“卷是在一个或多个容器中特殊指定的目录，卷会绕过联合文件系统，为持久化数据和共享数据提供几个有用的特性。
卷可以在容器间共享和重用。
共享卷时不一定要运行相应的容器。
对卷的修改会直接在卷上反映出来。
更新镜像时不会包含对卷的修改。
卷会一直存在，直到没有容器使用它们。
利用卷，可以在不用提交镜像修改的情况下，向镜像里加入数据（如源代码、数据或者其他内容），并且可以在容器间共享这些数据。”
**摘录来自: [澳] 詹姆斯·特恩布尔（James Turnbull）. “第一本Docker书（修订版）。” iBooks.** 

### 启动apache
```
docker run -d -P --volumes-from vector_blog vector/apache
```

>该 --volumes-from把指定容器里的所有卷都加入新创建的容器里。这意味着，Apache容器可以访问之前创建的james_blog容器里/var/www/html卷中存放的编译后的Jekyll网站。即便james_blog容器没有运行，Apache容器也可以访问这个卷
**摘录来自: [澳] 詹姆斯·特恩布尔（James Turnbull）. “第一本Docker书（修订版）。” iBooks. **

此时apache这个容器可以访问jekyll容器里的所有卷,我们进入apache内容看一下
```
docker exec -ti bdd9df87c189 /bin/bash
```
进入对应的目录可看到jekyll中的卷
```
/var/www/html
/data
```

查看宿主机与容器的端口映射情况
```
docker ps 
或
docker port e539ff7ed7e8 80
```
得到0.0.0.0:32768，然后宿主机访问`localhost:32768`

[![微信截图_20190115092234.png](http://upload-images.jianshu.io/upload_images/3167229-308e26f83f1246a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://i.loli.net/2019/01/15/5c3d366950ba8.png)


### 编辑内容

进入宿主机的jekyll模板源代码中，对_config.yml进行相关的修改，比如修改title为自己的名字或者其他的内容，保存后退出，然后执行

```
docker start vector_blog
```

通过查看日志`docker logs vector_blog`可以看到
```
Configuration file: /data/_config.yml
            Source: /data
       Destination: /var/www/html
      Generating... 
                    done.
 Auto-regeneration: disabled. Use --watch to enable.
```
 
已经跟新，这个时候，我们刷新下页面`localhost:32768`就可以看到最新的内容了，是不是很有趣
 
## 备份
 
 这里提供两种思路吧，
 第一种：我自己用的是hexo，一般都是直接备份在github上，jekyll也一样，保存在github上是很容易很方便的；
 
 第二种就是书上说的直接对卷进行备份
 ```
 run --rm --volumes-from vector_blog -v $(pwd):/backup ubuntu:18.04 \
 tar cvf /backup/vector_blog_backup.tar /var/www/html
 ```

> 指定了--rm标志，这个标志对于只用一次的容器，或者说用完即扔的容器，很有用。这个标志会在容器的进程运行完毕后，自动删除容器。对于只用一次的容器来说，这是一种很方便的清理方法。


个人感觉还是备份到git上方便。

