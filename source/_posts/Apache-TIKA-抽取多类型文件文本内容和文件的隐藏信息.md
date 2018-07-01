---
title: Apache TIKA---抽取多类型文件文本内容和文件的隐藏信息
date: 2017-05-08 21:26:57
categories: 技术
tags: 
 - apache
 - tika
 - web
 - 抽取文本内容
 
---

使用Apache的开源项目Tika来抽取各种类型文件的文本内容及MetaData
<!--more-->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=337167&auto=1&height=66"></iframe>

### 前言
有这样一个需求“用户上传一个文件，要得到这个文件的文本内容，和它的创建时间(用户创建的时间)”
乍一看上去，很简单啊，可以按字节读文件或按行读文件，也可以根据文件的类型引入对应的jar包去获取内容。文件的创建时间，我找了一些资料，可以通过下面代码实现：
```java
Process p = Runtime.getRuntime().exec("cmd /C dir "           
                    + filePath  
                    + "/tc" );  
InputStream is = p.getInputStream();   
BufferedReader br = new BufferedReader(new InputStreamReader(is));             
String line;  
while((line = br.readLine()) != null){  
    if(line.endsWith(".txt")){  
        strTime = line.substring(0,17);  
        break;  
     }                             
}   
```
到了这里，我正打算去实现，转念一想，apach的工具包里有提供文件复制的功能，即`IOUtils.copy(InputStream input, OutputStream output)`，那同样的Apach是否拥有提取文件内容和时间的工具类呢，还真有！这就是接下来要说的**Apache  Tika**

### Tika

>The Apache Tika™ toolkit detects and extracts metadata and text from over a thousand different file types (such as PPT, XLS, and PDF). All of these file types can be parsed through a single interface, making Tika useful for search engine indexing, content analysis, translation, and much more. You can find the latest release on the download page. Please see the Getting Started page for more information on how to start using Tika.

[官方地址](http://tika.apache.org/)

官方介绍说它可以支持上千种文件类型，同时支持搜索引擎索引、内容分析、翻译。。。看到这，感觉我又发现了新大陆！！！
下面这张图是tika的主要结构图：
![tika_architecture.jpg](http://upload-images.jianshu.io/upload_images/3167229-0d3c151f5d34f9ff.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

tika会根据你输入的文件**自动**分配解析工具，如`PDFParser`、`OOXMLParser`和`HtmlParser`等等，然后解析语言、MIME type、文本内容和Metedata

我这里使用到Tika两个功能：

- 文本内容抽取
- 元数据(MetaData)的获取
补充：MetaData即描述数据的数据！如一个文件的创建者，穿件时间，文档类型，字节数等等。[维基百科](https://en.wikipedia.org/w/index.php?title=Metadata&action=history)

这里是一个简单教程，能让你快速了解并使用其功能http://www.yiibai.com/tika/

### 使用
Tika是一个开源的项目，代码在这https://github.com/apache/tika，他提供了多个服务，有图形界面的也有Http服务
#### GUI
将项目checkout到本地并执行`mvn clean install`，然后进入tika-app目录，执行
```bash
java -jar tika-app-1.4.jar --gui
```
如下动图

![gui](http://upload-images.jianshu.io/upload_images/3167229-0f4b8c9ec0ecfac0.gif?imageMogr2/auto-orient/strip)

#### Server
关于官方指定使用`curl`，我就在ubuntu上启动了server，启动的方式为
```bash
java -jar tika-server/target/tika-server.jar
```
测试了一个文件，如下图

![server.png](http://upload-images.jianshu.io/upload_images/3167229-5606fa60fadcf365.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


还有其他的方法，如：

* Extract plain text:  
`curl -T price.xls http://localhost:9998/tika`

* Extract text with mime-type hint:  
`curl -v -H "Content-type: application/vnd.openxmlformats-officedocument.wordprocessingml.document" -T document.docx http://localhost:9998/tika`

* Get all document attachments as ZIP-file:  
`curl -v -T Doc1_ole.doc http://localhost:9998/unpacker > /var/tmp/x.zip`

* Extract metadata to CSV format:  
`curl -T price.xls http://localhost:9998/meta`

* Detect media type from CSV format using file extension hint:  
`curl -X PUT -H "Content-Disposition: attachment; filename=foo.csv" --upload-file foo.csv http://localhost:9998/detect/stream`

可根据自己的需要去调用

### 整合本地服务
虽然tika自己提供了服务，但是有的时候，我们想在本地自己搭建一个服务，在抽取完内容后或许还要做些其他的工作。就这个目的，我将1.14版本的tika整合到了本地，并自己搭建了一个简单的服务，接口如下

![1.png](http://upload-images.jianshu.io/upload_images/3167229-a96d28392ed317a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2.png](http://upload-images.jianshu.io/upload_images/3167229-7d08685f66e9ea63.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

依然使用swagger整理接口，如果你对swagger不了解，可以[点这里](http://www.jianshu.com/p/5bc11edca2d1),代码放在了github，欢迎[查看](https://github.com/vector4wang/springbootquick/tree/tika)

