---
title: Logstash 简易教程
copyright: true
permalink: 1
top: 0
date: 2019-06-25 23:03:28
tags:
    - Logstash
    - 数据同步
categories:
    - 技术
password:
---

对于数据同步功能你是否厌倦了写代码？那么使用logstash吧，带你进入新的坑~
<!--more-->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=86205&auto=1&height=66"></iframe>

建议在使用logstash之前先想清楚自己的需求是什么，从哪种数据源同步到哪里，需要经过怎么样的处理。因为logstash版本迭代较快，每个版本的插件都有点区别，比如filter中的http插件在6.6版本以后才有；output到现在(7.1)都没有jdbc的插件，然而你如果想使用output的jdbc插件就需要自己去安装热心人自己写的插件(logstash-output-jdbc),不幸的是，该作者指出没有很多的时间去维护此插件，不能保证6.3以后用户的正常使用。
也就是说，如果你想用output的jdbc，你就必须使用6.3以下(最好5.x)的版本，如果你想用官方filter的http插件，你就得用6.5以上的版本。话说logstash为什么不把jdbc加入到output中去呢。。。

另外：最好在linux下进行调试或使用，因为用windows会出现恶心的乱码问题。。。。

## 安装

> 如果目标数据源的type为jdbc，则建议安装logstash-5.4.1或6.x一下的版本，因为logstash-output-jdbc只在6.x以下有效，本教程对应的版本为logstash-5.4.1

建议直接下载zip包进行使用，解压完之后的目录如下
```
.
├── bin  # 二进制脚本，包括用于启动Logstash的logstash和用于安装插件的logstash-plugin
├── config # Configuration files, including logstash.yml and jvm.options
├── CONTRIBUTORS
├── data
├── Gemfile
├── Gemfile.jruby-2.3.lock
├── lib
├── LICENSE
├── logs # 日志文件
├── logstash-core
├── logstash-core-plugin-api
├── modules
├── NOTICE.TXT
├── tools
└── vendor
```

https://www.elastic.co/guide/en/logstash/7.1/dir-layout.html

可以在对应目录下，执行如下命令来验证是否安装成功
```
bin/logstash -e 'input { stdin { } } output { stdout {} }'
```
等待启动成功，直接输入“hello world”
会有如下显示
```
{
      "@version" => "1",
    "@timestamp" => 2019-06-20T06:46:48.310Z,
       "message" => "hello world",
          "host" => "localhost.localdomain"
}
```
则安装成功


## 语法
### 区段
Logstash 设计了自己的 DSL —— 有点像 Puppet 的 DSL，或许因为都是用 Ruby 语言写的吧 —— 包括有区域，注释，数据类型(布尔值，字符串，数值，数组，哈希)，条件判断，字段引用等。

Logstash 用 {} 来定义区域，区域内可以包括插件区域定义，你可以在一个区域内定义多个插件。插件区域内则可以定义键值对设置。示例如下：
```conf
input {}
filter {}
output {}
```

### 数据类型

|类型|示例|
|:---:|:---:|
|bool|debug => true|
|bytes|my_bytes => "113" # 113 bytes|
|string|host => "hostname"|
|number|port => 214|
|array|match =>[ "/var/log/messages", "/var/log/*.log" ]|
|hash|options => {key1 => "value1",key2 => "value2" }|

### 条件判断
|名称|符号|
|:---:|:---:|
|等于| ==|
|不等于| !=|
|小于| <|
|大于| >|
|小于等于| <=|
|大于等于| >=|
|匹配正则| =~|
|不匹配正则| !~|
|包含| in|
|不包含| not in |
|与|  and|
|或|  or|
|非与| nand|
|非或| xor|
|复合表达式| ()|
|取反符合| !()|


## 示例

### 读取日志文件
https://www.elastic.co/guide/en/logstash/current/plugins-inputs-file.html#plugins-inputs-file-type
各参数介绍与赋值

全量同步一个文件的所有内容，并且进行监听
```
input {
    file {
        path => ["/usr/local/logstash/logstash-tutorial-dataset"]
        type => "file_monitor"
        tags => ["有用的","标识用的"]
        start_position => "beginning"
    }

}
output {
   stdout {}
}
```

监听后不能打开编辑再保存必须使用echo >> 对文件进行追加内容，可以看到
```
{
       "message" => "2019年6月20日16:57:52",
    "@timestamp" => 2019-06-20T08:57:55.624Z,
          "tags" => [
        [0] "有用的",
        [1] "标识用的"
    ],
          "host" => "localhost.localdomain",
          "path" => "/usr/local/logstash/logstash-tutorial-dataset",
      "@version" => "1",
          "type" => "file_monitor"
}
```
### 读取mysql
```
input {
    jdbc {
        jdbc_driver_library => "/usr/local/logstash/mysql/mysql-connector-java-5.1.47.jar"
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        jdbc_connection_string => "jdbc:mysql://xxx:xxx/data"
        jdbc_user => "xxx"
        jdbc_password => "xxx"
        parameters => { "id" => 666 }
        statement => "SELECT * FROM xxx_t_job_function_net_bak20140828 WHERE id > :id"
    }
}
output {
    stdout {}
}
```

