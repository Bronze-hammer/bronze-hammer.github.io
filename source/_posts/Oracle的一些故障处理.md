---
title: Oracle的一些故障处理
date: 2019-04-08 13:17:55
tags:
- oracle
categories:
- 数据库
comments: true
---

这里汇总了一些在使用Oracle过程中遇到的问题及解决办法，一方面做为笔记帮助自己以后更快速处理问题，一方面分享处理供大家互相学习。

<!-- more -->

### ⚪SP2-0667: Message file sp1<lang>.msb not found

![](TIM20190408103118.png)

出错原因：
crontab里面的脚本，通常读取的是默认的环境变量，PATH里面不包含oracle数据库的路径。

解决办法：
```
vim ~/.bashrc
```

把一下内容填写其中

```
export ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
```

> 注意，ORACLE_HOME的路径要是你计算机中oracle真实的安装地址

环境变量设置完成，执行 `source ~/.bashrc` 使其生效。

![](TIM20190408122029.png)

### ⚪ORA-12162: TNS:net service name is incorrectly specified

![](TIM20190408121707.png)

一般出现这种错误，基本都是环境变量配置有问题，要么是没有配置正确的ORACLE_SID、ORACLE_HOME，要么是监听配置环境变量和.bash_profile环境变量配置不一致。

这里检查发现，是操作系统环境变量没有配置ORACLE_SID

![](TIM20190408122343.png)

因此，我们配置一下 ~/.bashrc ，在其中添加ORACLE_SID

```
export ORACLE_SID=erp
```

> 注意：ORACLE_SID的值要根据自己安装oracle时设置的为准

![](TIM20190408122811.png)

![](TIM20190408122556.png)


### ⚪ORA-04021: timeout occurred while waiting to lock object

#### 情景描述:

Oracle中本来有个用户NC63PM_PEIXUN1，我把这个用户名更改为了NC63PM_PEIXUN2（更改方法请参考[【Oracle更改用户名和密码】](/2019/04/19/Oracle更改用户名和密码)），之后我想按照旧的用户名再创建一个用户，但是创建的用户的SQL语句执行了十五分钟还没执行完，并报如下的错误：

![ORA-04021: timeout occurred while waiting to lock object](TIM20190420145348.png)

#### 解决办法

查看是否被锁表了

```
> SELECT object_name,machine,s.sid,s.serial#
> FROM v$locked_object l,dba_objects o ,v$session s
> WHERE l.object_id=o.object_id AND l.session_id=s.sid;
```

发现没有被锁表

![查看锁表](TIM20190420145700.png)

使用 DBA_DDL_LOCKS视图获得DDL锁定信息

```
> SELECT * FROM dba_ddl_locks;
```

发现有两条关于 NC63PM_PEIXUN1 用户的锁定信息

![](TIM20190420150236.png)

通过 session_id 找到对应的锁表信息

```
> SELECT sid,serial#,status FROM v$session a WHERE a.sid in (829,392);
```

![](TIM20190420150455.png)

注：因我是kill掉这两条信息后才截的图，所以 STATUS 才为 KILLED 的。

kill这两条锁表

```
> ALTER SYSTEM KILL SESSION '392, 5049';
> ALTER SYSTEM KILL SESSION '829, 25287';
```

再次执行创建用户的脚本就能顺利执行。
