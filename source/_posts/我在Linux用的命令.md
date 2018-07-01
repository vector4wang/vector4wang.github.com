---
title: 我在Linux用的命令
date: 2017-04-24 22:31:16
categories: 技术
tags:
  - Linux
  - shell
---

Linux相关命令
<!--more-->

### 控制应用进程的生死
```bash
## 查看进程
 ps -ef|grep tomcat
## 杀死进程
kill -9 xxxx
```
### 监控日志
```bash
## 实时监控日志
tail -f log.log
## 查看最后n行记录
tail -n log.log
```
### 查看文件
```bash
cat text.txt
more text.txt
less text.txt
## 可编辑
vi text.txt
```
### 下载
```bash
wget https://xxxxxxxxxxxxxxxxxxxxxxxx
```
### 使用secureCRT上传下载
```bash
## 上传
rs
## 下载
sz filepath
```
