---
layout: post
title:  "Oracle 之 spool"
date:   2016-07-11 11:06:05
categories: oracle
excerpt: oracle spool 
---
SPOOL可以把Oracle客户端SQLPLUS的输出导入到一个文本中，可以导出html、CSV等形式，其语法如下：

spool <filename> [rep/append]

屏幕输出保留到指定文件中，如果文件存在想替换内容使用replace,追加内容到文件中使用append
关闭并把输出发送到系统打印机打印用spool out，不过这个命令在某些系统不能用
关闭屏幕内容输出到文件使用spool off

比如我们想要把Oracle各表空间的使用情况输出为HTML格式的报表：

    SET MARKUP HTML ON SPOOL ON pre off entmap off
    SET ECHO OFF
    SET TERMOUT OFF
    SET TRIMOUT OFF
    set feedback off
    set heading on
    set linesize 200
    set pagesize 10000
    col tablespace_name format a15
    col total_space format a10
    col free_space format a10
    col used_space format a10
    col used_rate format 99.99
    spool /home/oracle/test.html
    select a.tablespace_name,a.total_space_Mb||'m' total_space,b.free_space_Mb||'m'
    free_space,a.total_space_Mb-b.free_space_Mb||'m' used_space,
    (1-(b.free_space_Mb/a.total_space_Mb))*100 used_rate,a.total_blocks,b.free_blocks from
    (select tablespace_name,sum(bytes)/1024/1024 total_space_Mb,sum(blocks) total_blocks from dba_data_files
    group by tablespace_name) a,
    (select tablespace_name, sum((bytes)/1024/1024) free_space_Mb,sum(blocks) free_blocks from dba_free_space
    group by tablespace_name) b
    where a.tablespace_name=b.tablespace_name order by used_rate desc;
    spool off

---
   
 最终导出结果如下：

 ![](http://i.imgur.com/horuUZQ.jpg) 