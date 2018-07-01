---
title: Html页面内引入抽取出来的Html，js，css示例
date: 2017-04-24 22:31:16
categories: 技术
tags:
  - Bootstrap
  - Html
  - js
  - css
---


　　在写纯Html网站的时候，每个页面的头部菜单、js、css和底部说明都是同样的，有的时候你要改，就要一个一个的去改，通过下面方法将这些相同的**抽取**出来，方便后期维护！
希望能帮到你～！
<!--more-->

在applyBusiness.html页面引入公共页头header.html。

1、applyBusiness.html：
```html
<!DOCTYPE html>
<html>
  <head>
    <title>我的申请任务</title>
    <script type="text/javascript" charset="utf-8" src="../../../js/platform/system/header.js"></script>
  </head>
  <body>
      <div class="container">
	     <div class="mainBody mt-20 applyBusiness">
	         <form class="form-inline mt-30" id="searchFormId">
		       <div class="clearfix">
		           <div class="form-group col-sm-4 col-md-4 padding-0">
		                <label class="text-right labelBlock">科信业务事项：</label>
		                <select id="businessType" name="businessType" class="form-control optionLength">
	                            <option> </option>
	                        <option value="104" servicetype="1">故障申报</option><option value="105" servicetype="2">服务申请</option>
	                    </select>
		            </div>
		            <div class="form-group col-sm-4 col-md-4 padding-0">
		                <label class="text-right labelBlock">状态：</label>
		                <select id="serviceStatus" name="serviceStatus" class="form-control optionLength">
	                            <option value="all">全部</option>
	                            <option value="pending">待提交</option>
	                            <option value="handling">办理中</option>
	                            <option value="completed">已办结</option>
	                   </select>
		            </div>
		            <div class="form-group col-sm-4 col-md-4 padding-0">
		                <label class="text-right labelBlock">业务来源：</label>
		                <select id="serviceSource" name="serviceSource" class="form-control optionLength">
	                   </select>
		            </div>
		       </div>
		        <div class="clearfix mt-20">
		            <div class="col-sm-5 col-md-5 halfWidth">
		                <div class="form-group" id="applyTime1">
		                    <label class="text-right" for="startApplyDate">申请时间：</label>
		                    <input type="text" readonly="readonly" size="16" value="" id="startApplyDate" name="startApplyDate" class="form-control form_date" data-date-format="yyyy-mm-dd">
		                </div>
		                <div class="form-group" id="applyTime2">
		                    <label for="endApplyDate">-</label>
		                    <input type="text" readonly="readonly" size="16" value="" id="endApplyDate" name="endApplyDate" class="form-control form_date" data-date-format="yyyy-mm-dd">
		                </div>
		            </div>
		            <div class="col-sm-2 col-md-2 margin-top-20">
		                <button id="search-btn" type="button" class="btn btn-primary btn-warning marginLeft">搜索</button>
		                <button type="button" id="resetId" class="btn btn-primary btnIp">重置</button>
		            </div>
		        </div>
		    </form>
		    <table style="margin-top:30px" id="table"></table>
	    </div>
      </div>
	</body>
</html>
```
2、header.js:
```javascript
/**
 * 引用JS和CSS头文件
 */
var rootPath = getRootPath(); //项目路径

/**
 * 动态加载CSS和JS文件
 */
var dynamicLoading = {
	meta : function(){
		document.write('<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">');
	},
    css: function(path){
		if(!path || path.length === 0){
			throw new Error('argument "path" is required!');
		}
		document.write('<link rel="stylesheet" type="text/css" href="' + path + '">');
    },
    js: function(path, charset){
		if(!path || path.length === 0){
			throw new Error('argument "path" is required!');
		}
        document.write('<script charset="' + (charset ? charset : "utf-8") + '" src="' + path + '"></script>');
    }
};

/**
 * 取得项目路径
 * @author wul
 */
function getRootPath() {
	//取得当前URL
	var path = window.document.location.href;
	//取得主机地址后的目录
	var pathName = window.document.location.pathname;
	var post = path.indexOf(pathName);
	//取得主机地址
	var hostPath = path.substring(0, post);
	//取得项目名
	var name = pathName.substring(0, pathName.substr(1).indexOf("/") + 1);
	return hostPath + name + "/";
}

//动态生成meta
dynamicLoading.meta();

//动态加载项目 JS文件
dynamicLoading.js(rootPath + "/js/common/jquery-1.9.1.min.js", "utf-8");
dynamicLoading.js(rootPath + "/js/common/bootstrap.min.js", "utf-8");
dynamicLoading.js(rootPath + "/js/process/center/common/baseApp.js", "utf-8");
dynamicLoading.js(rootPath + "/js/process/center/common/bootstrap-datetimepicker.min.js", "utf-8");
dynamicLoading.js(rootPath + "/js/process/center/common/bootstrap-datetimepicker.zh-CN.js", "utf-8");
dynamicLoading.js(rootPath + "/js/process/center/common/jquery.dataTables.js", "utf-8");
dynamicLoading.js(rootPath + "/js/platform/system/loadHeader.js", "utf-8");

//动态加载项目 CSS文件
dynamicLoading.css(rootPath + "/css/common/bootstrap-3.3.5/css/bootstrap.min.css");
dynamicLoading.css(rootPath + "/css/common/bootstrap-3.3.5/css/bootstrap-responsive.css");
dynamicLoading.css(rootPath + "/css/workflow/css/jquery.dataTables.css");
dynamicLoading.css(rootPath + "/css/workflow/css/newWorkflow.css");
3、loadHeader.js 
$(function(){
	$(".container").append('<div id="header"></div>');
	$("#header").load(getRootPath() + "header.html");
});
```
4、header.html
```html
<nav class="navbar navbar-default navbar-fixed-top navWhole">
     <div class="container-fluid">
         <!-- Brand and toggle get grouped for better mobile display -->
         <div class="navbar-header">
             <button type="button" class="navbar-toggle collapsed navToggle" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
                 <span class="sr-only">Toggle navigation</span>
                 <span class="icon-bar"></span>
                 <span class="icon-bar"></span>
                 <span class="icon-bar"></span>
             </button>
           <a class="navbar-brand" href="#">![](./images/process/center/top/gongan.png)</a>
           <p class="navbar-text navHeader">科信服务门户</p>
         </div>

         <!-- Collect the nav links, forms, and other content for toggling -->
         <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
             <ul class="nav navbar-nav navbar-right navToggle">
                 <li><a href="./index.html">首页 </a></li>
                 <li class="dropdown">
                     <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">个人服务中心 <span class="caret"></span></a>
                     <ul class="dropdown-menu">
                         <li><a href="newWorkflow/personalManage/applyBusiness/html" target="_blank">我申请的业务</a></li>
                         <li><a href="newWorkflow/personalManage/dealBusiness/html">我经办的业务</a></li>
                     </ul>
                 </li>
                 <li class="dropdown">
                     <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">服务管理中心 <span class="caret"></span></a>
                     <ul class="dropdown-menu">
                         <li><a href="newWorkflow/serviceManage/businessFlowManage/html" target="_blank">业务表单管理</a></li>
                         <li><a href="newWorkflow/serviceManage/businessReportManage/html" target="_blank">业务流程管理</a></li>
                     </ul>
                 </li>
             </ul>
         </div><!-- /.navbar-collapse -->
     </div><!-- /.container-fluid -->
 </nav>
```
