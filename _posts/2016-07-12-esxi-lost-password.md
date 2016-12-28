---
layout: post
title:  "esxi忘记密码修复"
date:   2016-07-11 11:06:05
categories: vmware
excerpt: esxi忘记密码修复 
---

##esxi 密码忘记

重新安装esxi吧，不要覆盖datastore，这样比较简单。
如果非要恢复密码

1、用linux启动光盘，如rhel5的安装光盘或knoppix启动

2、到命令行下，运行mount /dev/sda5 /mnt/sda5

cp /mnt/sda5/stage.tgz /tmp/.

cd /tmp

    tar xvfz stage.tgz
    tar xvfz local.tgz
5. vi /tmp/etc/passwd
把类似root:x:143434343:12232:9:99999:7
这一行中的143434343给清除
6. rm -f stage.tgz local.tgz
    tar czvf local.tgz etc
    tar czvf stage.tgz local.tgz
    cp local.tgz /mnt/sda5/.
7. 重启esxi即可重新设置root密码
ESXI重新安装 密码自动覆盖
里面的虚拟机不会丢失