###  写mysql
官方没有提供写mysql的插件，所以需要自己下载安装
https://github.com/theangryangel/logstash-output-jdbc

`./logstash-plugin install logstash-output-jdbc`

可惜的是，目前插件只支持6.3以前，当前logstash最新版本为7.1.1，
测试用的是logstash-5.4.1 较稳定

```
input {
    jdbc {
        jdbc_driver_library => "/usr/local/logstash/mysql/mysql-connector-java-5.1.47.jar"
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        jdbc_connection_string => "jdbc:mysql://120.55.168.18:8840/data"
        jdbc_user => "crawler"
        jdbc_password => "crawler9988@ed"
        parameters => { "id" => 666 }
        statement => "SELECT * FROM xxx_t_job_function_net_bak20140828 WHERE id > :id"
    }
}
output {
    jdbc {
        driver_jar_path => "D:\repo\mysql\mysql-connector-java\5.1.40\mysql-connector-java-5.1.40.jar"
        driver_class => "com.mysql.jdbc.Driver"
        connection_string => "jdbc:mysql://sss:8840/testcase"
        username => "sss"
        password => "csssd"
        statement => ["INSERT INTO job_function_20190621 ( code_val, name_val, level_val, source_name, version ) VALUES (?,?,?,?,?)","code","name","level","source_name","current_version"]
    }
    stdout {}
}
```

[input](https://www.elastic.co/guide/en/logstash/5.4/input-plugins.html)

[output](https://www.elastic.co/guide/en/logstash/5.4/output-plugins.html)

[filter](https://www.elastic.co/guide/en/logstash/5.4/filter-plugins.html)

以上三个链接为官方文档，基本上涵盖了可能会用到的数据源头与目标，如果没有的话，可以去github上搜索一下一般都会有人自己开发插件如`logstash-output-jdbc`

### 完整的示例

- 源数据。data数据库的`xxx_t_job_function_net_bak20140828`表，获取`id>666`的数据
- 过滤(清洗)数据。利用dict数据库`dict_location`表替换数据源中code，去除对应的`location_name_cn`,如遇到多条数据则合并为一条用“;”分开
- 目标数据。将整理好的数据保存在testcase数据库的·job_function_20190621`表中


```
input {
    jdbc {
        jdbc_driver_library => "D:\repo\mysql\mysql-connector-java\5.1.40\mysql-connector-java-5.1.40.jar"    
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        jdbc_connection_string => "jdbc:mysql://xxx:xxx/data"
        jdbc_user => "xxx"
        jdbc_password => "xxxx"
        statement => "SELECT * FROM xxx_t_job_function_net_bak20140828 WHERE id > :id"
        parameters => { "id" => 666 }
    }
}
filter {
    grok {
        match => {"@timestamp" => "%{YEAR:year}-%{MONTHNUM:month}-%{MONTHDAY:day}" }
        add_field => { "current_version" => "%{year}%{month}%{day}"}
    }
    jdbc_streaming {
        # 加了会错误，可能引用了上面的input jdbc_driver_library => "D:\repo\mysql\mysql-connector-java\5.1.40\mysql-connector-java-5.1.40.jar"
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        jdbc_connection_string => "jdbc:mysql://xxx:3306/xxx"
        jdbc_user => "xxx"
        jdbc_password => "xxx"
        statement => "SELECT location_name_cn FROM dict_location WHERE location_code = :codeParam"
        parameters => { "codeParam" => "code"}
        target => "code"
    }
    if [code] and [code][0] and ("location_name_cn" in [code][0]) {
        ruby {
            code => "
            r = ''
            event.get('code').each do |variable|
               # puts variable['location_name_cn']
               r = r + variable['location_name_cn'] + ';'
            end 
            event.set('code',r)
            "
        }
    } else {
        mutate {
            replace => { "code" => ""}
        }
    }
}
output {
    stdout {
        codec => rubydebug{}        
    }
    jdbc {
        driver_jar_path => "D:\repo\mysql\mysql-connector-java\5.1.40\mysql-connector-java-5.1.40.jar"
        driver_class => "com.mysql.jdbc.Driver"
        connection_string => "jdbc:mysql://xxx:8840/xxx"
        username => "xxx"
        password => "xxx"
        statement => ["INSERT INTO job_function_20190621 ( code_val, name_val, level_val, source_name, version ) VALUES (?,?,?,?,?)","code","name","level","source_name","current_version"]
    }
}

```

## 最佳实践
1、整理并调试好读取数据与插入数据的脚本或参数；

2、一般字符串的操作官方给的插件中就可以解决如Date、json、mutate等

3、遇到比较复杂的操作可以借助ruby，在编译器中调试好，直接复制过来，可以实时调试；

4、grok为提取日志的核心操作组件

5、注意变量的引用，内置参数为@timestamp
    
字符串中引用 "the longitude is %{[geoip][location][0]}"
作为判断的时候可以`[filed]`使用

https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html
