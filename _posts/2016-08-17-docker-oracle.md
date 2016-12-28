---
layout: post
title:  "docker 安装oracle 11g"
date:   2016-08-17
categories: docker

---
# Docker 安装Oracle database 11.2.0.4实录
## docker 安装

* docker image 安装

`docker pull oraclelinux:6.7`

* docker 容器启动--需指定--shm-size 1G,否则静默安装会报错

`sudo docker run -t -d -h oracledb --shm-size 1g --name oracledb11g  oraclelinux:6.7`

* 进入容器

`docker-enter oracledb11g`

## Oracle 环境准备

* 安装必要的包
`yum install -y wget unzip vim oracle-rdbms-server-11gR2-preinstall`

* 修改必要的参数

1. 修改hosts
```
vim /etc/hosts
127.0.0.1       oracledb localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2      oracledb
```

2. 修改network
```
vim /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=oracledb
NOZEROCONF=yes
```

3. 修改oracle用户变量
```
vim /home/oracle/.bash_profile
添加如下参数
export NLS_LANG="Simplified Chinese_china".ZHS16GBK
export ORACLE_BASE=/oracle
export ORACLE_HOME=/oracle/11.2.0.4/db_1
export ORACLE_SID=oracledb
PATH=$PATH:$HOME/bin:$ORACLE_HOME/bin
export PATH
export LANG=en
```


* 创建工作目录并授权
    
```
mkdir -p /setup --安装介质目录 
mkdir -p /oracle --oracle安装目录 
chown -R oracle:oinstall /setup
chown -R oracle:oinstall /oracle
chmod -R 775 /setup
chmod -R 775 /oracle
```

* 在宿主机搭建http服务器传入oracle database 安装介质

`python -m SimpleHTTPServer 9090`

* 在docker容器里面通过wget下载安装介质并解压

  ```
su - oracle
cd /setup
wget -c http://172.17.0.1:9090/p13390677_112040_Linux-x86-64_1of7.zip
wget -c http://172.17.0.1:9090/p13390677_112040_Linux-x86-64_2of7.zip
unzip p13390677_112040_Linux-x86-64_1of7.zip
unzip p13390677_112040_Linux-x86-64_2of7.zip
``` 

