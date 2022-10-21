---
title: 处理MySQL使用concat()函数时遇到null值的问题
date: 2022-10-21 13:43:23
tags:
	- MySQL
categories:
    - 数据库
---



#### 问题描述

使用`CONCAT()`拼接结果是，当`CONCAT()`函数中的一个参数为`null`，那么不管其他字符串是否有值，最后返回的拼接结果总是`null`，如下所示：
```sql
SELECT
	name,
	address,
	nationality,
	CONCAT('my name is ', name, ', to live in ', address, ', and i am from ', nationality) as str
FROM `user2`
```

<!-- more -->

{% asset_img 1.png %}



MySQL 官方文档有句话

{% asset_img 2.png %}


#### 解决办法
1. 使用 `COALESCE()` 函数转换null值

```sql
SELECT
	name,
	address,
	nationality,
	CONCAT('my name is ', COALESCE(name, ''), ', to live in ', COALESCE(address, ''), ', and i am from ', COALESCE(nationality, '')) as str
FROM `user2`
```

{% asset_img 3.png %}

2. 使用`IFNULL()` 函数转换null值
```sql
SELECT
	name,
	address,
	nationality,
	CONCAT('my name is ', ifnull(name, ''), ', to live in ', ifnull(address, ''), ', and i am from ', ifnull(nationality, '')) as str
FROM `user2`
```

{% asset_img 4.png %}

3. 尝试使用`CONCAT_WS()` 函数拼接字符串
```sql
SELECT
	name,
	address,
	nationality,
	CONCAT_WS(',',name,address,nationality) as str
FROM `user2`
```
{% asset_img 5.png %}


--------------------------------------
参考资料：

1. [https://stackoverflow.com/questions/15741314/mysql-concat-returns-null-if-any-field-contain-null](https://stackoverflow.com/questions/15741314/mysql-concat-returns-null-if-any-field-contain-null) 

2. [12.8 String Functions and Operators](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_concat)

3. [12.5 Flow Control Functions](https://dev.mysql.com/doc/refman/8.0/en/flow-control-functions.html#function_ifnull)
