---
layout: post
title:  "Oracle Rman 命令详解"
date:   2016-07-11 11:06:05
categories: oracle
excerpt: oracle rman 
---
* content
{:toc}

## Oracle Rman 命令详解

## 一、list常用命令总结备忘

list命令列出控制文件、RMAN恢复目录中备份信息， 是我们对所有可见的数据库备份文件的一个最直观的了解的方法

list incarnation;

list backup summary;

list backup of database summary;

list backup of tablespace summary;

list backup of datafile n,n summary;

list archivelog all summary;

list backup by file;

list backup;

list expired backup;

list copy;

list backup of spfile;

list backup of controlfile;

list backup datafile n,n,n;

list backup tablespace tablespace_name;

list backup of archivelog all;

list backup of archivelog from scn …；

list backup of archivelog until scn …；

list backup of archivelog from sequence ;

list backup of archivelog until time 'sysdate-10';

list backup of archivelog {all, from, high, like, logseq, low, scn, sequence, time, until};



### 1、List 当前RMAN所备份的数据库：

RMAN> list incarnation;

汇总查询：--如果备份文件多的话多用这两个list命令可以对备份文件有个总体了解。

（1）list backup summary; --概述可用的备份

B 表示backup

A 表示Archivelog、 F 表示full backup、 0,1,2 表示incremental level备份

A 表示可用AVALIABLE、 X 表示EXPIRED

这个命令可以派生出很多类似命令，例如

list backup of database summary

list backup of archivelog all summary

list backup of tablespace users summary;

list backup of datafile n,n,n summary

这些命令可以让我们对已有的备份文件有一个整体，直观的了解。


（2）list backup by file;--按照文件类型分别列出

分别为：数据文件列表、归档日志列表、控制文件列表、SPFILE列表


（3）list backup;

这个命令列出已有备份集的详细信息。


（4）list expired backup;

列出过期的备份文件


（5）list copy;

列出copy文件

list copy of database;

list copy of controlfile;

list copy of tablespace users;

list copy of datafile n,n,n;

list copy of archivelog all;

list copy of archivelog from scn 10000;

list copy of archivelog until sequence 12;


###2、List 相关文件的信息

list backup of {archivelog, controlfile, database, datafile, spfile, tablespace};

list backup of database; --full,incremental,tablespace,datafile

（1）服务器参数文件：

list backup of spfile;

（2）控制文件：

list backup of controlfile;

（3）数据文件：

list backup of datafle n,n,n,n;

（4）表空间：

list backup of tablespace tablespace_name;--表空间对应的backup

（5）归档日志：

list backup of archivelog {all, from, high, like, logseq, low, scn, sequence, time, until};

list backup of archivelog all;

list backup of archivelog until time 'sysdate-1';

list backup of archivelog from sequence 10;

list backup of archivelog until sequence 10;

list backup of archivelog from scn 10000;

list backup of archivelog until scn 200000;

list archivelog from scn 1000;

list archivelog until scn 2000;

list archivelog from sequence 10;

list archivelog until sequence 12;




## 二、report常用命令总结备忘

report用于判断数据库当前可恢复状态、以及数据库已有备份的信息。

最常使用的是report obsolete; report schema;

report {device, need, obsolete, schema, unrecoverable}

report schema;

report obsolete;

report unrecoverable;

report need backup;

report need backup days=3; --报告最近3天内没有备份的文件

report need backup redundancy=3; --报告冗余次数小于3的数据文件。

report need backup recovery window of 2 days;

（1）report schema;

报告数据库模式


（2）report obsolete;

报告已丢弃的备份集（配置了保留策略）。


（3）report unrecoverable;

报告当前数据库中不可恢复的数据文件（即没有这个数据文件的备份、或者该数据文件的备份已经过期）


（4）report need backup;

报告需要备份的数据文件（根据条件不同）

report need backup days=3;

--最近三天没有备份的数据文件（如果出问题的话，这些数据文件将需要最近3天的归档日志才能恢复）

report need backup incremental=3;

--需要多少个增量备份文件才能恢复的数据文件。（如果出问题，这些数据文件将需要3个增量备份才能恢复）

report need backup redundancy=3;

--报告出冗余次数小于3的数据文件

--例如数据文件中包含2个数据文件system01.dbf和users01.dbf.

