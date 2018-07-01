---
title: 为你的网站添加SSL证书，让HTTPS为网站保驾护航（Godaddy）
date: 2017-04-24 22:31:16
categories: 技术
tags:
  - Https
---

　　如果你担心http连接不安全的话，可以参考下面试着安装SSL证书，让http变成https~，我用的是Godaddy的，但无论是通过什么公司去申请的，都需要经过以下步骤
`生成秘钥和CSR->将CSR内容发送给相应公司->验证域名所有权（如果是EV，审核想对复杂）->域名邮箱确认验证->下载证书->安装证书`

<!--more-->
## 在 Tomcat 4.x/5.x/6.x/7.x 中生成和安装 SSL 证书(基于Godaddy)
### 前言
申请 SSL 证书时，你必须通过自己的服务器提供证书签名申请 (CSR)。CSR 包含你的公用密钥，还必须包含与你账户的在线申请表一致的详细信息。当你的申请通过审核并且证书获得颁发后，你需要下载并安装提供的所有文件以完成安装过程。
>以下步骤介绍如何使用 keytool 安装证书，因此你必须在自己的服务器上安装 Java 2 SDK 1.2 或更高版本。

### 在 Tomcat 中生成密钥库和 CSR
在 Tomcat 中生成密钥库和 CSR 的步骤
1、在 keytool 中输入下列命令以生成密钥库： 
```bash
keytool -keysize 2048 -genkey -alias tomcat -keyalg RSA -keystore tomcat.keystore
```
2、在“Password”（密码）中输入密码。默认密码为 changeit。
3、在“Distinguished Information”（识别信息）中输入相关信息：

    - First and Last Name（姓名）— 你要保护的完全限定域名或 URL。如果你申请的是通配符证书，请在要添加通配符的通用名称左边添加星号 (*)，例如 *.coolexample.com。
    - Organizational Unit（组织单位）—（可选）你可以在该字段中输入 DBA 名称（如果有）。
    - Organization（组织）— 你的组织的法律名称。列出的组织必须是证书申请中的域名的合法注册人。如果你是以个人身份注册，请在“Organization”（组织）中输入证书申请人的名字，并在“Organizational - - Unit”（组织单位）中输入 DBA（经营部门）名称。
    - City/Locality（城市/地区）— 你的组织注册或所在的城市，请勿使用缩写。
    - State/Province（州/省）— 你的组织所在的州或省，请勿使用缩写。
    - Country Code（国家/地区）— 你的组织依法注册所在国家/地区的国家/地区代码，以国际标准化组织的两字母格式表示
4、在 keytool 中输入下列命令以生成 CSR：
```bash
keytool -certreq -keyalg RSA -alias tomcat -file csr.csr -keystore tomcat.keystore
```
5、在“Password”（密码）中输入第 2 步中提供的密码。
6、打开 CSR 文件，复制所有文本，包括
----BEGIN NEW CERTIFICATE REQUEST----

和

----END CERTIFICATE REQUEST----
7、将所有文本粘贴到在线申请表中并完成你的申请。
一般就是验证下域名的所有权，通过审核后，页面就会给你一个连接去下载加密证书。


### 在 Tomcat 中安装 SSL
证书获得颁发后，请通过证书管理器下载证书，并将其放到你的密钥库所在的文件夹。然后，使用 keytool 输入下列命令以安装证书。

根证书和中级证书的文件名取决于你的签名算法。

    SHA-1 根证书：gd_class2_root.crt
    SHA-2 根证书：gdroot-g2.crt
    SHA-1 中级证书：gd.intermediate.crt
    SHA-2 中级证书：gdig2.crt
    （仅限 Java 6/7）SHA-2 根证书：gdroot-g2_cross.crt

你还可以从资源库下载证书。
在 Tomcat 中安装 SSL的步骤

通过运行以下命令来安装根证书：
```bash
keytool -import -alias root -keystore tomcat.keystore -trustcacerts -file [根证书的名称]
```
通过运行以下命令来安装中级证书：
```bash
keytool -import -alias intermed -keystore tomcat.keystore -trustcacerts -file [中级证书的名称]
```
通过运行以下命令将颁发的证书安装到密钥库中：
```bash
keytool -import -alias tomcat -keystore tomcat.keystore -trustcacerts -file [颁发的证书的名称]
```
可以参考下面这张图

![对应证书](http://upload-images.jianshu.io/upload_images/3167229-9a410e229564ac81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将 server.xml 文件中的密钥库位置更新为其在 Tomcat 目录中的正确位置。

注意：默认情况下，HTTPS 连接符是放在注释部分中的。要启用 HTTPS，请删除注释标记。
	Tomcat 4.x — 在 Tomcat 4.x 的 server.xml 中更新以下元素：
```xml
clientAuth="false"
protocol="TLS" keystoreFile="/etc/tomcat5/tomcat.keystore"
keystorePass="changeit" />
```
Tomcat 5.x、6.x 和 7.x — 在 Tomcat 5.x、6.x 和 7.x 的 server.xml 中更新以下元素：
```xml
<-- Define a SSL Coyote HTTP/1.1 Connector on port 8443 -->
<Connector 
		   port="8443" maxThreads="200"
		   scheme="https" secure="true" SSLEnabled="true"
		   keystoreFile="[path to your keystore file]" keystorePass="changeit"
		   clientAuth="false" sslProtocol="TLS"/>
```
保存你对 server.xml 所做的更改，然后重新启动 Tomcat 以开始使用你的 SSL。 你的 SSL 证书已安装完毕。

希望能幫助到你
