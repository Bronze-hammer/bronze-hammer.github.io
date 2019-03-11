---
title: Oracle触发器的实现
date: 2019-01-03 18:20:15
tags:
- Oracle 触发器
categories:
- 数据库
---

触发器的定义就是说某个条件成立的时候，触发器里面所定义的语句就会被自动的执行。因此触发器不需要人为的去调用，也不能调用。触发器分为语句级触发器和行级触发器。语句级触发器可以在某些语句执行前或者执行后触发，而行级触发器是定义指定表中行数据改变时的触发器。

<!-- more -->

工作中又遇到这样的问题：
> 一张表pm_cm_payapply中的vreserve14字段，总是不知道在哪步操作中被致为空（原来的值不为空）

于是打算在表中设置一个触发器

触发器实现：
> 当表被更新时，判断如果vreserve14被更新为空值，则提示报错


触发器内容：

```oracle
CREATE OR REPLACE TRIGGER tri_vreserve
AFTER UPDATE OF vreserve14 ON pm_pa_payapply
DECLARE
  myexp exception
BEGIN
  IF old.vreserve14 != '' AND new.vreserve14 == '' THEN
    RAISE myexp;
  END IF;
  EXCEPTION_INIT
    WHEN myexp THEN raise_application_error('-20002','vreserve14字段的值被设置为空')；
END;
```
测试：
![在这里插入图片描述](20190103133038329.png)

oracle查询触发器

首先我们要知道这个触发器是指向哪个表，比如我们本次是在表 pm_cm_payapply 中写的触发器，那我在 all_trigger 表中通过表明table_name查询触发器名trigger_name

```oracle
select trigger_name from all_triggers where table_name='PM_PA_PAYAPPLY';
```

>注意，表名要大写

得到触发器名为 tri_vreserve ，再通过触发器名在表all_source 表中查询触发器的具体内容

```oracle
select text from all_source where type='TRIGGER' AND name='TRI_VRESERVE';
```
>注意，触发器表名也要大写
