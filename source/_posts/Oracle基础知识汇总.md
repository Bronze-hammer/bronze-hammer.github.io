---
title: Oracle基础知识汇总
date: 2019-03-15 19:24:43
tags:
- Oracle
categories:
- 数据库
---

Oracle Database，又名Oracle RDBMS，或简称Oracle。是甲骨文公司的一款关系数据库管理系统。它是在数据库领域一直处于领先地位的产品。可以说Oracle数据库系统是目前世界上流行的关系数据库管理系统，系统可移植性好、使用方便、功能强，适用于各类大、中、小、微机环境。它是一种高效率、可靠性好的 适应高吞吐量的数据库解决方案。

对于开发来说，了解和学习Oracle数据库是非常有必要的。

> 这篇文章仅仅是对Oracle做了一个简单的总结，后续如果用到或接触到的知识点，都会做补充


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

>Oracle中 dba_users 和 user_users 都可以查询用户的信息，这两张表都是查看表空间信息的表，它们的区别是：前者是只供管理员用户查询的，后者是管理员用户和普通用户都可以查询的。


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

如果要解锁它，我们首先需要登陆其他用户，并通过SQL语句`alter user scott account unlock;`解锁它。

```
alter user scott account unlock;
```
> scott用户的默认密码为 tiger

接下来通过 'connect scott/tiger' 来登陆scott用户。如果scott用户密码过期的话，系统会提示你更改密码

![更改scott用户的密码](TIM20190316173459.png)

或者通过SQL语句更改scott用户的密码

```
alter user scott identified by tiger;
```

##### ⚪表空间

###### 表空间的分类

- 永久表空间  存放永久性数据，如表，索引等。
- 临时表空间  不能存放永久性对象，用于保存数据库排序，分组时产生的临时数据。
- UNDO表空间  保存数据修改前的镜象。

###### 查看用户的表空间

在前面我们提到的 dba_users 表中有两个字段

1. DEFAULT_TABLESPACE
2. TEMPORARY_TABLESPACE

这两个字段分别代表用户的默认表空间和临时表空间。

比如我们要查看system用户的默认表空间和临时表空间

![用户system的默认表空间和临时表空间](TIM20190316174555.png)

###### 查看表空间的信息

这里有两张表

1. dba_tablespaces
2. user_tablespaces

这两张表都是查看表空间信息的表，它们的区别是：前者是只供管理员用户查询的，后者是管理员用户和普通用户都可以查询的。

![查看表空间的信息](TIM20190316175754.png)

###### 更改我们指定用户的默认表空间或者临时表空间

```
ALTER USER username DEFAULT|TEMPORARY TABLESPACE tablespace_name;
```

###### 新建表空间

```
CREATE [TEMPORARY] TABLESPACE tablespace_name TEMPFILE|DATAFILE 'xxx.dbf' SIZE xxx;
```

表空间创建完毕后，我们可以查看表空间的位置在哪，比如我们查看表空间名称为SYSTEM的位置

```
SELECT file_name FROM dba_data_files WHERE tablespace_name='SYSTEM';
```

注意，如果要查看创建的临时表空间的数据文件的地址，就不能查表 dba_data_files 了，而要查 dba_temp_file 表。

```
SELECT file_name FROM dba_temp_files WHERE tablespace_name='TEMPORARY_TABLESPACE';
```

###### 修改表空间的状态

设置表空间的联机或脱机状态

```
ALTER TABLESPACE tablespace_name ONLINK|OFFLINK;
```

设置表空间的只读或可读写的状态

```
ALTER TABLESPACE tablespace_name READ ONLY|READ WRITE;
```

> 表空间为脱机状态，那么系统不允许修改该表空间的读写状态。表空间联机的状态，默认该表空间为可读写的状态。

###### 修改表空间中的数据文件

增加数据文件

```
ALTER TABLESPACE tablespace_name ADD DATAFILE|TEMPFILE 'xxx.dbf' SIZE xxx;
```

删除数据文件

```
ALTER TABLESPACE tablespace_name DROP DATAFILE 'filename.dbf';
```

###### 删除表空间

```
DROP TABLESPACE tablespace_name [INCLUDING CONTENTS];
```

如果只删除表空间而不删除数据文件，那么不用加上[]里面的内容。


##### ⚪约束

数据的完整性用于确保数据库数据遵从一定的商业和逻辑规则。在Oracle中，数据完整性可以使用约束、触发器、应用程序（过程、函数）三种方法来实现，在这三种方法中，因为约束易于维护，并且具有最好的性能，所以作为维护数据完整性的首选。

约束用于确保数据库数据满足特定的商业规则。在Oracle中，约束包括：not null、unique、primary key， foreign key和check五种。
- not null（非空）
如果在列上定义了not null，那么当插入数据时，必须为列提供数据。
- unique（唯一）
当定义了唯一约束后，该列值是不能重复的，但是可以为null。
- primary key（主键）
用于唯一的标识表行的数据，当定义主键约束后，该列不但不能重复而且不能为NULL。一张表最多只能有一个主键，但是可以由多个unique约束。
- foreign key（外键）
用于定义主表和从表之间的关系，外键约束要定义在从表上，主要则必须具有主键约束或是unique约束，当定义外键约束后，要求外键列数据必须在主表的主键列存在或是为NULL。
- check
用于强制行数据必须满足的条件，假定在sal列上定义了check约束，并要求sal列值在1000～2000之间，如果不在1000～2000之间就会提示出错。


##### ⚪修改表名

```
ALTER TABLE 旧表名 RENAME TO 新表名;
```