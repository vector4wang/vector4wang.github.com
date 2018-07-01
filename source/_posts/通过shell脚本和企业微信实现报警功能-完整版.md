---
title: 通过shell脚本和企业微信实现报警功能(完整版)
date: 2018-03-18 00:16:27
categories: 技术
tags: 
	- shell
	- 监控
	- 微信
	- 工具
---

使用微信实现的服务器监控报警的完整脚本

<!--more-->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=224326&auto=1&height=66"></iframe>

最终效果如图(GIF太大了 就截了两张图)
![1.png](https://upload-images.jianshu.io/upload_images/3167229-095bd9dba9dce8c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2.png](https://upload-images.jianshu.io/upload_images/3167229-75106b9eca39a2a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```shell
#!/bin/sh

expireTime=7200

dbFile="db.json"

corpid=xxx
corpsecret=xxx

touser="xxx"
toparty="xxx"
agentid="xxx"
content="服务器快崩了，你还在这里吟诗作对？"

# s 为秒，m 为 分钟，h 为小时，d 为日数  
interval=1s

## 发送报警信息
sendMsg(){
	if [ ! -f "$dbFile" ];then
			touch "$dbFile"
	fi

	# 获取token
	req_time=`jq '.req_time' $dbFile`
	current_time=$(date +%s)
	refresh=false
	if [ ! -n "$req_time" ];then
			refresh=true
	else
			if [ $((current_time-req_time)) -gt $expireTime ];then
				refresh=true
			fi
	fi
	if $refresh ;then
		req_access_token_url=https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=$corpid\&corpsecret=$corpsecret
		access_res=$(curl -s -G $req_access_token_url | jq -r '.access_token')

		## 保存文件
		echo "" > $dbFile
		echo -e "{" > $dbFile
		echo -e "\t\"access_token\":\"$access_res\"," >> $dbFile
		echo -e "\t\"req_time\":$current_time" >> $dbFile
		echo -e "}" >> $dbFile

		echo ">>>刷新Token成功<<<"
	fi 

	## 发送消息
	msg_body="{\"touser\":\"$touser\",\"toparty\":\"$toparty\",\"msgtype\":\"text\",\"agentid\":$agentid,\"text\":{\"content\":\"$content\"}}"
	access_token=`jq -r '.access_token' $dbFile`
	req_send_msg_url=https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=$access_token
	req_msg=$(curl -s -H "Content-Type: application/json" -X POST -d $msg_body $req_send_msg_url | jq -r '.errmsg')

	echo "触发报警发送动作，返回信息为：" $req_msg	
	
}


loopMonitor(){
    echo 'loop'
    flag=`uptime | awk '{printf "%.2f\n", $11 "\n"}'`

	
	# 0.7 这个阈值可以视情况而定，如cpu核数为n，则可以设置为0.7 * n  具体视情况而定
    c=$(echo "$flag > 0.7" | bc)

	echo ">>>>>>>>>>>>>>>>>>`date`<<<<<<<<<<<<<<<<<<"
	free -m | awk 'NR==2{printf "Memory Usage: %s/%sMB (%.2f%%)\n", $3,$2,$3*100/$2 }'
	df -h | awk '$NF=="/"{printf "Disk Usage: %d/%dGB (%s)\n", $3,$2,$5}'
	uptime | awk '{printf "CPU Load: %.2f\n", $11 "\n"}'
	echo ">>>>>>>>>>>>>>>>>>end<<<<<<<<<<<<<<<<<<"
	
    if [ $c -eq 1  ];then
         sendMsg
	fi
}


while true; do
    loopMonitor
    sleep $interval
done
```

让CPU达到100%的脚本
```bash
for i in `seq 1 $(cat /proc/cpuinfo |grep "physical id" |wc -l)`; do dd if=/dev/zero of=/dev/null & done
```
欢迎提[PR](https://github.com/vector4wang/shell-tools/blob/master/%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%9B%91%E6%8E%A7%E5%BE%AE%E4%BF%A1%E6%8A%A5%E8%AD%A6/wx-monitor-release.sh)

CSDN：[http://blog.csdn.net/qqhjqs?viewmode=list](https://link.jianshu.com/?t=http%3A%2F%2Fblog.csdn.net%2Fqqhjqs%3Fviewmode%3Dlist)
博客：[http://vector4wang.tk/](https://link.jianshu.com/?t=http%3A%2F%2Fvector4wang.tk%2F)
简书：[https://www.jianshu.com/u/223a1314e818](https://www.jianshu.com/u/223a1314e818)
Github:[https://github.com/vector4wang](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fvector4wang)
Gitee:[https://gitee.com/backwxc](https://link.jianshu.com/?t=https%3A%2F%2Fgitee.com%2Fbackwxc)
