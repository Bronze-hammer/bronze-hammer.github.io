---
title: Oracle更改用户名和密码
date: 2019-04-19 19:20:47
tags:
- Oracle
categories:
- 数据库
---

### 1、用sysdba账号登入数据库，然后查询到要更改的用户信息：

```
> SELECT user#,name FROM user$;
```

### 2、更改用户名并提交：

```
> UPDATE USER$ SET NAME='PORTAL' WHERE user#=88;
```
```
> COMMIT;
```
### 3、强制刷新：

```
> ALTER SYSTEM CHECKPOINT;
```
```
> ALTER SYSTEM FLUSH SHARED_POOL;
```

### 4、更新用户的密码：
```
> ALTER USER PORTAL IDENTIFIED BY 123;
```
