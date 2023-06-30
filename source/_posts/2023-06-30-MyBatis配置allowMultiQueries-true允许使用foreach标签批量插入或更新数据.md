---
title: MyBatis配置allowMultiQueries=true允许使用foreach标签批量插入或更新数据
toc: true
date: 2023-06-30 12:59:35
tags:
	- MySQL
	- allowMultiQueries
categories:
	- MySQL
	- 代码人生
	- 数据库
---


执行update更新操作

```xml
<update id="batchUpdate" parameterType="java.util.List">
    <foreach collection="list" item="item" separator=";" open="" close="">
        update test_table
        <set>
            <if test="item.a != null">output_amount = #{item.a},</if>
            <if test="item.b!= null">invoice_amount = #{item.b},</if>
            <if test="item.c!= null">payment_amount = #{item.c},</if>
        </set>
        where id = #{item.id}
    </foreach>
</update>
```

执行报错：
```
Error updating database.  
Cause: java.sql.SQLSyntaxErrorException: 
You have an error in your SQL syntax; 
check the manual that corresponds to your MySQL server version for the right syntax to use near 'update test_table set a = 1, ' at line 14
```

刚开始出现这个错误，以为是update语句写的有问题，但是检查了很多遍都没有问题。奇怪的是，同样的代码，同样的数据，本地启的环境不行，测试环境却可以。经过同事提醒了一下，于是检查了一下配置文件，果然发现配置文件上的jdbc配置，测试环境比开发环境多了个`allowMultiQueries=true`。

```xml
jdbc:mysql://127.0.0.1:3306/db?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true
```

MyBatis默认情况下不允许批量插入或更新数据的原因是出于安全考虑。在许多情况下，数据的插入和更新都需要经过验证和控制，以确保数据的完整性和一致性。如果允许默认的批量操作，可能会导致不正确的数据插入或更新，从而影响应用程序的正常运行。

通过配置`allowMultiQueries=true`可以开启MyBatis的批量操作功能。这个配置项告诉MyBatis允许在单个数据库连接中执行多个SQL语句，从而实现批量插入或更新数据的功能。但是要注意，在开启批量操作之前，确保你已经了解并理解了可能引发的安全风险，并且在使用批量操作时要进行适当的验证和控制，以确保数据的完整性和安全性。
