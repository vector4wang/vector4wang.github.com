---
title: 使用Bootstrap的后台猿人可能会用到的功能
date: 2017-04-24 22:31:16
categories: 技术
tags:
  - Bootstrap
---

使用bootstrap制作的门户网站
<!--more-->
### 写在前面
本人主要做服务器端这块，对前端页面了解不是很深，最近公司要求做一个门户网站，我就使用bootstrap样式框架来写，我把中间用到的小模块收集起来做个记录，下次直接复制使用
**注：以下基于bootstrap3.1.1**
### 引入样式
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
		<title></title>
		<meta charset="utf-8">
		<meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no" />
		<link href="//cdn.bootcss.com/bootstrap/3.1.1/css/bootstrap.min.css" rel="stylesheet">
		<script src="//cdn.bootcss.com/jquery/2.1.1-rc2/jquery.min.js"></script>
		<script src="//cdn.bootcss.com/bootstrap/3.1.1/js/bootstrap.min.js"></script>
	</head>
	<body>
	</body>
</html>
```

###水平导航
```html
<nav class="navbar navbar-default navbar-fixed-top">
	<div class="container">
		<div class="navbar-header">
			<button class="navbar-toggle collapsed" type="button" data-toggle="collapse" data-target="#navbar">
				<span class="sr-only">Toggle navigation</span>
				<span class="icon-bar"></span>
				<span class="icon-bar"></span>
				<span class="icon-bar"></span>
			</button>
			<a class="navbar-brand" href="#">LOGO</a>
		</div>
		<div id="navbar" class="navbar-collapse collapse">
			<ul class="nav navbar-nav">
				<li class="active">
					<a href="home.html">首    页</a>
				</li>
				<li>
					<a href="#" data-toggle="modal" data-target="#myModal">下载</a>
				</li>
				<li>
					<a href="investment.html">投资顾问</a>
				</li>
				<li>
					<a href="help.html">帮助</a>
				</li>
				<li>
					<a href="about.html">关于</a>
				</li>
			</ul>
			<!--
				应该能看懂navbar-right的用法吧
            -->
			<form class="navbar-form navbar-right">
				<a class="btn" id='btnCtrl' href="#">登录</a>
				<a class="btn btn-primary" href="#">注册</a>
			</form>
		</div>
	</div>
</nav>
```

![水平导航](http://upload-images.jianshu.io/upload_images/3167229-caaf99b4bb097f4f.gif?imageMogr2/auto-orient/strip)

###bootstrap轮播
```html
<div id="myCarousel" class="carousel slide">
			<!-- 轮播（Carousel）指标 -->
			<ol class="carousel-indicators">
				<li data-target="#myCarousel" data-slide-to="0" class="active"></li>
				<li data-target="#myCarousel" data-slide-to="1"></li>
				<li data-target="#myCarousel" data-slide-to="2"></li>
				<li data-target="#myCarousel" data-slide-to="3"></li>
			</ol>
			<!-- 轮播（Carousel）项目 -->

			<div class="carousel-inner" style="position: relative;">
				<div class="item active">
					![](img/mm_02.jpg)
				</div>
				<div class="item">
					![](img/mm_02.jpg)
				</div>
				<div class="item">
					![](img/mm_02.jpg)
				</div>
				<div class="item">
					![](img/mm_02.jpg)
				</div>
			</div>
			<!-- 轮播（Carousel）导航 -->
			<a class="carousel-control left" href="#myCarousel" data-slide="prev">&lsaquo;</a>
			<a class="carousel-control right" href="#myCarousel" data-slide="next">&rsaquo;</a>
		</div>
		<script>
			$('#myCarousel').carousel({
				interval: 5000
			});
			$('#myCarousel').on('slide.bs.carousel', function() {
				console.log('start');
			});
			$('#myCarousel').on('slid.bs.carousel', function() {
				console.log('end');
			});
		</script>
```

![轮播效果](http://upload-images.jianshu.io/upload_images/3167229-13d9ff0c4e5c293a.gif?imageMogr2/auto-orient/strip)


###图片自适应
页头加上
`<metaname="viewport"content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no"/>`
图片加上`img-responsive`类名

###清除浮动
`<div class="clearfix" style="margin-top:49px;"></div>`

