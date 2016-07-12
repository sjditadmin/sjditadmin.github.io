---
layout: post
title:  "Oracle Flashback之Flashback Query"
date:   2016-07-11 11:06:05
categories: oracle
excerpt: Oracle Flashback之Flashback Query
---
Oracle Flashback之Flashback Query

在Oracle 10g中，Flash back家族分为以下成员：
Flashback Database
Flashback Drop
Flashback Table
Flashback Query(分Flashback Query,Flashback Version Query，Flashback Transaction Query)
下面介绍一下Flashback Query
 
Flashback 是ORACLE 自9i 就开始提供的一项特性，在9i 中利用oracle 查询多版本一致的特点，实现从回滚段中读取表一定时间内操作过的数据，可用来进行数据比对，或者修正意外提交造成的错误数据，该项特性也被称为Flashback Query。
一、Flashback Query
正如前言中所提，Flashback Query 是利用多版本读一致性的特性从UNDO 表空间读取操作前的记录数据！
什么是多版本读一致性
Oracle 采用了一种非常优秀的设计，通过undo 数据来确保写不堵塞读，简单的讲，不同的事务在写数据时，会将数据的前映像写入undo 表空间，这样如果同时有其它事务查询该表数据，则可以通过undo 表空间中数据的前映像来构造所需的完整记录集，而不需要等待写入的事务提交或回滚。
 
flashback query 有多种方式构建查询记录集，记录集的选择范围可以基于时间或基于scn，甚至可以同时查询出记录在undo 表空间中不同事务时的前映象。用法与标准查询非常类似，要通过flashback query 查询undo 中的撤销数据，最简单的方式只需要在标准查询语句的表名后面跟上as of timestamp(基于时间)或as of scn(基于scn)即可。as of timestamp|scn 的语法是自9iR2 后才开始提供支持。
 
这个功能和flashback on/off 以及recyclebin on/off没有关系.
1、As of timestamp 的示例：
SQL> select * from test1;
        ID NAME
---------- ----------
   3764577 1
SQL> select to_char(sysdate,'yy-mm-dd hh24:mi:ss') time from dual;
TIME
-----------------
12-01-13 16:59:29
 
模拟用户误操作，删除数据
SQL> alter table test1 enable row movement; --要将表改为可以允许行移动.
SQL> delete from test1;
SQL> commit;
SQL> select * from test1;
no rows selected
 
查看删除之前的状态：
假设当前距离删除数据已经有5 分钟左右的话：
SQL> select * from test1 as of timestamp sysdate-5/1440;
        ID NAME
---------- ----------
   3764577 1
 
或者：
SQL> select * from test1 as of timestamp to_timestamp('12-01-13 16:59:29','YY-MM-DD hh24:mi:ss');
        ID NAME
---------- ----------
   3764577 1
用Flashback Query恢复之前的数据：
SQL> insert into test1 select * from test1 as of timestamp to_timestamp('12-01-13 16:59:29','YY-MM-DD hh24:mi:ss');
1 row created.
SQL> commit;
SQL> select * from test1;
        ID NAME
---------- ----------
   3764577 1
如上述示例中所表示的，as of timestamp 的确非常易用，但是在某些情况下，我们建议使用as of scn 的方式执行flashback query，比如需要对多个相互有主外键约束的表进行恢复时，如果使用as of timestamp 的方式，可能会由于时间点不统一的缘故造成数据选择或插入失败，通过scn 方式则能够确保记录的约束一致性。
 
2. As of scn 示例
查看SCN:
SELECT dbms_flashback.get_system_change_number FROM dual;
SELECT CURRENT_SCN FROM V$DATABASE;
SQL> SELECT CURRENT_SCN FROM V$DATABASE;
CURRENT_SCN
-----------
    3765554
 
删除数据：
SQL> delete from test1;
1 row deleted.
SQL> commit;
查看删除之前的状态：
SQL> select * from test1 as of scn 3765554;
        ID NAME
---------- ----------
   3764577 1
用Flashback Query恢复之前的数据：
SQL> insert into test1 select * from test1 as of scn 3765554;
1 row created.
SQL> commit;
Commit complete.
SQL> select * from test1;
        ID NAME
---------- ----------
   3764577 1
 
事实上，Oracle 在内部都是使用scn，即使你指定的是as of timestamp，oracle 也会将其转换成scn，系统时间标记与scn 之间存在一张表，即SYS 下的SMON_SCN_TIME
每隔5 分钟，系统产生一次系统时间标记与scn 的匹配并存入sys.smon_scn_time 表，该表中记录了最近1440个系统时间标记与scn 的匹配记录，由于该表只维护了最近的1440 条记录，因此如果使用as of timestamp 的方式则只能flashback 最近5 天内的数据（假设系统是在持续不断运行并无中断或关机重启之类操作的话）。
注意理解系统时间标记与scn 的每5 分钟匹配一次这句话，举个例子，比如scn:3764577,3765348 分别匹配12-01-12 13:52:00 和12-01-12 13:57:00，则当你通过as of timestamp 查询12-01-12 13:52:00或12-01-12 13:57:00 这段时间点内的时间时，oracle 都会将其匹配为scn:3764577到undo 表空间中查找，也就说在这个时间内，不管你指定的时间点是什么，查询返回的都将是12-01-12 13:52:00 这个时刻的数据。
查看SCN 和timestamp 之间的对应关系：
select scn,to_char(time_dp,'yyyy-mm-dd hh24:mi:ss')from sys.smon_scn_time;
 
                    Flashback version Query
