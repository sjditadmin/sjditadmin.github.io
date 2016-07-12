---
layout: post
title:  "Oracle轻松取得建表和索引的DDL语句"
date:   2016-07-12 10:31:58 
categories: Oracle
excerpt:  获取oracle建表的dll,获取oracle建index的dll
---
* content
{:toc}



## 序 ##


---
## 获得单个表和索引DDL语句的方法：##
    
    set heading off;
    
    set echo off;
    
    Set pages 999;
    
    set long 90000;
    
    spool get_single.sql
    
    select dbms_metadata.get_ddl('TABLE','SZT_PQSO2','SHQSYS') from dual;
    
    select dbms_metadata.get_ddl('INDEX','INDXX_PQZJYW','SHQSYS') from dual;　
    
    spool off; 
    



下面是输出。我们只要把建表/索引语句取出来在后面加个分号就可以直接运行了。




    SQL> select dbms_metadata.get_ddl('TABLE','SZT_PQSO2','SHQSYS') from dual;　
    
    CREATE TABLE "SHQSYS"."SZT_PQSO2" 
    
    ( "PQBH" VARCHAR2(32) NOT NULL ENABLE, 
    
    "ZJYW" NUMBER(10,0), 
    
    "CGSO" NUMBER(10,0) NOT NULL ENABLE, 
    
    "SOLS" VARCHAR2(17), 
    
    "SORQ" VARCHAR2(8), 
    
    "SOWR" VARCHAR2(8), 
    
    "SOCL" VARCHAR2(6), 
    
    "YWHM" VARCHAR2(10), 
    
    "YWLX" VARCHAR2(6) 
    
    ) PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 NOCOMPRESS LOGGING 
    
    STORAGE(INITIAL 1048576 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645 
    
    PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1 BUFFER_POOL DEFAULT) 
    
    TABLESPACE "DATA1" 

    SQL> select dbms_metadata.get_ddl
    
    ('INDEX','INDXX_PQZJYW','SHQSYS') from dual;
    
    CREATE INDEX "SHQSYS"."INDXX_PQZJYW" ON "SHQSYS"."SZT_PQSO2" ("ZJYW") 
    
    PCTFREE 10 INITRANS 2 MAXTRANS 255 
    
    STORAGE(INITIAL 1048576 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645 
    
    PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1 BUFFER_POOL DEFAULT) 
    
    TABLESPACE "DATA1" 
    
    SQL> 
    
    SQL> spool off; 
---



## 获得整个SCHEMA DDL语句的方法：##



    
    set pagesize 0
    
    set long 90000
    
    set feedback off
    
    set echo off 
    
    spool get_schema.sql 
    
    connect ***/***@***;
    
    SELECT DBMS_METADATA.GET_DDL('TABLE',u.table_name)
    
    FROM USER_TABLES u;
    
    SELECT DBMS_METADATA.GET_DDL('INDEX',u.index_name)
    
    FROM USER_INDEXES u;
    
    spool off; 

---


**需要注意的是，当我们的表中有外健(参照约束)时，我们需要判别参照表之间的顺序，确保重建时按照合理的顺序进行。你可以通过查询dba_constraints and dba_cons_columns来确定各表之间的顺序，不再详述。**