--在3次或都3次以上备份中都包含system01.dbf这个数据文件，而users01.dbf则小于3次

--那么，报告出来的数据文件就是users01.dbf

--即，报告出数据库中冗余次数小于 n 的数据文件

report need backup recovery window of 2 days;

--报告出恢复需要2天归档日志的数据文件





## 三、backup常用命令总结备忘

1、设置备份标记

backup database tag='full_bak1';

注：每个标记必须唯一，相同的标记可以用于多个备份只还原最新的备份。


2、设置备份集大小（一次备份的所有结果为一个备份集，要注意备份集大小）

backup database maxsetsize=100m tag='datafile1';

注：maxsetsize限定备份集的大小。所以必须大于数据库总数据文件的大小，否则会报错。

RMAN-06183: datafile or datafile copy larger than MAXSETSIZE: file# 1 /data/oradata/system01.dbf


3、设置备份片大小（磁带或文件系统限制）

run {

allocate channel c1 type disk maxpicecsize 100m format '/data/backup/full_0_%U_%T';

backup database tag='full_0';

release channel c1;

}

可以在allocate子句中设定每个备份片的大小，以达到磁带或系统限制。

也可以在configure中设置备份片大小。

Configure channel device type disk maxpiecesize 100 m;

configure channel device type disk clear;


4、备份集的保存策略

backup database keep forever;                  --永久保留备份文件

backup database keep until time='sysdate+30'; --保存备份30天


5、重写configure exclude命令

backup databas noexclude keep forever tag='test backup';


6、检查数据库错误

backup validate database;

使用RMAN来扫描数据库的物理/逻辑错误，并不执行实际备份。


7、跳过脱机，不可存取或只读文件

backup database skip readonly;

backup database skip offline;

backup database skip inaccessible;

backup database ship readonly skip offline ship inaccessible;


8、强制备份

backup database force;


9、基于上次备份时间备份数据文件

1>只备份添加的新数据文件

backup database not backed up;

2>备份"在限定时间周期内"没有被备份的数据文件

backup database not backed up since time='sysdate-2';


10、备份操作期间检查逻辑错误

backup check logical database;

backup validate check logical database;


11、生成备份副本

backup database copies=2;


12、备份控制文件

backup database device type disk includ current controlfile;




## 四、configure常用命令总结备忘

1、显示当前的配置信息

（1）RMAN> show all;

RMAN 配置参数为：

CONFIGURE RETENTION POLICY TO REDUNDANCY 1; # default

CONFIGURE BACKUP OPTIMIZATION OFF; # default

CONFIGURE DEFAULT DEVICE TYPE TO DISK; # default

CONFIGURE CONTROLFILE AUTOBACKUP OFF; # default

CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '%F'; # default

CONFIGURE DEVICE TYPE DISK PARALLELISM 1 BACKUP TYPE TO BACKUPSET; # default

CONFIGURE DATAFILE BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default

CONFIGURE ARCHIVELOG BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default

CONFIGURE MAXSETSIZE TO UNLIMITED; # default

CONFIGURE ENCRYPTION FOR DATABASE OFF; # default

CONFIGURE ENCRYPTION ALGORITHM 'AES128'; # default

CONFIGURE ARCHIVELOG DELETION POLICY TO NONE; # default

CONFIGURE SNAPSHOT CONTROLFILE NAME TO 'D:/ORACLE/PRODUCT/10.2.0/DB_1/DATABASE/S

NCFDBA.ORA'; # default


（2）查询RMAN设置中非默认值：

SQL> select name,value from v$rman_configuration;


2、常用的configure选项

（1）保存策略 （retention policy）

configure retention policy to recovery window of 7 days;

configure retention policy to redundancy 5;

configure retention policy clear;

CONFIGURE RETENTION POLICY TO NONE;

第一种recover window是保持所有足够的备份，可以将数据库系统恢复到最近七天内的任意时刻。任何超过最近七天的数据库备份将被标记为obsolete。

第二种redundancy 是为了保持可以恢复的最新的5份数据库备份，任何超过最新5份的备份都将被标记为redundancy。它的默认值是1份。

第三四：NONE 可以把使备份保持策略失效，Clear 将恢复默认的保持策略

