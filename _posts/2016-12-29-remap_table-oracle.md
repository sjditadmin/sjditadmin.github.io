
---
layout: post
title:  "remap_table Parameter of Data-Pump in Oracle 11g"
date:   2016-12-29
categories: oracle
excerpt: remap_table Parameter of Data-Pump in Oracle 11g

---
## remap_table Parameter of Data-Pump in Oracle 11g
Oracle 11g datapump provide a new feature remap_table command to remap the table data to new table name on target database.we can use the REMAP_TABLE parameter to rename entire tables.
Syntax :
REMAP_TABLE=[schema.]old_tablename[.partition]:new_tablename .

In 10g datapump ,we use the REMAP_SCHEMA parameter to remap the schema name during the import or we use the FROMUSER and TOUSER parameters in the original import . There is no parameter to remap table names . This means that Import DataPump can only import data into a table with the same name as the original table.

If we have to import a table data having same structure into a database i.e, it is containing the table with same name then we have to perform it in two ways .

I.) Rename the original source table temporarily : 

II.) If the original source table cannot be rename then follow the below steps :
a.) import the dump into another schemas.
b.) rename the table name.
c.) Again export the table .
d.) Finally import the table name .

Remap_table allows us to rename tables during an import operation . Here is demo of the remap_table :

Here , we will create a table and take export of it and import it in the same schemas . In this scenario we have table name "test" and we will rename it as "newtest".

1.) Create a table  "test"

SQL> conn hr/hr@noida
Connected.
SQL> create table test(id number);
Table created.
SQL> insert into test values (1);
1 row created.
SQL> insert into test values (2);
1 row created.
SQL> insert into test values (3);
1 row created.
SQL> insert into test values (4);
1 row created.
SQL> commit;
Commit complete.

SQL> select * from test;
        ID
----------
         1
         2
         3
         4

2.) Export the table "test"

SQL> host expdp hr/hr@noida    dumpfile=hr_test.dmp    logfile=hrtestlog.log     tables=test

Export: Release 11.2.0.1.0 - Production on Fri May 27 11:20:43 2011
Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.
Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
Starting "HR"."SYS_EXPORT_TABLE_01":  hr/********@noida dumpfile=hr_test.dmp logfile=hrtestlog.log tables=test
Estimate in progress using BLOCKS method...
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
Total estimation using BLOCKS method: 64 KB
Processing object type TABLE_EXPORT/TABLE/TABLE
. . exported "HR"."TEST"                                 5.031 KB       4 rows
Master table "HR"."SYS_EXPORT_TABLE_01" successfully loaded/unloaded
******************************************************************************
Dump file set for HR.SYS_EXPORT_TABLE_01 is:
  D:\APP\NEERAJS\ADMIN\NOIDA\DPDUMP\HR_TEST.DMP
Job "HR"."SYS_EXPORT_TABLE_01" successfully completed at 11:21:16

Since,we have the dump of the table "test". We import into hr schemas with new name  "newtest"

3.) Import the dump with remap_table Parameter

SQL>host impdp hr/hr@noida  dumpfile=hr_test.dmp logfile=imphrtestlog.log remap_table=hr.test:newtest
Import: Release 11.2.0.1.0 - Production on Fri May 27 11:22:11 2011
Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.
Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
Master table "HR"."SYS_IMPORT_FULL_04" successfully loaded/unloaded
Starting "HR"."SYS_IMPORT_FULL_04":  hr/********@noida dumpfile=hr_test.dmp logfile=imphrtestlog.log remap_table=hr.test:newtest
Processing object type TABLE_EXPORT/TABLE/TABLE
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
. . imported "HR"."NEWTEST"                              5.031 KB       4 rows
Job "HR"."SYS_IMPORT_FULL_04" successfully completed at 11:22:25

Since the job is successfully completed .So we check the imported table i.e, "newtest"

SQL> select * from tab;
TNAME                                     TABTYPE                         CLUSTERID
----------------------                       ------------                         ----------------
COUNTRIES                               TABLE
DEPARTMENTS                         TABLE
EMPLOYEES                              TABLE
EMP_DETAILS_VIEW                VIEW
JOBS                                          TABLE
JOB_HISTORY                           TABLE
LOCATIONS                               TABLE
NEWTEST                                  TABLE
REGIONS                                    TABLE
SYS_IMPORT_FULL_01             TABLE
SYS_IMPORT_FULL_02             TABLE
SYS_IMPORT_FULL_03             TABLE
TEST                                            TABLE

13 rows selected.

SQL> select * from newtest;
        ID
----------
         1
         2
         3
         4

Note : Only objects created by the Import will be remapped. In particular, preexisting tables will not be remapped if TABLE_EXISTS_ACTION is set to TRUNCATE or APPEND