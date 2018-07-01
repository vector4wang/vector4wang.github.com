---
title: 通过shell脚本和企业微信实现实时报警功能
date: 2018-03-11 23:06:24
categories: 技术
tags:
	- 监控
	- shell
	- 微信
---

工作中，我们会有一些应用跑在线上服务器，那么这些应用出现问题，如内存、CPU超过阈值之后我们必须要在第一时间知道，第一时间处理这些问题，尽可能的让用户感受不到应用的异常。

<!--more-->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=211532&auto=1&height=66"></iframe>

有的公司有运维人员，那么恭喜了，运维自己应该会有一套监控体系，作为开发者就可以专心的攻克业务逻辑；但是有的公司可能没有，那么应用的状态就需要我们开发者来监控了。


关于监控，有发送邮件的、有搭建Zabbix的、也有通过企业微信的等等；我毫不犹豫的选择了微信，简单方便，我邮箱天天有人发送垃圾邮件，直接屏蔽了，zabbix（音同 zæbix）是一个基于WEB界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案，太重了，先不玩它。


下面就介绍下如何实现微信报警的

### 注册微信企业号
本以为企业微信号需要企业的相关信息之类的，没想到什么都不要，只需要一个微信号，建议大家看到的这里的都去注册一个吧~[注册地址](https://work.weixin.qq.com/wework_admin/register_wx?from=loginpage)

注册完之后再创建一个应用，填上一些基本信息之后就可以创建成功了,部分界面如下
[![1.png](http://upload-images.jianshu.io/upload_images/3167229-3217d5ad5aba5f7f..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://i.loli.net/2018/03/11/5aa53ba6ec611.png)
[![2.png](http://upload-images.jianshu.io/upload_images/3167229-441175cc491c91f2..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://i.loli.net/2018/03/11/5aa53ba6f0d2f.png)

### shell 脚本
本来打算用python写的，后来想了一下，不行！用任何语言写都不如用shell，因为linux自身就支持shell，不需要安装任何环境，这一点是最重要的！

于是乎先分析了下企业微信的开发API文档
[官方文档](https://work.weixin.qq.com/api/doc#11279)
[在线调试](http://work.weixin.qq.com/api/devtools/devtool.php)
关于API，这里注意一点
**access_token 有一个过期时间，2小时。不要频繁的去获取token，获取一次保存起来即可。**

关于shell，实在不敢恭维，大概花了3个小时才搞定，看来以后要多学学shell编程了。

注意的地方

- shell中对json的处理 可以参考此[博客](https://www.ibm.com/developerworks/cn/linux/1612_chengg_jq/index.html)
- shell中curl的参数使用
- if else 的使用
- shell将数据写入文件

最终的效果如下：
[![wma.gif](http://upload-images.jianshu.io/upload_images/3167229-34b4a82b1a1fd9f8..gif?imageMogr2/auto-orient/strip)](https://i.loli.net/2018/03/11/5aa54264eb724.gif)

这里只是把shell发送报警信息这个流程打通了 ，至于如何去监控应用状态、服务器的状态等等，我会在后面去实现，敬请关注
脚本我放在了[github](https://github.com/vector4wang/shell-tools/blob/master/wx-monitor-alarm.sh)上，欢迎提PR

CSDN：http://blog.csdn.net/qqhjqs?viewmode=list 
博客：http://vector4wang.tk/ 
简书：https://www.jianshu.com/u/223a1314e818 
Github:https://github.com/vector4wang 
Gitee:https://gitee.com/backwxc
