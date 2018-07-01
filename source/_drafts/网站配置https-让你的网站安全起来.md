---
title: '网站配置https,让你的网站安全起来'
date: 2016-08-08 22:30:21
tags:
  - 日常撸码
---

现在是互联网时代，网上的链接各式各样，花花绿绿，层出不穷。那么问题来了，当多如牛毛的链接摆在你面前，你能知道哪个链接是安全的，哪个链接后面隐藏着重磅炸弹吗？Https就完美的解决了这一问题

<!-- more -->

对于堕入牛毛的链接，如果其中有** Https **标志的，你大可放心去点击，这种连接相当的安全~~~。

>HTTPS (also called HTTP over TLS,HTTP over SSL,and HTTP Secure) is a protocol for secure communication over a computer network which is widely used on the Internet. HTTPS consists of communication over Hypertext Transfer Protocol (HTTP) within a connection encrypted by Transport Layer Security or its predecessor, Secure Sockets Layer. The main motivation for HTTPS is authentication of the visited website and protection of the privacy and integrity of the exchanged data.

>In its popular deployment on the internet, HTTPS provides authentication of the website and associated web server with which one is communicating, which protects against man-in-the-middle attacks. Additionally, it provides bidirectional encryption of communications between a client and server, which protects against eavesdropping and tampering with or forging the contents of the communication.In practice, this provides a reasonable guarantee that one is communicating with precisely the website that one intended to communicate with (as opposed to an impostor), as well as ensuring that the contents of communications between the user and site cannot be read or forged by any third party.

>Historically, HTTPS connections were primarily used for payment transactions on the World Wide Web, e-mail and for sensitive transactions in corporate information systems. In the late 2000s and early 2010s, HTTPS began to see widespread use for protecting page authenticity on all types of websites, securing accounts and keeping user communications, identity and web browsing private.


>HTTPS （也称为 HTTP tls，HTTP ssl，和 HTTP Secure) 是安全通信协议通过计算机网络的广泛应用在互联网上。HTTPS 包含通信通过超文本传输协议 (HTTP) 内由传输层安全性或其前身，安全套接字层加密的连接。HTTPS 的主要动机是访问网站的身份验证和保护隐私和交换数据的完整性。

>在互联网上的流行部署 HTTPS 提供身份验证的网站和关联的 web 服务器与一个通信，其中保护人在中东的攻击。此外，它还提供双向加密的客户端和服务器，可防止窃听和篡改或伪造的通信内容之间的通信。在实践中，这提供合理的保证，其中一个沟通与精确的网站，其中一份拟与沟通 （而不是冒名顶替者），并确保不能读或伪造任何第三方的用户与网站之间的通讯内容。

>从历史上看，HTTPS 连接，主要用于电子邮件万维网上的付款交易记录和敏感的事务企业信息系统。在 2000 年后期和年代，HTTPS 开始看到为保护页面上所有类型的网站的真实性，保护帐户的安全和保持用户通信、 身份和 web 浏览私人的广泛使用。

以上是wiki百科的介绍，介绍的很详细也很专业，对于有了https的网站，用户在使用的时候，会给用户一种绝对的安全感，现在来介绍一下https的配置。

https级别
