---
title: 如何优雅的“编写”api接口文档（续）
date: 2017-05-04 23:35:23
categories: 技术
tags:
	- Swagger
	- api
	- 文档
---


解决上传文件、单页面查询多服务API和完善API显示的信息功能

<!--more-->


![Swagger-logo-white.png](http://upload-images.jianshu.io/upload_images/3167229-d43f5cb18cd10fc1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接着上一篇[如何优雅的“编写”api接口文档](/2017/05/03/如何优雅的“编写”api接口文档/) 

这篇续篇主要说一下以下三点

- 文件上传参数配置
- 单独部署SwaggerUI(实现在一个页面查找不同域名的API功能)
- 完善SwaggerUI的展示内容

### 上传文件接口配置

　　接着昨天的bug，有一接口需要上传文件，怎样去配置注解，才能让Swagger的页面拥有测试的功能即有**文件上传的入口**(不知客官有咩有明白。。。)
在网上找了半天的资料也没有找到，后来我索性直接将`@ApiImplicitParams`注解去掉，竟然可以了，英吹思婷~，文件这块解决。

### 单页面查询多服务API文档

　　不知道看到这个二级标题有咩有明白，上一篇是将Swagger集成在单独的服务里，但是，现在你有多个服务，难道要在地址栏输入多个地址去查看API么，总感觉怪怪的，就不能再一个页面去查找API么，我反复的查看服务启动的swagger日志，发现了如下图的日志

![swagger 日志](http://upload-images.jianshu.io/upload_images/3167229-d646dff2488451d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我又看了看官网提供的[Tools](http://swagger.io/tools/),好像明白了什么
> Swagger 根据注解会将接口的相关信息生成一个供SwaggerUI界面使用的一个接口即`/v2/api-docs` 

官网里的LIVE DEMO是一个在线服务，为了安全我这里不使用在线的，准备在本地自己搭一个
将SwaggerUI源代码checkout到本地
```bash
git clong https://github.com/swagger-api/swagger-ui`
```
如果你用的linux，进入目录并使用下面命令
```bash
docker pull swaggerapi/swagger-ui
docker run -p 80:8080 swaggerapi/swagger-ui
```
使用windows，直接进入dist目录，点击index.html运行，或者将整个dist目录放在你的服务器里，随服务器启动。
我的80端口被占用了，就是用的81，地址栏输入**http://yourdomain:81/?#**(注意后面的**?#**，一定要带上，不然页面不会有数据返回，至于为什么，还在研究中...)，界面如下
![默认界面.png](http://upload-images.jianshu.io/upload_images/3167229-e9e200960e482ac8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*注意：这里说一下，因为不同的服务可能会牵扯到跨域访问的问题，为了解决这个问题，我在每个服务添加了下面配置*


```java
@Configuration  
public class CorsConfig extends WebMvcConfigurerAdapter {  

    @Override  
    public void addCorsMappings(CorsRegistry registry) {  
        registry.addMapping("/**")  
                .allowedOrigins("*")  
                .allowCredentials(true)  
                .allowedMethods("GET", "POST", "DELETE", "PUT")  
                .maxAge(3600);  
    }  

}  
```
如果上线了，你可以把`allowedOrigins`的*号换成指定的域名

好了，在界面里的输入框里输入你想查找的API文档的地址，如
`http://127.0.0.1:8080/v2/api-docs`
这个时候你就能看到内容啦~~~~(注意和直接在地址栏输入`http://127.0.0.1:8080/swagger-ui.html#/`)的区别

![单独API.png](http://upload-images.jianshu.io/upload_images/3167229-cffb562350fbe70a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![可查询API页面.png](http://upload-images.jianshu.io/upload_images/3167229-f2f2d2860ae43017.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 完善界面显示内容
添加如下代码，可以自定义API页面显示的内容，看起来更加的规范
```java
@SpringBootApplication
@EnableSwagger2
public class Application {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.quick.api")) //扫描API的包路径
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("SpringBoot整合Swagger2") // 标题
                .description("api接口的文档整理，支持在线测试") // 描述
                .termsOfServiceUrl("http://vector4wang.tk/") //网址
                .contact("Vector.Wang") // 作者
                .version("1.0") // 版本号
                .build();
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class);
    }
}
```
结果如上面的单独API