###让div 居中
```css
.help-box{
	width:65%;
	height:100px;
	border:1pxsolid#2A70C8;
	border-radius:10px;
	margin-bottom:10px;
	/*width:auto;*/
	margin-left:auto;
	margin-right:auto;
}
```
###点击文字折叠
```html
<div class="panel panel-default">
						<div class="panel-heading">
							<h4 class="panel-title">
								<a data-toggle="collapse" data-parent="#accordion" 
								   href="#collapse1">
									标题一
								</a>
							</h4>
						</div>
						<div id="collapse1" class="panel-collapse collapse in">
							<div class="panel-body">
								内容一
							</div>
						</div>
					</div>
					<div class="panel panel-default">
						<div class="panel-heading">
							<h4 class="panel-title">
								<a data-toggle="collapse" data-parent="#accordion" 
								   href="#collapse2">
									标题二
								</a>
							</h4>
						</div>
						<div id="collapse2" class="panel-collapse collapse">
							<div class="panel-body">
								内容二
							</div>
						</div>
					</div>
```

![文字折叠面板](http://upload-images.jianshu.io/upload_images/3167229-3a6069bde43f5364.gif?imageMogr2/auto-orient/strip)

###左侧折叠菜单
```html
<div  class="container-fluid">
				<div class="row">
				<div class="col-lg-2">
					<ul id="main-nav" class="main-nav nav nav-tabs nav-stacked text-center
						" style="">
						<li>
							<a href="#applyAccount" class="nav-header" data-toggle="collapse">
								一级菜单
							</a>
							<ul id="applyAccount" class="nav nav-list secondmenu collapse" style="">
								<li>
									<a href="#1_1">    二级菜单</a>
								</li>
								<li>
									<a href="#1_2">    二级菜单</a>
								</li>
							</ul>
						</li>
						<li>
							<a href="#" class="nav-header" data-toggle="collapse">
								一级菜单
							</a>
						</li>
					</ul>
				</div>
				<div class="col-lg-10">
					右侧内容
				</div>
			</div>
</div>
```

![左侧菜单效果](http://upload-images.jianshu.io/upload_images/3167229-760e61ac530d8d54.gif?imageMogr2/auto-orient/strip)

###返回顶部
这里使用的是jquery的一个插件，引入js，只需几句js代码，很好用`jquery.goup.min.js`
```javascript 
$.goup({
	trigger:100,
	bottomOffset:150,
	locationOffset:100,
	title:'返回顶部',
	titleAsText:true
});
```
### 鼠标进入div 颜色改变
```html
<!--第一种-->
<div onmouseover="this.style.backgroundColor='#F4F9FD'"  onmouseout="this.style.backgroundColor='#FFFFFF'"></div >   
<!--第二种-->
<style> 
.d_over{
	background-color:#307172;
}  
.d_out{
	background-color:#EFEFEF;
}  
</style>  
<div class="d_out" onmouseover="this. className='d_over'"  onmouseout="this. className='d_out'">div 背景颜色</div >  
```
### css3 过渡 div 变大
```css
* {
	transition:All 0.4s ease-in-out;
	-webkit-transition:All 0.4s ease-in-out;
	-moz-transition:All 0.4s ease-in-out;
	-o-transition:All 0.4s ease-in-out;
}
*:hover {
	transform:scale(1.2);
	-webkit-transform:scale(1.2);
	-moz-transform:scale(1.2);
	-o-transform:scale(1.2);
	-ms-transform:scale(1.2);
}

```
### div  onclick 新的窗口打开页面
```html
<div  onclick="window.opemn('url')"></div >
```

### 回车键 触发方法
```javascript 
document.onkeypress = function(event) {
	if(event.keyCode == 13) {
		console.log(123)
		return false;
	}
}
```

### 验证码倒计时
```javascript 
var wait = 60;
function timer() {
	console.log("timer");
	var codediv  = $('#codediv ');
	if(wait == 0) {
		codediv .text('获取验证码');
		codediv .removeAttr('disabled');
		wait = 60;
	} else {
		codediv .attr('disabled','disabled');
		codediv .text(wait + "秒后可重新发送");
		wait--;
		setTimeout(function() {
				timer();
			},
			1000)
	}
}
```
![验证码倒计时效果](http://upload-images.jianshu.io/upload_images/3167229-3dfca6640806a6f5.gif?imageMogr2/auto-orient/strip)

### 用户注册步骤效果
本来我是打算用图片做，通过jquery来控制其是否显示，后来在网上找到了这个感觉很棒，自己也用到了。

![注册步骤效果图](http://upload-images.jianshu.io/upload_images/3167229-a70b18bae30c7413.gif?imageMogr2/auto-orient/strip)

[下载地址](http://download.csdn.net/detail/qqhjqs/9740258)
