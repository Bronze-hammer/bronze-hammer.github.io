---
title: MySQL根据出生日期计算当前年龄
date: 2022-10-21 14:03:18
tags:
	- MySQL
categories:
	- 数据库
---



假如人员的出生日期为 `1994-10-01`，首先用 MySQL 的 `now()` 函数获取当前系统日期，然后利用`DATE_FORMAT()` 函数计算出当前年龄。

<!-- more -->

> 注意：`DATE_FORMAT()` 方法后面要加`0`

```sql
select DATE_FORMAT(FROM_DAYS(DATEDIFF(now(), '1994-10-01')), '%Y')+0 as age
```



实践一下，当前系统时间为  2022-08-29 10:50:54

{% asset_image 1.png %}

- `DATEDIFF()` 函数返回两个日期之间的天数。
- `FROM_DAYS()` 函数：给定一个天数N，并返回一个日期值。
- `DATE_FORMAT()` 函数用于以不同的格式显示日期/时间数据。

> 使用FROM_DAYS()谨慎旧日期，它不打算使用与之前的公历(1582年)的到来值。


-----------------------------------------
参考资料：

[https://www.tutorialspoint.com/calculate-age-based-on-date-of-birth-in-mysql](https://www.tutorialspoint.com/calculate-age-based-on-date-of-birth-in-mysql)