* 修改静默安装参数文件

 此文件在/setup/database/response下，有db_install.rsp dbca.rsp netca.rsp三个文件
 内容如下
 db_install.rsp

   ```
oracle.install.responseFileVersion=/oracle/install/rspfmt_dbinstall_response_schema_v11_2_0
oracle.install.option=INSTALL_DB_SWONLY
ORACLE_HOSTNAME=oracledb
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/oracle/oraInventory
SELECTED_LANGUAGES=en,zh_CN
ORACLE_HOME=/oracle/11.2.0.4/db_1
ORACLE_BASE=/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.EEOptionsSelection=false
oracle.install.db.optionalComponents=oracle.rdbms.partitioning:11.2.0.4.0,oracle.oraolap:11.2.0.4.0,
oracle.rdbms.dm:11.2.0.4.0,oracle.rdbms.dv:11.2.0.4.0,oracle.rdbms.lbac:11.2.0.4.0,oracle.rdbms.rat:11.2.0.4.0
oracle.install.db.DBA_GROUP=oinstall
oracle.install.db.OPER_GROUP=dba
oracle.install.db.CLUSTER_NODES=
oracle.install.db.isRACOneInstall=false
oracle.install.db.racOneServiceName=
oracle.install.db.config.starterdb.type=GENERAL_PURPOSE
oracle.install.db.config.starterdb.globalDBName=oracledb
oracle.install.db.config.starterdb.SID=oracledb
oracle.install.db.config.starterdb.characterSet=ZHS16GBK
oracle.install.db.config.starterdb.memoryOption=true
oracle.install.db.config.starterdb.memoryLimit=
oracle.install.db.config.starterdb.installExampleSchemas=false
oracle.install.db.config.starterdb.enableSecuritySettings=true
oracle.install.db.config.starterdb.password.ALL=dockerdb11g
oracle.install.db.config.starterdb.password.SYS=
oracle.install.db.config.starterdb.password.SYSTEM=
oracle.install.db.config.starterdb.password.SYSMAN=
oracle.install.db.config.starterdb.password.DBSNMP=
oracle.install.db.config.starterdb.control=DB_CONTROL
oracle.install.db.config.starterdb.gridcontrol.gridControlServiceURL=
oracle.install.db.config.starterdb.automatedBackup.enable=false
oracle.install.db.config.starterdb.automatedBackup.osuid=
oracle.install.db.config.starterdb.automatedBackup.ospwd=
oracle.install.db.config.starterdb.storageType=FILE_SYSTEM_STORAGE
oracle.install.db.config.starterdb.fileSystemStorage.dataLocation=/oracle/oradata
oracle.install.db.config.starterdb.fileSystemStorage.recoveryLocation=/oracle/oraInventory
oracle.install.db.config.asm.diskGroup=
oracle.install.db.config.asm.ASMSNMPPassword=
MYORACLESUPPORT_USERNAME=
MYORACLESUPPORT_PASSWORD=
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false
DECLINE_SECURITY_UPDATES=true -- 这个参数很重要，否则安装报错，日志还看不出来什么信息
PROXY_HOST=
PROXY_PORT=
PROXY_USER=
PROXY_PWD=
PROXY_REALM=
COLLECTOR_SUPPORTHUB_URL=
oracle.installer.autoupdates.option=SKIP_UPDATES
oracle.installer.autoupdates.downloadUpdatesLoc=
AUTOUPDATES_MYORACLESUPPORT_USERNAME=
AUTOUPDATES_MYORACLESUPPORT_PASSWORD=
```

  dbca.rsp
  ```
[GENERAL]
RESPONSEFILE_VERSION = "11.2.0"
OPERATION_TYPE = "createDatabase"
[CREATEDATABASE]
GDBNAME = "oracledb"
SID = "oracledb"
TEMPLATENAME = "General_Purpose.dbc"
SYSPASSWORD = "dockerdb11g"
SYSTEMPASSWORD = "dockerdb11g"
SYSMANPASSWORD = "dockerdb11g"
DBSNMPPASSWORD = "dockerdb11g"
CHARACTERSET = "ZHS16GBK"
NATIONALCHARACTERSET= "UTF8"
LISTENERS = "listener"
[createTemplateFromDB]
SOURCEDB = "myhost:1521:orcl"
SYSDBAUSERNAME = "system"
TEMPLATENAME = "My Copy TEMPLATE"
[createCloneTemplate]
SOURCEDB = "orcl"
TEMPLATENAME = "My Clone TEMPLATE"
[DELETEDATABASE]
SOURCEDB = "orcl"
[generateScripts]
TEMPLATENAME = "New Database"
GDBNAME = "orcl11.us.oracle.com"
[CONFIGUREDATABASE]
[ADDINSTANCE]
DB_UNIQUE_NAME = "orcl11g.us.oracle.com"
NODELIST=
SYSDBAUSERNAME = "sys"
[DELETEINSTANCE]
DB_UNIQUE_NAME = "orcl11g.us.oracle.com"
INSTANCENAME = "orcl11g"
SYSDBAUSERNAME = "sys"
```

  netca.rsp

  ```
[GENERAL]
RESPONSEFILE_VERSION="11.2"
CREATE_TYPE="CUSTOM"
SHOW_GUI=false
[oracle.net.ca]
INSTALLED_COMPONENTS={"server","net8","javavm"}
INSTALL_TYPE=""typical""
LISTENER_NUMBER=1
LISTENER_NAMES={"LISTENER"}
LISTENER_PROTOCOLS={"TCP;1521"}
LISTENER_START=""LISTENER""
NAMING_METHODS={"TNSNAMES","ONAMES","HOSTNAME"}
NSN_NUMBER=1
NSN_NAMES={"EXTPROC_CONNECTION_DATA"}
NSN_SERVICE={"PLSExtProc"}
NSN_PROTOCOLS={"TCP;HOSTNAME;1521"}
``` 

* 静默安装(oracle用户，在/setup/database/目录下执行)

1. 安装oracle软件
  ```./runInstaller -silent -noconfig -ignorePrereq -responseFile /setup/database/response/db_install.rsp
  ```
  
安装完成后，切换到root用户执行
   
  ```
  /oracle/oraInventory/orainstRoot.sh
/oracle/11.2.0.4/db_1/root.sh
```
2. 创建数据库
  `dbca -silent -responseFile /setup/database/response/dbca.rsp ` 
这里要注意一下，11.2.0.4的一个BUG,要在数据库创建到70左右的时候用执行如下操作
  ```
  sqlplus / as sysdba
  alter system set JAVA_JIT_ENABLED=FALSE
  ```
否则会在76%的时候hang住，报ORA-29516错误
3. 创建监听
`netca -silent responseFile /setup/database/response/netca.rsp`

* 清理环境
  删除 /setup目录
  删除/tmp目录
  删除/oracle/diag/rdbms/oracledb/oracledb/trace/文件
  yum clean all
  
* 生成镜像
`sudo docker commit -m "oracle database 11.2.0.4 on oracle linux 6.7" 52b7bba778a1 oracledb11g:latest`

* 启动容器
`sudo docker run -dt -h oracledb --name oracle -p 1521:1521 oracledb11g`
