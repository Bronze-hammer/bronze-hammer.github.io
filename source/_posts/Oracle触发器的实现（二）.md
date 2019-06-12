---
title: Oracle触发器的实现（二）
date: 2019-05-30 17:04:38
tags:
- Oracle 触发器
categories:
- 数据库
---

在上一篇文章[Oralce触发器的实现](/2019/01/03/Oracle触发器的实现/)简单的实现了一个触发器，这篇文章会稍加复杂一点。

需求描述：

有一张`students` 的表，当表中如果插入（新增）一条数据的时候，在新增的那条数据的 `remarks` 字段中添加`This is a new data` 的备注；如果修改了一条数据，那么就把这条数据插入到 `students_updated` 这张表，并添加数据的修改时间。

<!-- more -->

创建一个测试表 `students` 表

```
CREATE TABLE student(
  ID VARCHAR2(50) DEFAULT sys_guid() PRIMARY KEY,
  name VARCHAR(20),
  age NUMBER(4,1),
  school VARCHAR2(100),
  grade VARCHAR2(50),
  address VARCHAR2(500)，
  remarks VARCHAR2(500)
);
```
插入一条测试数据（这步无所谓，可以先不要测试数据也可以）

```
INSERT INTO students (NAME, age, school, grade, address) VALUES ('xiaoming', '20', 'Changchun University of Architecture', 'Junior', 'Changchun, Jilin');
```
在新建 `students_updated` 表

```
CREATE TABLE students_updated(
  ID VARCHAR2(50) DEFAULT sys_guid() PRIMARY KEY,
  student_id VARCHAR2(50),
  name VARCHAR(20),
  age NUMBER(4,1),
  school VARCHAR2(100),
  grade VARCHAR2(50),
  address VARCHAR2(500),
  remarks VARCHAR2(500),
  ts CHAR(19)
);
```

接下来创建触发器

```

```