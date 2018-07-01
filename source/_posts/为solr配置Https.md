---
title: 为solr配置Https
date: 2017-04-24 22:31:16
categories: 技术
tags:
  - Solr
  - Https
---

为solr配置Https
<!--more-->
## 为solr配置Https
### 使用keytool生成*.keystore密钥
```bash
keytool -genkeypair -alias solr-ssl -keyalg RSA -keysize 2048 -keypass secret -storepass secret -validity 9999 -keystore solr-ssl.keystore.jks -ext SAN=DNS:localhost,IP:192.168.1.3,IP:127.0.0.1 -dname "CN=localhost, OU=Organizational Unit, O=Organization, L=Location, ST=State, C=Country"
```
注意里面的IP，还有后面的一些参数"localhost、Organizational Unit、Organization。。。"
命令执行后，就会产生一个文件`solr-ssl.keystore.jks`
### 转换密钥
再次通过keytool工具将 JKS 密钥库转换成使用 keytool PKCS12 格式
```bash
keytool -importkeystore -srckeystore solr-ssl.keystore.jks -destkeystore solr-ssl.keystore.p12 -srcstoretype jks -deststoretype pkcs12
```
注意：以上两个步骤，会让你输入密码，一个是密钥库密码(自己设置)还有一个源密码(默认为*secret*)，要记住！
接下来使用 openssl 命令转换 PKCS12 格式 keystore，包括证书和密钥：
```bash
openssl pkcs12 -in solr-ssl.keystore.p12 -out solr-ssl.pem
```
密码为刚刚自己设置的。

### 配置solr.in.sh
在solr文件下"solr-5.5.3/bin/solr.in.sh"
找到对应代码
```bash
#SOLR_SSL_KEY_STORE=etc/solr-ssl.keystore.jks
#SOLR_SSL_KEY_STORE_PASSWORD=secret
#SOLR_SSL_TRUST_STORE=etc/solr-ssl.keystore.jks
#SOLR_SSL_TRUST_STORE_PASSWORD=secret
#SOLR_SSL_NEED_CLIENT_AUTH=false
#SOLR_SSL_WANT_CLIENT_AUTH=false
```
将注释放掉，然后STORE和PASSWORD改成对应的即可

以上的做法时可以配置https,，但是这样并不是安全的，因为https并没有被互联网认可！
如果你之前为tomcat成功的配置了Https，可以将tomcat.keystore直接应用过来，这样最简单


**以上是我根据自己的环境使用Godaddy来配置的，并不一定适应其他环境**
