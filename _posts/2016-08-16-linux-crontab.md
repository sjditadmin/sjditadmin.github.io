---
layout: post
title:  "crontab 使用说明"
date:   2016-08-16
categories: Linux
excerpt: crontab 使用说明 
---

安装crond服务
`yum install crontabs`
服务操作说明：
`/sbin/service crond start` //启动服务
`/sbin/service crond stop` //关闭服务
`/sbin/service crond restart` //重启服务
`/sbin/service crond reload `//重新载入配置
加入开机自动启动：
`/sbin/chkconfig --level 35 crond on` 

Crontab 命令_**参数**_示意图
![](https://camo.githubusercontent.com/df6a8fa3e0ed7249ae137386de993af9597b2172/68747470733a2f2f696d672e616c6963646e2e636f6d2f7470732f544231345178334d585858585858415856585858585858585858582d3532332d3431392e6a7067)
