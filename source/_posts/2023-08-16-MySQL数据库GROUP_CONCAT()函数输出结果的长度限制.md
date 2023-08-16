---
title: MySQL数据库GROUP_CONCAT()函数输出结果的长度限制
date: 2023-08-16 15:09:39
tags:
	- MySQL
	- group_concat
	- group_concat_max_len
categories:
	- 数据库
toc: true
---

GROUP_CONCAT()函数输出的结果，发现被截取了一部分，并没有显示完整，原来GROUP_CONCAT() 默认的输出长度为1024字节，超出的部分会被截掉不显示。

<!-- more -->

## GROUP_CONCAT()

MySQL官方文档关于[GROUP_CONCAT()](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_group-concat)函数的说明：

{% asset_image 微信截图_20230816145238.png %}


文档中除了 GROUP_CONCAT()函数的语法和使用，还提到：

> 结果被截断为 `group_concat_max_len` 系统变量指定的最大长度，该变量的默认值为 1024

group_concat() 函数输出的结果长度，由 `group_concat_max_len` 系统变量所限制，超出设置的最大长度，将会被截掉， `group_concat_max_len` 的默认长度为`1024`

在运行时更改  `group_concat_max_len`  值的语法如下，其中 val 是无符号整数

```mysql
SET [GLOBAL | SESSION] group_concat_max_len = val;
```



## GROYP_CONCAT_MAX_LEN

MySQL官方文档关于服务器系统变量 [group_concat_max_len](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_group_concat_max_len) 的解释：

{% asset_image 微信截图_20230816144139.png %}

GROUP_CONCAT() 函数的长度以字节为单位，默认值为 1024。

64位系统值`18446744073709551615`,63为系统值`4294967295`,最小值都为`4` 。



