---
layout: post
title:  "Oracle 锁查询及解锁"
date:   2016-07-11 11:06:05
categories: oracle
excerpt: Oracle 锁查询及解锁
---

1.查询被锁表

select   p.spid,a.serial#, c.object_name,b.session_id,b.oracle_username,b.os_user_name   from   v$process   p,v$session   a,   v$locked_object   b,all_objects   c   where   p.addr=a.paddr   and   a.process=b.process   and   c.object_id=b.object_id ;

2.解锁

alter system kill session  'sid,serial#';

例：
alert system kill session '20541,173';

3.查看那个用户那个进程照成死锁

select b.owner,b.object_name,l.session_id,l.locked_mode from v$locked_object l, dba_objects b where b.object_id=l.object_id;

4.查看连接的进程 

SELECT sid, serial#, username, osuser FROM v$session; 
