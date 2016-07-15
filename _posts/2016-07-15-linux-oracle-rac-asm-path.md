---
layout: post
title:  "Linux多路径问题造成rac asm无法识别磁盘组"
date:   2016-07-15
categories: oracle
excerpt: Linux多路径问题造成rac asm无法识别磁盘组 
---


## Linux多路径问题造成rac asm无法识别磁盘组

* 重启后，无法在当前启动实例挂载磁盘组USDATA01
该磁盘组对应的裸盘为/dev/raw/raw4。这个裸盘正常对应多路径的mpathf盘（可使用ls -l /dev/mapper查看对应磁盘号）
* 移除当前错误的裸盘并对应正确的多路径盘符（以root用户操作）
    raw /dev/raw/raw4 0 0
    raw /dev/raw/raw4 253 2
* 启动这个磁盘组（以grid用户操作）
    su - grid 
    sqlplus / as sysasm
    sql>alter diskgroup usdata01 mount;
    sql>exit
* 查看实例状态（应该磁盘挂载后，自动启动实例）
    $srvctl status instance -d racdb -n rac01
* 如果没有启动，使用下面命令启动实例
    $srvctl start instance -d racdb -n rac01
