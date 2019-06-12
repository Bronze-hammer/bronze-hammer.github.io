---
title: Oracle存储过程的实现
date: 2018-03-03 15:22:46
tags:
- Oracle 存储过程
categories:
- 数据库
---

Oracle存储过程是一组为了完成特定功能的SQL语句集，经编译后存储在数据库中。用户通过指定存储过程的名字并给出参数（是否给参数要看该存储过程定义的过程中是否设置了参数）来执行它。

<!-- more -->

#### 准备工作

创建一张测试表 `students`

```sql
create table STUDENTS(
  id      VARCHAR2(50) default sys_guid() not null,
  name    VARCHAR2(20),
  age     NUMBER(4,1),
  school  VARCHAR2(100),
  grade   VARCHAR2(50),
  address VARCHAR2(500),
  remarks VARCHAR2(500),
  ts      CHAR(19) default to_char(sysdate,'yyyy-mm-dd hh24:mi:ss')
)
```

插入一条测试数据。当然，也可以插入自己想插入的内容。

```sql
insert into STUDENTS (id, name, age, school, grade, address, remarks, ts)
values ('8A17DE17428E45D6E0530100007FABEB', 'xiaoming', 20, 'Changchun University of Architecture', 'Junior', 'Changchun, Jilin', null, '2019-06-12 11:07:57');
```

#### 第一个简单的存储过程

```sql
CREATE OR REPLACE PROCEDURE stu_school
AS school_name VARCHAR2(100);
BEGIN
  SELECT school INTO school_name FROM students WHERE ID='8A17DE17428E45D6E0530100007FABEB';
  dbms_output.put_line(school_name);
END;
```
执行存储过程，可以在PLSQL对象中看到我们刚才新创建的存储过程，并且没有报错，代表编译成功。

![简单的存储过程](TIM20190612114117.png)

执行存储过程

```sql
CALL stu_school();
```
> 调用时，"()"是必不可少的，无论是有参数还是无参数

在SQL窗口输出页签中可以看到正确的输出内容

![输出](TIM20190612114535.png)

**存储过程有一下四种情况**
- 无参数存储过程
- 仅有输入参数存储过程
- 仅有输出参数存储过程
- 既有输入又有输出存储过程

下面将对这四种存储过程分别举例说明

#### 无参数存储过程

无参数存储过程就如上面写的那个简单的存储过程，也可以这样写：

```sql
CREATE OR REPLACE PROCEDURE stu_school
AS school_name students.school%TYPE;
BEGIN
  SELECT school INTO school_name FROM students WHERE ID='8A17DE17428E45D6E0530100007FABEB';
  dbms_output.put_line(school_name);
END;
```

#### 仅有输入参数存储过程

```sql
CREATE OR REPLACE PROCEDURE stu_address(stu_id IN students.id%TYPE)
AS addr students.address%TYPE;
BEGIN
  SELECT address INTO addr FROM students WHERE ID=stu_id;
  dbms_output.put_line(addr);
END;
```

执行存储过程

```sql
CALL stu_address('8A17DE17428E45D6E0530100007FABEB');
```
![仅有输入参数存储过程](TIM20190612121128.png)

#### 仅有输出参数存储过程

```sql
CREATE OR REPLACE PROCEDURE stu_age(stu_age OUT students.age%TYPE)
AS
BEGIN
  SELECT age INTO stu_age FROM students WHERE ID='8A17DE17428E45D6E0530100007FABEB';
END;
```

需要注意的是，此种存储过程不能直接通过call来调用，不然会出现下面的错误

![](TIM20190612142651.png)

可以通过一下方式执行

> 注意，如果通过这种方式执行存储过程，要记得在存储过程中添加输出语句，不然的话，纵然执行成功，也没有结果输出。
> `dbms_output.put_line(stu_age);`

```sql
DECLARE
stuage students.age%TYPE;
BEGIN
  stu_age(stuage);
END;
```

或者通过oracle函数调用带有输出参数的存储过程

```sql
CREATE OR REPLACE FUNCTION get_stuage(stuage OUT NUMBER) RETURN NUMBER IS
BEGIN
  stu_age(stuage);
  RETURN stuage;
END;
```
执行函数

```sql
DECLARE
  stuage students.age%TYPE;
BEGIN
  dbms_output.put_line('return result:' || get_stuage(stuage));
END;

```

#### 既有输入又有输出参数的存储过程

```sql
CREATE OR REPLACE PROCEDURE stu_name(stuid IN students.id%TYPE, stuname OUT students.name%TYPE)
AS
BEGIN 
  SELECT NAME INTO stuname FROM students WHERE ID=stuid;
END;
```

新建存储函数调用存储过程

```sql
CREATE OR REPLACE FUNCTION get_stuname(stuid IN students.id%TYPE, stuname OUT students.name%TYPE) RETURN VARCHAR2 IS
BEGIN
  stu_name(stuid, stuname);
  RETURN stuname;
END;
```

执行函数

```sql
DECLARE
  stuname students.name%TYPE;
BEGIN
  dbms_output.put_line('The student name is:' || get_stuname('8A17DE17428E45D6E0530100007FABEB', stuname));
END;

```
这里有一个关于Oracle存储过程的PPT文档，供大家下载学习[点击下载](oracle存储过程.ppt)
