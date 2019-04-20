---
title: Oracle通过expdp导出/impdp导入
date: 2019-04-20 11:58:48
tags:
- Oracle
- expdp
- impdp
categories:
- 数据库
---

EXP和IMP是客户端工具程序，它们既可以在客户端使用，也可以在服务端使用。
EXPDP和IMPDP是服务端的工具程序，他们只能在ORACLE服务端使用，不能在客户端使用。
IMP只适用于EXP导出的文件，不适用于EXPDP导出文件；IMPDP只适用于EXPDP导出的文件，而不适用于EXP导出文件。

### 创建逻辑目录

逻辑目录用于存放导出的dmp数据文件和log日志文件

#### 查看逻辑目录

```
SELECT * FROM dba_directories;
```

<!-- more -->

![查看逻辑目录](TIM20190420121004.png)

#### 新增逻辑目录

如果不想用上面查到的已有的逻辑目录，可以自己新增

```
CREATE directory DUMP_NAME AS '/home/dump_name';
```

> 目录名称和路径自己定义

#### 为目录文件夹赋予修改和读取的权限

```
GRANT read, write on directory DUMP_NAME to NC63PM_PEIXUN1;
```
### expdp导出数据

导出NC63PM_PEIXUN1用户下的数据

```
expdp NC63PM_PEIXUN1/NC63PM_PEIXUN1@192.168.18.49/erp directory=DUMP_NAME dumpfile=peixun190420.dmp logfile=peixun190420.log
```

![expdp导出数据](TIM20190420120602.png)

![导出成功](TIM20190420123152.png)

### impdp导入数据

```
impdp soctt/tiger remap_schema=NC63PM_PEIXUN1:soctt  dumpfile=peixun190420.dmp Logfile=peixun190420.log directory=backup
```