相对于Flashback Query 只能看到某一点的对象状态，Oracle 10g引入的Flashback Version Query可以看到过去某个时间段内，记录是如何发生变化的。 根据这个历史，DBA就可以快速的判断数据是在什么时点发生了错误，进而恢复到之前的状态。
先看一个伪列ORA_ROWSCN.  所谓的伪列，就是假的，不存在的数据列，用户创建表时虽然没有指定，但是Oracle为了维护而添加的一些内部字段，这些字段可以像普通文件那样的使用。
最熟悉的伪列就是ROWID， 它相当于一个指针，指向记录在磁盘上的位置。ORA_ROWSCN 是Oracle 10g 新增的，暂且把它看作是记录最后一次被修改时的SCN。Flashback Version Query 就是通过这个伪列来跟踪出记录的变化历史。
举个例子：
SQL> select * from test1;
        ID NAME
---------- ----------
   3764577 1
SQL> insert into test1 values(dbms_flashback.get_system_change_number,'2');
1 row created.
SQL> select * from test1;
        ID NAME
---------- ----------
   3764577 1
   3765736 2
SQL> commit;
SQL> select ora_rowscn,id,name from test1;
ORA_ROWSCN         ID NAME
---------- ---------- ----------
   3765611    3764577 1
   3765611    3765736 2
获取更多的历史信息
SQL> Select versions_xid,versions_startscn,versions_endscn,
  2  DECODE(versions_operation,'I','Insert','U','Update','D','Delete', 'Original') "Operation",
  3  id,name from test1 versions between scn minvalue and maxvalue;
VERSIONS_XID     VERSIONS_STARTSCN VERSIONS_ENDSCN Operatio         ID NAME
---------------- ----------------- --------------- -------- ---------- ----------
0400210080030000           3765611                 Insert      3764577 1
0400210080030000           3765611                 Delete      3764577 1
03002A0049040000           3765520         3765611 Insert      3764577 1
03002A0049040000           3765520                 Delete      3765348 A
                                           3765520 Original    3765348 A
或者
SQL>select xid,commit_scn,commit_timestamp,operation,undo_sql
from flashback_transaction_query q where q.xid in(select versions_xid from B versions between scn 3764577 and 3765348 );
下面我们来讲下伪列，Flashback Version Query 技术其实有很多伪列，但是ORA_ROWSCN是最重要。它记录的是最后一次被修改时的SCN， 注意是被提交的修改。如果没有提交，这个伪列不会发生变化。
ORA_ROWSCN 缺省是数据块级别的，也就是一个数据块内的所有记录都是一个ORA_ROWSCN，数据块内任意一条记录被修改，这个数据库块内的所有记录的ORA_ROWSCN都会同时改变。上例的查询结果以证明。
不过我们可以在建表时使用关键字rowdependencies， 可以改变这种缺省行为，使用这个关键字后，每条记录都有自己的ORA_ROWSCN。
举例：
SQL> create table test1(id int,seq int) rowdependencies;
Table created.
SQL> insert into test1 values(dbms_flashback.get_system_change_number,1);
1 row created.
SQL> insert into test1 values(dbms_flashback.get_system_change_number,2);
1 row created.
SQL> commit;
Commit complete.
SQL> insert into test1 values(dbms_flashback.get_system_change_number,3);
1 row created.
SQL> commit;
Commit complete.
SQL> select ora_rowscn,id,seq from test1;
ORA_ROWSCN         ID        SEQ
---------- ---------- ----------
   3769182    3769179          1
   3769182    3769180          2
   3769186    3769184          3
 
说明是最后一次被修改时的SCN，如果没有提交，是不会变的。
 
                    Flashback Transaction Query
Flashback Transaction Query也是使用UNDO信息来实现。利用这个功能可以查看某个事务执行的所有变化，它需要访问flashback_transaction_query 视图，这个视图的XID列代表事务ID，利用这个ID可以区分特定事务发生的所有数据变化。
示例：
SQL> truncate table test1;
Table truncated.
SQL> select current_scn from v$database;
CURRENT_SCN
-----------
    3769319
SQL> insert into test1 values(dbms_flashback.get_system_change_number,1);
1 row created.
SQL> commit;
Commit complete.
SQL> select current_scn from v$database;
CURRENT_SCN
-----------
    3769333
 
查看视图，每个事务都对应相同的XID
SQL> Select xid,operation,commit_scn,undo_sql from flashback_transaction_query where xid in (
  2  Select versions_xid from test1 versions between scn minvalue and maxvalue);
 
XID              OPERATION                        COMMIT_SCN
---------------- -------------------------------- ----------
UNDO_SQL
-----------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------
02000E009C040000 INSERT                              3769330
delete from "YORKER"."TEST1" where ROWID = 'AAANlOAAEAAAAL2AAA';
02000E009C040000 BEGIN                               3769330
或者
select xid,commit_scn,commit_timestamp,operation,undo_sql
from flashback_transaction_query q where q.xid in
(select versions_xid from test1 versions between scn 3769319 and 3769333);