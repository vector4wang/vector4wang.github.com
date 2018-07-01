---
title: CSDN博客迁移至Hexo之Html2Md服务(专为csdn打造)
date: 2017-07-23 16:25:03
categories: 技术
tags:
	- csdn
	- html2md
	- 服务
---

此篇介绍下html2md服务，我将上篇遗留的问题---csdn中的代码高亮转换失败的问题修复了下，结果还算满意，自己搭了服务，大家可以试一试，玩一玩，有问题可以私信我~

<!--more-->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=480768678&auto=1&height=66"></iframe>

接着上篇[CSDN博客迁移至Hexo之同步CSDN博文到本地MD文件](http://www.jianshu.com/p/02ef1a98bf56)
此篇介绍下html2md服务，我将上篇遗留的问题---csdn中的代码高亮转换失败的问题修复了下，结果还算满意，自己搭了服务，大家可以试一试，玩一玩，有问题可以私信我~
地址为：http://60.205.191.82:8002/tool/html2md
界面：

![view.png](http://upload-images.jianshu.io/upload_images/3167229-fae059d5a47e003b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
很简洁支持两种输入
- 博文的链接，如http://blog.csdn.net/qqhjqs/article/details/75208466
- 直接输入html源码，如
```html
<h1>haha</h1>
```

效果大家可以自行测试

当然了，结果肯定不是十全十美的，只能说差强人意，html结构千变万化，有很多种情况你都需要去考虑然后去针对性的编写代码。所以也不要过分依赖html2md，网上有其他人提供了工具，我也都试了试，共同点是，针对简单的html代码，转换的很完美，但是稍复杂一些的，多多少少都会有一些转换失败的结果。

