---
title: 学习Docker之Docker、容器和镜像的简介和常用命令
date: 2018-01-22 00:02:38
categories: 技术
tags:
	- Docker
---

Docker、容器和镜像的简介和常用命令

<!--more-->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=1167022&auto=1&height=66"></iframe>

第一篇通过部署SpringBoot项目来见识到了Docker的强大[点我](http://vector4wang.tk/2018/01/18/%E5%AD%A6%E4%B9%A0Docker%E4%B9%8BDocker%E5%88%9D%E4%BD%93%E9%AA%8C-SpringBoot%E9%9B%86%E6%88%90Docker%E7%9A%84%E9%83%A8%E7%BD%B2%E3%80%81%E5%8F%91%E5%B8%83%E4%B8%8E%E5%BA%94%E7%94%A8/)，接下来就来简单聊聊什么是Docker?

- 什么是Docker
- 镜像与容器
- Docker常用命令

## 问题

- 小王在工作之余接了个私单，功能很简单，开发完之后只要在本地将程序跑起来，就能完成客户的需求。于是小王每天晚上花个一两个小时去开发，一周之内搞定了，然而在远程给客户部署的时候，出现了各种各样的问题，光配置环境就花了两三个小时，好不容易搞定了一台机器，客户说“辛苦了，还有十几台要帮我安装一下”，小王听完差点一口老血喷了出来。

- 工作上小王开发的服务要部署在各个环境上，有的环境还不止一两个节点，一些环境的配置反反复复的去做花费了小王好长的时间，真的是开发十分钟，配置两小时啊。

这些问题看完下面相信你心中就会有解决的办法了。

## 什么是Docker

> *Docker是一个能够把开发的应用程序自动部署到容器的开源引擎。由Docker公司的团队编写，基于Apache2.0开源授权协议发行
Docker在虚拟化的容器执行环境中增加了一个应用程序部署引擎。改引擎的目标就是提供一个轻量、快速的环境，能够运行开发者的程序，并方便高效地将程序从开发者的笔记本部署到测试环境，然后再部署到生产环境。Docker及其简洁，它所需的全部环境只是一台仅仅安装了兼容版本的Linux内核和二进制文件最小限制的宿主机。*
摘自《THE DOCKER BOOK》

简单的说，程序员只要把程序开发好，然后通过Docker就可以很简单很快速的将服务部署在任何一个安装了Docker的机器上。这里引入了容器的概念，Docker可以帮用户构建和部署容器，用户只需要把自己的应用程序或服务打包放进容器即可。
[![可爱的Docker.png](https://i.loli.net/2018/01/21/5a649b380cd7d.png)](https://i.loli.net/2018/01/21/5a649b380cd7d.png)

Docker借鉴了标准集装箱的概念。标准集装箱将货物运往世界各地，Docker将这个模型运用到自己的设计哲学中，唯一不同的是：*集装箱运输货物，而Docker运输软件*。
每个容器都包含一个软件镜像，也就是说容器的“货物”，而且与真正的货物一样，容器里的软件镜像可以进行一些操作。例如：镜像可以被创建、启动、关闭、重启以及销毁。
和集装箱一样，Docker在执行上述操作时，并不关心容器里塞进了是么，它不管里面是Web服务器，还是数据库，或者是应用服务器是么的。所有容器都按照相同的方式将内容“装载”进去。
Docker也不关心用户要把容器运到何方：用户可以在自己的笔记本中构建容器，上传到Registry,然后下载一个物理的或虚拟的服务器来测试。像标准集装箱一样，Docker容器方便替换，可以叠加，易于分发，并且尽量通用。(内容都在《THE DOCKER BOOK》上)

## 镜像与容器
[10张图带你深入理解Docker容器和镜像](http://dockone.io/article/783)
网上介绍镜像与容器的文章很多，每个人都有自己的一套方式去理解，我的理解如下：
> 容器好比是快递箱(集装箱)，镜像好比是集装箱里的货物(实物)。不同的货物在装配的时候所需要的填充物不同，如电子设备可能需要大量的泡沫纸、生鲜需要冰袋等等。这个时候Docker 就好比快递公司,Registry好比某购物平台。假如你想要某个实物(镜像)，快递公司会自动将将实物(镜像)打包好送到你的手里，开箱即用。你也可以自己制作实物(镜像)，然后指明这个镜像所需要的一些环境等配置，再一并提交给某购物平台(Registry)，方便他人使用。

注意：这里只为方便的去理解容器与镜像，可能不同层次理解是不一样的。当然了深层次的容器和镜像并不是这样子的，后续的文章会继续介绍。

接下来就举两个例子

- 通过已有镜像来启动
之前自己开发了一个服务放在了DockerHub上[quick-docker](https://hub.docker.com/r/vector4wang/quick-docker/),这个是已知的，我们接下来直接在docker上运行这个启动它
[![已知镜像运行.png](https://i.loli.net/2018/01/21/5a64ad97e2110.png)](https://i.loli.net/2018/01/21/5a64ad97e2110.png)
如上图，直接运行一个镜像，docker会将镜像pull到本地，然后按照镜像所需要的环境去创建容器，然后去启动。

- 自己创建镜像并提交
这里就不在赘述，可参见上一篇博客[Docker初体验](http://vector4wang.tk/2018/01/18/%E5%AD%A6%E4%B9%A0Docker%E4%B9%8BDocker%E5%88%9D%E4%BD%93%E9%AA%8C-SpringBoot%E9%9B%86%E6%88%90Docker%E7%9A%84%E9%83%A8%E7%BD%B2%E3%80%81%E5%8F%91%E5%B8%83%E4%B8%8E%E5%BA%94%E7%94%A8/)，里面有详细的步骤。

> 注意：以上基于个人理解，只是便于去理解，容器和镜像的概念远不如此，并且容器与镜像比较重要，希望看到这里的客观多留意一下，多参看官方的文档。

## Docker常用命令
### images
- 搜索image  
`docker search image_name`  
- 下载image  
`docker pull image_name`  
- 列出镜像列表
`docker images` 可加参数如下
-a, --all=false Show all images;
--no-trunc=false Don't truncate output; 
-q, --quiet=false Only show numeric IDs 

- 删除images，删除images，通过image的id来指定删除谁
`docker rmi <image id>`
- 删除images id 为none的
`docker rmi $(docker images | grep "^<none>" | awk "{print $3}")`
- 删除全部image
`docker rmi $(docker images -q)`
- 显示一个镜像的历史
`docker history image_name` 可加参数
--no-trunc=false Don't truncate output;
-q, --quiet=false Only show numeric IDs  

### container
- 列出当前所有正在运行的container  
`docker ps`  
- 列出所有的container  
`docker ps -a`  
- 列出最近一次启动的container  
`docker ps -l`
- 停止所有的container，这样才能够删除其中的images：
`docker stop $(docker ps -a -q)`
- 删除所有container：
`docker rm $(docker ps -a -q)`

更多命令参见:[菜鸟教程](http://www.runoob.com/docker/docker-command-manual.html)


## 最后
Docker入门还是很简单的，本文简单的做了Docker的介绍、容器与镜像的相关内容还有常用的命令，希望对你有所帮助。




CSDN：http://blog.csdn.net/qqhjqs?viewmode=list 
博客：http://vector4wang.tk/ 
简书：https://www.jianshu.com/u/223a1314e818 
Github:https://github.com/vector4wang 
Gitee:https://gitee.com/backwxc 
如果感觉有帮助的话，点个赞哦~