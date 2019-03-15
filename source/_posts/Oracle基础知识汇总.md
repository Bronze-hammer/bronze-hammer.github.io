---
title: Oracle基础知识汇总
date: 2019-08-15 19:24:43
tags:
- Oracle
categories:
- 数据库
---

Oracle Database，又名Oracle RDBMS，或简称Oracle。是甲骨文公司的一款关系数据库管理系统。它是在数据库领域一直处于领先地位的产品。可以说Oracle数据库系统是目前世界上流行的关系数据库管理系统，系统可移植性好、使用方便、功能强，适用于各类大、中、小、微机环境。它是一种高效率、可靠性好的 适应高吞吐量的数据库解决方案。

对于开发来说，了解和学习Oracle数据库是非常有必要的。

<!-- more -->

*********************

##### ⚪查看登陆的用户

show user 命令

```
SQL> show user
```

![当前登陆用户](TIM20190315193636.png)

##### ⚪dba_users 数据字典

通过 dba_users 这个表查看登陆的用户信息，那么在查询之前我们先看看 dba_users 这个表的表结构，可以通过 `desc dba_users` 这个命令查看指定表的表结构

```
SQL> desc dba_users
```
![表结构](TIM20190315193905.png)

可看到该表结构中包含用户名、用户的ID和密码等内容

现在我们查看系统中用户的信息
```
select * from dba_users;
```
![系统中的用户](TIM20190315194654.png)


##### ⚪几个典型的用户说明

Oracle中有那么几个典型的用户，它们是在安装Oracle的时候默认创建的，分别具有不同的作用。下面我们介绍Oracle中sys,system,scott,hr用户：

- scott 是个演示用户，是让你学习Oracle用的；
- hr用户是个示例用户，是在创建数据库时选中“示例数据库”后产生的，实际就是模拟一个人力资源部的数据库；
- SYSDBA 不是用户，可以认为是个权限，超级权限。默认中sys就拥有这种超级权限，是权限最高的用户。

超级用户分两种 SYSDBA和SYSOPT
SYSOPT 后面3个字母是operator的意思，也就是操作数据库的人，而SYSDBA 则是管理数据库的人;
SYSDBA比SYSOPT的权限还要大，而SYS用户就完全是个SYSDBA，但SYSTEM用户默认是SYSOPT，不过它也能以SYSDBA的权限登陆。

它们之间的不同之处可以下载[Oracle中sys-system-scott-hr用户的区别](Oracle中sys-system-scott-hr用户的区别.doc) 这份文档了解学习。


##### ⚪启动（解锁）被锁定的用户

Oracle中有个scott用户默认是被锁定的

![scott表被锁](TIM20190315201123.png)

如果要解锁它，我们首先需要登陆其他用户，并通过SQL语句`alter user scott account unlick;`解锁它。

```
alter user scott account unlick;
```
