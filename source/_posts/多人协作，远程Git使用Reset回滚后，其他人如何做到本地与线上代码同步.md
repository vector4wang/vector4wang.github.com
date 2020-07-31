---
title: 多人协作，远程Git使用Reset回滚后，其他人如何做到本地与线上代码同步
copyright: true
permalink: 1
top: 0
date: 2020-07-31 08:45:06
tags:
	- git
	- 版本管理
categories:
	- 工具
password:
---



一般公司里开发某个系统都是多人协作开发，使用Git作为代码版本管理工具，那你可能或者肯定会遇到其他人错提代码到线上,如何回退代码并且让其他人正确同步最新代码呢？

<!-- more -->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=26341097&auto=1&height=66"></iframe>

### 背景
一般公司里开发某个系统都是多人协作开发，使用Git作为代码版本管理工具，那你可能或者肯定会遇到其他人错提代码到线上，回退之后其他人再去pull的时候，代码并不会改变，本以为已经更新了，后续再次提交的时候，线上的代码又被“污染”了，然后再次回滚，其他人依然没有pull到新代码，然后往复循环，最后搞得大家都很疲惫。

### 场景重现

#### 准备

重现很简答，随意在远程git创建一个仓库，我在gitee创建一个测试库，然后本地创建两个目录模拟两个人在同时开发，一个student1，一个是student2，然后分别在两个文件夹下clone仓库，如图

![](http://cdn.wangxc.club//image1.png?imageslim)


然后我们对student1中的update.txt文件每增加一行，就commit和push一次，模拟代码提交了四次，如图

![](http://cdn.wangxc.club//image2.png?imageslim)


然后将student2下的代码pull一下，保持跟远程的一样

![](http://cdn.wangxc.club//image3.png?imageslim)


#### 重现

现在突然发现student1下的代码提交错了，代码需要回滚到第二次提交的地方，这里有两种方法，可以用Reset也可以用Revert来回滚，简答的来说用Reset会把第三次和第四次提交的记录直接删掉，如果用Revert他会生成第五次提交，即把第二次提交的记录重新提交一次变成第五次提交，这里主要说Reset

用命令行的话，是这样

```
# git reset --hard [commit hash]
git reset --hard 7b8bcaa0b02959126b923fe554824fa9df1dfd87
```

如图

![在这里插入图片描述]![](http://cdn.wangxc.club//image4.png?imageslim)


用git小乌龟的话，可以`git show log`,右键你需要回滚的commit，在勾选 hard，最后点确定，如图

![](http://cdn.wangxc.club//image5.png?imageslim)


用idea的话可以这样

![](http://cdn.wangxc.club//image6.png?imageslim)


回滚完记得push到远程

直接push会报错，会提示你本地的HEAD在远程代码的前面，如图

![](http://cdn.wangxc.club//image7.png?imageslim)

这个时候需要`git push -f`强制推到远程，成功之后，再去student2下pull最新的代码，这个时候可能很多人会认为**pull下来的代码就是远程回滚后的代码**了，其实并不是，pull之后的提示如下

![](http://cdn.wangxc.club//image8.png?imageslim)


很奇怪，并没有代码拉下来的提示，如果你在idea下pull的时候，只有`Already up to date`的提示，如果你认为这已经跟线上同步了，然后继续写代码提交，那么提交上去的还是回滚之前的代码，即远程的代码被你又改回去了，那么问题出在哪里呢？

### 发现并解决问题

student2下pull之后，先不要改动，使用命令`git log --pretty=oneline graph` 或者`gitk`看下git树有啥异常，跟student1的做个对比

![](http://cdn.wangxc.club//image9.png?imageslim)


发现了什么，student1里的git HEAD 本地和线上是一致的，而student2里，则不是本地的HEAD在最新的第四次提交上，远程HEAD上在第二次的提交上即要回滚到的提交上，说明了以下问题：

1、git reset 改的是git的HEAD

2、其他人的需要统一本地和线上的HEAD

那么如何去解决呢？很简单，用这个命令

```
git reset --hard origin/master
```

就是重置HEAD跟某个分支的保持一致，origin/master就是远程的master，结果如图

![](http://cdn.wangxc.club//image10.png?imageslim)


这样就跟远程的保持了统一，但是用hard的危害也很大，直接会把第二次提交之后的代码删掉，如果你本地做了修改，用这个命令的话，很抱歉，直接被干掉了。所以确保没有变更在做hard或者备份一份

用git小乌龟右键需要同步的HEAD的commit，再reset

![](http://cdn.wangxc.club//image12.png?imageslim)

用IDEA的话就更简单了
![](http://cdn.wangxc.club//image11.png?imageslim)

### 最后

代码不会说谎，错了就是错了，一定是有原因的，找到原因并解决它，后面就不会再犯了