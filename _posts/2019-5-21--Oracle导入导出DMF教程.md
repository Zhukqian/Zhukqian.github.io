---
layout:     post
title:      Oracle导入导出DMF数据教程
subtitle:   Oracle
date:       2019-5-10
author:     朱坤乾
header-img: ![]()
catalog: true
tags:
    - 数据库
---
![]()


## Oracle简介

###  数据库

Oracle数据库是数据的物理存储.包括(数据文件ORA,DBF,控制文件,联机日志,参数文件),其中Oracle数据库与其他数据库不一样,这里的数据库一个操作系统只能有一个数据库
.可以看做Oracle就只有一个大数据库

###  实例
一个数据库有一系类的Oracle进程和内存结构组成,一个数据库可以有多个实例

###  数据文件(DMF)
数据文件是数据库的物理存放单位,数据库中的数据是存放在表空间中的.一个表空间可以有多个数据文件组成,一个数据文件只能属于一个表空间.
如果数据文件一旦加入表空间,就不能删除这个数据文件,如果要删除这个数据文件,只能删除这个表空间

###  表空间
是Oracle对物理数据库上相关数据文件(ORA或者DMF文件)的逻辑映射.一个数据库逻辑上被划分成一到多个表空间.每个表空间包含在逻辑上相关联的一组数据.
每个数据库之上有一个表空间(默认的System表空间)

###  用户
用户是在实例下创建的.不同实例下可以创建相同名字的用户.


###  重点:

		ORACLE一般过程是先建sid，再建表空间，再建用户，给用户分配刚建的表空间！最后建表

		Oracle数据库一个用户只能有一个表空间和一个临时表空间          一个表空间可以对应多个用户


##  DMF导入到数据库步骤

首先在需要导入的数据库创建一样的表空间名和用户(要求一模一样的)

###  1:首先远程数据库安装的服务器

###  2:然后查看其他的表空间的dbf表空间存放地址

###  3:连接数据库 创建表空间

创建表空间

```
create tablespace "表空间name" datafile 
  'G:\database\acarslsspace\表空间name.dbf' size 2097152000
  autoextend on next 209715200 maxsize 32767m
  logging online permanent blocksize 8192
  extent management local autoallocate default nocompress  segment space management auto
```

创建临时表空间

   create temporary tablespace "临时表空间" datafile 
  'G:\database\acarslsspace\临时表空间名.dbf' size 2097152000
  autoextend on next 209715200 maxsize 32767m
  logging online permanent blocksize 8192
  extent management local autoallocate default nocompress  segment space management auto

>  具体配置大小和意义自行百度

###  4:创建用户,然后将用户分表空间和临时表空间

```
create user 用户名
identified by 密码
default tablespace 表空间名 temporary tablespace 临时表空间名;
```

###  5:给这两个用户权限,登录,增删改查,设置

```
grant dba,connect,resource,
aq_administrator_role,aq_user_role,
authenticateduser to 用户名;
```

###  6:导入表空间  
参考:https://www.cnblogs.com/hanmk/p/7238581.html

不再写,我用的时候就参考这个网站内容,基本正确


```
删除表空间
drop tablespace 表空间名 including contents and datafiles; 
删除用户
DROP USER 用户名 CASCADE
```

##  导出数据的两种方式



方法1::exp
exp 账号/密码@192.168.xx.xx/sid file=D:\export\xx.dmp log=D:\export\gd_base.log full=y

log导入导出日志

file导出的dmp目录


数据导出(根据需求分为三种)：

1.完全导出

exp weixin/weixin@localhost:1521/orcl buffer=64000 file=F:/weixin.dmp full='y'

2.按用户导出

exp weixin/weixin@localhost:1521/orcl buffer=64000 file=F:/weixin.dmp owner='weixin'

3.按表导出

exp weixin/weixin@localhost:1521/orcl buffer=64000 file=F:/weixin.dmp tables='AJBJ'

###  说明

1)EXP和IMP是客户端工具程序,它们既可以在可以客户端使用,也可以在服务端使用。

2)EXPDP和IMPDP是服务端的工具程序,他们只能在ORACLE服务端使用,不能在客户端使用。速度比较快

3)IMP只适用于EXP导出文件,不适用于EXPDP导出文件;IMPDP只适用于EXPDP导出文件,而不适用于EXP导出文件。 

