---
title: 我的ElasticSearch使用笔记
date: 2018-03-04 22:43:17
categories: 
	- 技术
tags:
	- Docker
	- ElasticSearch
---

在工作中，我开始接触并使用ES的笔记

<!--more-->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=5283988&auto=1&height=66"></iframe>

> 以下基于Elastic 5.4版本

### 部署
这里使用Docker部署

- 获取镜像`docker pull elasticsearch:5.4`
- 启动 `docker run -d -p 9200:9200 -p 9100:9100  elasticsearch:5.4`

注意: 通过`docker ps`可以看到es的启动情况，如果没有成功可以通过`docker logs elasticsearch:5.4` 查看日志，一般会报这个错误`Cannot allocate memory`，此时加上`-e ES_JAVA_OPTS="-Xms512m -Xmx512m"`,全命令即
```bash
docker run -d -p 9200:9200 -p 9100:9100 -e ES_JAVA_OPTS="-Xms512m -Xmx512m" elasticsearch:5.4
```

### ES 的相关使用
#### 查看状态
`GET _cat/health?v`
[![1.png](http://upload-images.jianshu.io/upload_images/3167229-dbe1e2c1af0ad3bf..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://i.loli.net/2018/03/04/5a9c02bb4a964.png)

#### 查看节点列表
`GET _cat/nodes?v`
[![2.png](http://upload-images.jianshu.io/upload_images/3167229-74eada220fc922be..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://i.loli.net/2018/03/04/5a9c02bc15461.png)
#### 创建索引
`PUT index?pretty`
[![3.png](http://upload-images.jianshu.io/upload_images/3167229-05caefa1d8650ac3..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://i.loli.net/2018/03/04/5a9c02bbdfb8b.png)
#### 查看索引列表
`GET _cat/indices?v`
[![4.png](http://upload-images.jianshu.io/upload_images/3167229-dd0e18625452c92e..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://i.loli.net/2018/03/04/5a9c02bc1afcf.png)
#### 删除索引
`DELETE index?pretty`
[![5.png](http://upload-images.jianshu.io/upload_images/3167229-b71aa9dcad3174b1..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://i.loli.net/2018/03/04/5a9c02ba88e2f.png)
#### 创建Mapping
`POST index/type/_mapping`
```json
{
  "student": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "jieba_index",
        "search_analyzer": "jieba_search"
      },
      "class": {
        "type": "keyword"
      },
      "age": {
        "type": "integer"
      },
      "sex": {
        "type": "integer"
      },
      "ranking": {
        "type": "integer"
      }
    }
  }
}
```
[![6.png](http://upload-images.jianshu.io/upload_images/3167229-bae8df0641528034..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://i.loli.net/2018/03/04/5a9c02bc2b46e.png)
因为没有安装结巴分词插件，所以创建失败
#### 查看Mapping
`GET index/_mapping?pretty`
[![7.png](http://upload-images.jianshu.io/upload_images/3167229-f2cfce11dae2109a..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://i.loli.net/2018/03/04/5a9c02bbc354f.png)
#### 索引文档
`POST index/type/{id}`
```json
{"age":23,"class":"一年级2班","name":"小黑","ranking":13,"sex":1}
```
[![8.png](http://upload-images.jianshu.io/upload_images/3167229-42532742913e3c6b..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://i.loli.net/2018/03/04/5a9c02bc41033.png)
#### 删除所有文档数据
`POST index/type/_delete_by_query?conflicts=proceed`
```json
{
  "query": {"match_all": {}}
}`
```

### 重点

#### ES支持的类型
这里用的是5.4 

|类型|包括|
|---|---|
|String|text, keyword|
|Number|long, integer, short, byte, double, float, half_float, scaled_float|
|Date|date|
|Boolean|boolean|
|Binary|binary|
|Rande|integer_range, float_range, long_range, double_range, date_range|

官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/5.4/mapping-types.html

关于String类型的text和keyword，我的理解是
> 如果该字段需要被分词，就使用text，如果不需要分词就使用keyword

#### 分词
分词在ES中是比较重要的，因为分词的好坏直接影响到搜索结果准确度的高低！
*分词器 接受一个字符串作为输入，将 这个字符串拆分成独立的词或 语汇单元（token） （可能会丢弃一些标点符号等字符），然后输出一个 语汇单元流（token stream） 。*
https://www.elastic.co/guide/cn/elasticsearch/guide/current/standard-tokenizer.html

分词分为两种：*索引分词*、*搜索分词*
在创建mapping的时候声明，如上
```json
"name": {
    "type": "text",
    "analyzer": "jieba_index",
    "search_analyzer": "jieba_search"
  }
```

(下面是个人观点，如有问题，欢迎指出）
举个例子：
“我爱吃肠粉” 经过分词后可能有以下几个结果
- 我，爱，吃，肠，粉
- 我爱，吃，肠粉
- 我爱吃，肠粉
- 。。。

那么分词在ES中是怎样应用的？
当“我爱吃肠粉”索引到ES中之后，ES中对此句的描述变为“我”，“爱”，“吃”，“肠”，“粉”，此为索引分词

当用户查询“我”的时候，ES会将分词结果中包含“我”的结果输出，所以“我爱吃肠粉”会被搜索出来；如果输入“我爱”，而且搜索分词的结果也为“我爱”的时候，“我爱吃肠粉”则不会被搜索出来

ES默认的分词为“英文分词”，即“我爱吃肠粉”的第一种分词结果，这很显然不符合我们一般的应用场景，所以这个时候就需要引入第三方插件了，如“结巴分词”和IK分词
IK: https://github.com/medcl/elasticsearch-analysis-ik
Jieba: https://github.com/sing1ee/elasticsearch-jieba-plugin

按照自己的具体搜索场景来选择合适的分词插件

#### term 和 match 的使用
可学习这篇博客：http://www.cnblogs.com/yjf512/p/4897294.html 写的详细全面

### 2018年3月5日 更新

#### 更新

`PUT /index/type/id`
```json
{
  "title": "My first blog entry",
  "text":  "I am starting to get the hang of this...",
  "date":  "2014/01/02"
}
```
这个id是es自己的id(可在索引的时候设置id)
java实现

```java
UpdateRequest updateRequest = new UpdateRequest();
updateRequest.index(index);
updateRequest.type(document_type);
updateRequest.id(resumeId);
updateRequest.doc(jsonBuilder().startObject().field(fileName, fileValue).endObject());
UpdateResponse updateResponse = elasticSearchClient.getClient().update(updateRequest).get();
```
https://www.elastic.co/guide/cn/elasticsearch/guide/current/update-doc.html

#### updateByquery

`POST index/_update_by_query`
```json
{
  "script": {
    "inline": "ctx._source.likes++",
    "lang": "painless"
  },
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
```
ctx._source 为 一条记录对象
上面是“将查出来的文档中likes的值加1”

java实现
```java
TransportClient client = elasticSearchClient.getClient();
UpdateByQueryRequestBuilder updateByQueryRequestBuilder = UpdateByQueryAction.INSTANCE.newRequestBuilder(client);
String script = "";
if (1 == switchValue) {
    script = "ctx._source.is_buy = 1";
} else {
    script = "ctx._source.is_buy = 0";
}
Script scriptObj = new Script(script);
BulkByScrollResponse bulkByScrollResponse = updateByQueryRequestBuilder.source(index)
        .script(scriptObj)
        .filter(QueryBuilders.termQuery("owner_id", ownId)).abortOnVersionConflict(false).get();
List<BulkItemResponse.Failure> bulkFailures = bulkByScrollResponse.getBulkFailures();
for (BulkItemResponse.Failure bulkFailure : bulkFailures) {
    logger.error(bulkFailure.getMessage());
        }
```

https://www.elastic.co/guide/en/elasticsearch/reference/5.4/docs-update-by-query.html

以上就是我在工作中使用ES 总结的内容，入门到会使用应该是没问题。之后会继续学习并更新~

CSDN：http://blog.csdn.net/qqhjqs?viewmode=list 
博客：http://vector4wang.tk/ 
简书：https://www.jianshu.com/u/223a1314e818 
Github:https://github.com/vector4wang 
Gitee:https://gitee.com/backwxc 