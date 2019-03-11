---
title: 通过SQL修改报表模板
date: 2019-01-22 17:03:12
tags:
- Oracle
- NC报表
categories:
- 数据库
---

> 本文档为工作随笔，所以请大家略过

报表模板数据表为  pub_report_templet

通过报表编码查询报表的主体内容

```oracle
SELECT * FROM pub_report_templet WHERE NVL(dr,0)=0;
```

<!-- more -->

parent_code 字段为报表的编码字段

可根据 pk_templet 字段查出报表的所有列，并对所有列进行操作

```oracle
SELECT * FROM pub_report_model WHERE pk_templet='1001Z31000000000YC41' AND NVL(dr,0)=0;
```
