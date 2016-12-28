---
layout: post
title:  "超级系统工具Sysdig"
categories: Linux
excerpt: 超级系统工具Sysdig，比 strace、tcpdump、lsof 加起来还强大 
---
* content
{:toc}
可以用sysdig命令做很多很酷的事情


## 网络

查看占用网络带宽最多的进程

sysdig -c topprocs_net
显示主机192.168.0.1的网络传输数据

as binary:
sysdig -s2000 -X -c echo_fds fd.cip=192.168.0.1
as ASCII:
sysdig -s2000 -A -c echo_fds fd.cip=192.168.0.1
查看连接最多的服务器端口

in terms of established connections:
sysdig -c fdcount_by fd.sport "evt.type=accept"
in terms of total bytes:
sysdig -c fdbytes_by fd.sport
查看客户端连接最多的ip

in terms of established connections
sysdig -c fdcount_by fd.cip "evt.type=accept"
in terms of total bytes
sysdig -c fdbytes_by fd.cip
列出所有不是访问apache服务的访问连接

sysdig -p"%proc.name %fd.name" "evt.type=accept and proc.name!=httpd"

## 容器

查看机器上运行的容器列表及其资源使用情况

sudo csysdig -vcontainers
查看容器上下文的进程列表

sudo csysdig -pc
查看运行在wordpress1容器里CPU的使用率

sudo sysdig -pc -c topprocs_cpu container.name=wordpress1
查看运行在wordpress1容器里网络带宽的使用率

sudo sysdig -pc -c topprocs_net container.name=wordpress1
查看在wordpress1容器里使用网络带宽最多的进程

sudo sysdig -pc -c topprocs_net container.name=wordpress1
查看在wordpress1 容器里占用 I/O 字节最多的文件

sudo sysdig -pc -c topfiles_bytes container.name=wordpress1
查看在wordpress1 容器里网络连接的排名情况

sudo sysdig -pc -c topconns container.name=wordpress1
显示wordpress1容器里所有命令执行的情况

sudo sysdig -pc -c spy_users container.name=wordpress1

## 应用

查看机器所有的HTTP请求

sudo sysdig -s 2000 -A -c echo_fds fd.port=80 and evt.buffer contains GET
查看机器所有的SQL select查询

sudo sysdig -s 2000 -A -c echo_fds evt.buffer contains SELECT
See queries made via apache to an external MySQL server happening in real time

sysdig -s 2000 -A -c echo_fds fd.sip=192.168.30.5 and proc.name=apache2 and evt.buffer contains SELECT
硬盘 I/O

查看使用硬盘带宽最多的进程

sysdig -c topprocs_file
列出使用大量文件描述符的进程

sysdig -c fdcount_by proc.name "fd.type=file"
See the top files in terms of read+write bytes

sysdig -c topfiles_bytes
Print the top files that apache has been reading from or writing to

sysdig -c topfiles_bytes proc.name=httpd
Basic opensnoop: snoop file opens as they occur

sysdig -p "%12user.name %6proc.pid %12proc.name %3fd.num %fd.typechar %fd.name" evt.type=open
See the top directories in terms of R+W disk activity

sysdig -c fdbytes_by fd.directory "fd.type=file"
See the top files in terms of R+W disk activity in the /tmp directory

sysdig -c fdbytes_by fd.filename "fd.directory=/tmp/"
Observe the I/O activity on all the files named 'passwd'

sysdig -A -c echo_fds "fd.filename=passwd"
Display I/O activity by FD type

sysdig -c fdbytes_by fd.type
进程和CPU使用率

See the top processes in terms of CPU usage

sysdig -c topprocs_cpu
See the top processes for CPU 0

sysdig -c topprocs_cpu evt.cpu=0
Observe the standard output of a process

sysdig -s4096 -A -c stdout proc.name=cat
性能和错误

See the files where most time has been spent

sysdig -c topfiles_time
See the files where apache spent most time

sysdig -c topfiles_time proc.name=httpd
See the top processes in terms of I/O errors

sysdig -c topprocs_errors
See the top files in terms of I/O errors

sysdig -c topfiles_errors
See all the failed disk I/O calls

sysdig fd.type=file and evt.failed=true
See all the failed file opens by httpd

sysdig "proc.name=httpd and evt.type=open and evt.failed=true"
See the system calls where most time has been spent

sysdig -c topscalls_time
See the top system calls returning errors

sysdig -c topscalls "evt.failed=true"
snoop failed file opens as they occur

sysdig -p "%12user.name %6proc.pid %12proc.name %3fd.num %fd.typechar %fd.name" evt.type=open and evt.failed=true
Print the file I/O calls that have a latency greater than 1ms:

sysdig -c fileslower 1


## 安全

Show the directories that the user "root" visits

sysdig -p"%evt.arg.path" "evt.type=chdir and user.name=root"
Observe ssh activity

sysdig -A -c echo_fds fd.name=/dev/ptmx and proc.name=sshd
Show every file open that happens in /etc

sysdig evt.type=open and fd.name contains /etc
Show the ID of all the login shells that have launched the "tar" command

sysdig -r file.scap -c list_login_shells tar
Show all the commands executed by the login shell with the given ID

sysdig -r trace.scap.gz -c spy_users proc.loginshellid=5459