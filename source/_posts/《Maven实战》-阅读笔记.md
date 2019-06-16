---
title: 《Maven实战》 阅读笔记
copyright: true
permalink: 1
top: 0
date: 2019-06-16 16:17:14
tags: 
	- Java
	- Maven
	- 读书笔记
categories:
	- 技术
password:
---



无Maven不项目，这是我的口号，但是一直没有系统的去看书，前段时间把《Maven实战》过了一遍做了些笔记，方便后面查阅

<!-- more -->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=35040360&auto=1&height=66"></iframe>

### 每个项目都有自己的坐标

- groupId
- artifactId
- version
- packaging
- classifier
![1.png](https://i.loli.net/2019/06/16/5d05e8190ec0915874.png)

![2.png](https://i.loli.net/2019/06/16/5d05e81a50f3c33060.png)


- install 安装在本地
- deploy 打包发布到远端


### 依赖范围

![3.png](https://i.loli.net/2019/06/16/5d05e8192057d54933.png)

范围类型有: 编译、测试、运行 三种classpath

- compile，默认值，对编译、测试、运行三种classpath都有效；
- test: 测试依赖范围，只对测试classpath有效，在编译主代码或者裕兴项目的使用是则无法使用此类依赖。
- provided: 已提供依赖范围 只在编译和测试classpath有效，运行时无效。
- runtime: 运行时依赖范围，对测试和运行classpath有效，编译无效；
- system: 系统依赖范围,
![4.png](https://i.loli.net/2019/06/16/5d05e81b4121380622.png)

(如第三方给的jar包，且仓库中心又没有，可以使用system范围，如)
```
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.0</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/src/lib/jsoup-proxy.jar</systemPath>
</dependency>
```
- import（Maven 2.0.0及以上）：导入依赖范围。不会对三种classpath产生实际的影响；(不够严谨，类似complie还是什么？)
![5.png](https://i.loli.net/2019/06/16/5d05e8190fc6229631.png)


### 传递性依赖

![6.png](https://i.loli.net/2019/06/16/5d05e8195122884781.png)


### 依赖调解

两个依赖关系
A->B->C->X(1.0)
A->D->X(2.0)
两个依赖都间接来到X，
maven引用原则(依赖调解(Dependency Mediation))

- 第一原则：路径最紧者优先 即 x(2.0)会被使用

A->B->Y(1,0)
A->C->Y(2,0)

- 第二原则： 在满足第一原则的前提下第一声明者优先(就近原则) 即 Y(2,0)会被使用

### 可选依赖
![7.png](https://i.loli.net/2019/06/16/5d05e818cac4265196.png)

![8.png](https://i.loli.net/2019/06/16/5d05e8191a4b550589.png)

![9.png](https://i.loli.net/2019/06/16/5d05e81b4121375170.png)

![10.png](https://i.loli.net/2019/06/16/5d05e8191c4fb75994.png)


### 仓库的布局
![12.png](https://i.loli.net/2019/06/16/5d05ef377af5983200.png)


### 生命周期
- clean:清理项目
![13.png](https://i.loli.net/2019/06/16/5d05ef365b75068487.png)
- default: 构建项目
![14.png](https://i.loli.net/2019/06/16/5d05ef380234184140.png)
- site: 建立项目站点
![16.png](https://i.loli.net/2019/06/16/5d05ef368053172674.png)

![15.png](https://i.loli.net/2019/06/16/5d05ef36a7ab540023.png)

### 继承
正确的设置relativePath很重要
![17.png](https://i.loli.net/2019/06/16/5d05ef374e9ff68912.png)


可继承的pom元素
![18.png](https://i.loli.net/2019/06/16/5d05ef3701c5266346.png)


依赖范围 import的用法
![19.png](https://i.loli.net/2019/06/16/5d05ef370a0fe16641.png)

插件跟依赖一样可以使用*Management来管理
![20.png](https://i.loli.net/2019/06/16/5d05ef3706a5231265.png)
![21.png](https://i.loli.net/2019/06/16/5d05ef37440a772560.png)

### 反应堆
![22.png](https://i.loli.net/2019/06/16/5d05f1028ffe650360.png)

![23.png](https://i.loli.net/2019/06/16/5d05f10688fa856752.png)

![24.png](https://i.loli.net/2019/06/16/5d05f1029e1b845641.png)

![25.png](https://i.loli.net/2019/06/16/5d05f102e957929216.png)

对于裁剪功能，需要用的时候可以查阅文档

### 测试

![26.png](https://i.loli.net/2019/06/16/5d05f102a702991471.png)
![27.png](https://i.loli.net/2019/06/16/5d05f102a410334513.png)

跳过测试
`mvn package -DskipsTests`

![28.png](https://i.loli.net/2019/06/16/5d05f102a1da325737.png)

![29.png](https://i.loli.net/2019/06/16/5d05f102a002311742.png)

![30.png](https://i.loli.net/2019/06/16/5d05f1032352821568.png)


注意：上述几种命令行动态指定测试类的方法都应该只是临时使用，如果长时间只运行项目的某几个测试，那么测试就会慢慢失去其本来的意义。

### 加入测试
![31.png](https://i.loli.net/2019/06/16/5d05f102f154b44118.png)

也可以使用excludes排除一些测试
![32.png](https://i.loli.net/2019/06/16/5d05f1f39ad6452792.png)


### WEB应用
![33.png](https://i.loli.net/2019/06/16/5d05f1f392fb165585.png)


版本号定义约定
![34.png](https://i.loli.net/2019/06/16/5d05f1f39905845755.png)

![35.png](https://i.loli.net/2019/06/16/5d05f1f35d0a452424.png)


### Maven属性
#### 内置属性:
```
${basedir} 标识项目根目录即包含pom.xml 文件的目录；
${version}标识项目版本；
```
#### POM属性:

- `${project.artifactId}` 对应了<project><artifactId>元素的值
- `${project.build.sourceDirectory}` 项目的主源码目录 默认src/main/java/
- `${project.build.testSourceDirectory}` 项目的测试源码目录，默认为src/test/java
- `${project.outputDirectory}` 项目主代码编译输出目录，默认为target/classes
- `${project.testOutputDirectory}`： 项目测试代码编译输出目录 ，默认为target/testclasses/
- `${project.groupId}`： 项目的groupId
- `${project.artifactId}` 项目的artifactId
- `${project.version}` 项目的version 与`${version}`等价
- `${project.build.finalName}` 项目打包输出文件的名称，默认为`${project.artifactId}`-`${project.version}`

#### 自定义属性
可以通过`<properties><xxx>val</xxx></properties>`

#### Setting属性
![36.png](https://i.loli.net/2019/06/16/5d05f1f36cde883488.png)


#### Java属性变量
![37.png](https://i.loli.net/2019/06/16/5d05f1f473d2a25905.png)


#### 环境变量属性
![38.png](https://i.loli.net/2019/06/16/5d05f1f4a918493822.png)

### 最佳实践
- artifactId 使用实际项目名称作为artifactId的前缀
- 优化依赖： 使用 `dependency:list` 或 `dependency:tree` 来查看依赖关系
- `mvn clean install` 在执行真正的项目构建之前清理项目是一个很好的实践