一般最安全的方法是采用第二种保持策略。


（2）备份优化 backup optimization

configure backup optimization on;

configure backup optimization off;

configure backup optimization clear;

默认值为关闭，如果打开，rman将对备份的数据文件及归档等文件进行一种优化的算法。


（3）默认设备 default device type

configure default device type to disk;

configure default device type to stb;

configure default device type clear;

是指定所有I/O操作的设备类型是硬盘或者磁带，默认值是硬盘

磁带的设置是CONFIGURE DEFAULT DEVICE TYPE TO SBT;


（4）控制文件 controlfile

configure controlfile autobackup on;

configure controlfile autobackup format for device type disk to '/cfs01/backup/conf/conf_%F';

configure controlfile autobackup clear;

configrue controlfile autobackup format for device type disk clear;

configrue snapshot controlfile name to '/cfs01/backup/snapcf/scontrofile.snp';

--是配置控制文件的快照文件的存放路径和文件名，这个快照文件是在备份期间产生的，用于控制文件的读一致性。

configrue snapshot controlfile name clear;

强制数据库在备份文件或者执行改变数据库结构的命令之后将控制文件自动备份，默认值为关闭。这样可以避免控制文件和catalog丢失后，控制文件仍然可以恢复。


（5）并行数（通道数） device type disk|stb pallelism n;

configure device type disk|stb parallelism 2;

configure device type disk|stb clear; --用于清除上面的信道配置

configure channel device type disk format 'e/:rmanback_%U';

configure channel device type disk maxpiecesize 100m

configure channel device type disk rate 1200K

configure channel 1 device type disk format 'e/:rmanback_%U';

configure channel 2 device type disk format 'e/:rmanback_%U';

configure channel 1 device type disk maxpiecesize 100m

配置数据库设备类型的并行度。


（6）生成备份副本 datafile|archivelog backup copies

configure datafile backup copies for device type disk|stb to 3;

configure archivelog backup copies for device type disk|stb to 3;

--是设置数据库的归档日志的存放设备类型

configure datafile|archivelog backup copies for device type disk|stb clear

BACKUP DEVICE TYPE DISK DATABASE

FORMAT '/disk1/backup/%U', '/disk2/backup/%U', '/disk3/backup/%U';

是配置数据库的每次备份的copy数量，oracle的每一次备份都可以有多份完全相同的拷贝。


（7）排除选项 exclude

configure exclude for tablespace 'users';

configrue exclude clear;

此命令用于将指定的表空间不备份到备份集中， 此命令对只读表空间是非常有用的。


（8）备份集大小 maxsetsize

configure maxsetsize to 1G|1000M|1000000K|unlimited;

configure maxsetsize clear;


（9）其它选项 auxiliary

CONFIGURE AUXNAME FOR DATAFILE 1 TO '/oracle/auxfiles/aux_1.f';

CONFIGURE AUXNAME FOR DATAFILE 2 TO '/oracle/auxfiles/aux_2.f';

CONFIGURE AUXNAME FOR DATAFILE 3 TO '/oracle/auxfiles/aux_3.f';

CONFIGURE AUXNAME FOR DATAFILE 4 TO '/oracle/auxfiles/aux_4.f';

-

CONFIGURE AUXNAME FOR DATAFILE 1 CLEAR;

CONFIGURE AUXNAME FOR DATAFILE 2 CLEAR;

CONFIGURE AUXNAME FOR DATAFILE 3 CLEAR;

CONFIGURE AUXNAME FOR DATAFILE 4 CLEAR;

Rman的format格式中的%

%c 备份片的拷贝数

%d 数据库名称

%D 位于该月中的第几天 （DD）

%M 位于该年中的第几月 （MM）

%F 一个基于DBID唯一的名称，这个格式的形式为c-IIIIIIIIII-YYYYMMDD-QQ,其中IIIIIIIIII为该数据库的DBID，YYYYMMDD为

日期，QQ是一个1-256的序列

%n 数据库名称，向右填补到最大八个字符

%u 一个八个字符的名称代表备份集与创建时间

%p 该备份集中的备份片号，从1开始到创建的文件数

%U 一个唯一的文件名，代表%u_%p_%c

%s 备份集的号

%t 备份集时间戳

%T 年月日格式（YYYYMMDD）
