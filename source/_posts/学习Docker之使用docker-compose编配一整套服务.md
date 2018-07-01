---
title: 学习Docker之使用docker-compose编配一整套服务
date: 2018-04-14 11:18:56
categories:
	- 技术
tags:
	- Docker
	- 自动化部署
	- compose
---
使用docker compose可以**一键**完成“一整套”服务的搭建也可以完成服务集群化部署。

<!--more-->


![docker-compose.png](https://upload-images.jianshu.io/upload_images/3167229-a722448accf9dfec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


　这两天在用python写一个爬虫，数据暂时保存在本地的mongodb数据库，到部署的时候，发现线上的服务器没有python环境和mongodb，这个时候立马就想到了Docker！
　最初的思路就是run一个mongodb容器，然后再把爬虫程序构建为镜像并run起来。准备动手的时候突然脑海里闪过compose这个东东，之前看docker书的时候正好看到有关compose的这一章，我花了十分钟简单的过了一下，发现使用compose可以更加完美的实现一键构建、部署与启动的过程，接下来就以python与mongodb组合为例

>官网使用的是python与redis https://docs.docker.com/compose/gettingstarted/

先看一下python程序
```python
from flask import Flask
from pymongo import MongoClient
import random
app = Flask(__name__)
client = MongoClient('mongodb')
db=client['datas']

@app.route('/')
def hello():
    db.col.insert({"hits":random.random()})
    return 'Hello World! I have been seen %s times.' % (db.col.count())

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)

```
使用flask做python的web服务框架，每请求一次就会在mongodb的datas数据库插入一个文档，然后返回datas下面的总数，非常简单。

然后需要把python所依赖的模块抽取出来，这里推荐使用`pipreqs`
安装pipreqs
```bash
pip install pipreqs
```
然后执行脚本
```bash
# 我直接在项目的根目录下执行，当然也可以带上路径 如 pipreqs /project/path
pipreqs . 
```
生成的requirements如下
```txt
pymongo==3.6.1
Flask==0.12.2
```
接下来需要编写Dockerfile
```Dockerfile
FROM python:2.7
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python","app.py"]
```
这里用的是python 2.7版本

然后定位到code目录下;

将requirements.txt下依赖的模块一并安装;

最后执行app.py

如果是第一次接触Docker那就以往文章了解下~
[学习Docker之Dockerfile的命令](https://www.jianshu.com/p/10ed530766af)
[学习Docker之10张图带你深入理解Docker容器和镜像](https://www.jianshu.com/p/e7cd0352e0de)
[学习Docker之Docker、容器和镜像的简介和常用命令](https://www.jianshu.com/p/a60045ea96be)
[学习Docker之Docker初体验---SpringBoot集成Docker的部署、发布与应用](https://www.jianshu.com/p/efd70ad53602)

紧接着开始docker-compose yml 命令与写法跟Dockerfile类似，很容易理解，如下：
```yml
version: '2'
services:
    web:
        build: .
        ports:
            - "5000:5000"
    mongodb:
        image: mongo
        ports:
            - "27017:27017"   
```
**这里要注意一下mongodb，就是python中使用的‘域名’**

可以这样理解，web服务和mongodb服务都在同一个局域网，然后mongodb服务的ip对应域名就是“mongodb”

docker-compose 一般需要独自安装，我这里使用的是ubuntu，直接使用`apt install docker-compose`,当然也可以按照官网安装

最终目录为
```bash
.
├── app.py
├── docker-compose.yml
├── Dockerfile
└── requirements.txt
```
执行命令`docker-compose up`

经过漫长的构建，docker会自动将其启动，整个过程如下
[![wma.gif](http://upload-images.jianshu.io/upload_images/3167229-652c3cc69b676024.gif?imageMogr2/auto-orient/strip)](https://i.loli.net/2018/04/14/5ad1699763fb0.gif)


到这里使用docker-compose编配一个web服务和一个数据服务就到此结束了!

使用compose我们可以把一整套的项目包括应用、数据存储、消息中间件等等的安装、部署与启动整合在一个yml配置中，真的可以达到一键启动应用！！！

CSDN：http://blog.csdn.net/qqhjqs?viewmode=list 
博客：http://vector4wang.tk/ 
简书：https://www.jianshu.com/u/223a1314e818 
Github:https://github.com/vector4wang 
Gitee:https://gitee.com/backwxc 
如果感觉有帮助的话，点个赞哦~