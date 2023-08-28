---
title: 【翻译】MySQL数据库InnoDB存储引擎
toc: true
date: 2023-08-24 20:28:42
tags:
	- MySQL
	- InnoDB
categories:
	- 数据库
---

InnoDB is a general-purpose storage engine that balances high reliability and high performance. In MySQL 8.0, InnoDB is the default MySQL storage engine. Unless you have configured a different default storage engine, issuing a CREATE TABLE statement without an ENGINE clause creates an InnoDB table.

InnoDB是一种平衡高可靠性和高性能的通用存储引擎。在MySQL 8.0中，InnoDB是MySQL默认的存储引擎。除非您配置了不同的默认存储引擎，否则发出不带 ENGINE 子句的 CREATE TABLE 语句将创建一个 InnoDB 表。

<!-- more -->

# Key Advantages of InnoDB InnoDB的主要优势

Its DML operations follow the ACID model, with transactions featuring commit, rollback, and crash-recovery capabilities to protect user data.

其DML操作遵循ACID模型，事务具有提交、回滚和崩溃恢复功能，以保护用户数据

Row-level locking and Oracle-style consistent reads increase multi-user concurrency and performance.

行级锁定和 Oracle 风格的一致性读取提高了多用户并发性和性能

InnoDB tables arrange your data on disk to optimize queries based on primary keys. Each InnoDB table has a primary key index called the clustered index that organizes the data to minimize I/O for primary key lookups.

InnoDB 表在磁盘上排列数据以根据主键优化查询。每个 InnoDB 表都有一个称为聚集索引的主键索引，用于组织数据以最大程度地减少主键查找的 I/O。

To maintain data integrity, InnoDB supports FOREIGN KEY constraints. With foreign keys, inserts, updates, and deletes are checked to ensure they do not result in inconsistencies across related tables

为了保持数据完整性，InnoDB 支持 FOREIGN KEY 约束。使用外键，检查插入、更新和删除，以确保它们不会导致相关表之间的不一致。


# ACID

An acronym standing for atomicity, consistency, isolation, and durability. These properties are all desirable in a database system, and are all closely tied to the notion of a transaction. The transactional features of InnoDB adhere to the ACID principles.

ACID代表原子性、一致性、隔离性和持久性的缩写。这些属性都是数据库系统所需要的，并且都与事务的概念密切相关。 InnoDB 的事务特性遵循 ACID 原则。

Transactions are atomic units of work that can be committed or rolled back. When a transaction makes multiple changes to the database, either all the changes succeed when the transaction is committed, or all the changes are undone when the transaction is rolled back.

事务是可以提交或回滚的原子工作单元。当一个事务对数据库进行多次更改时，要么在事务提交时所有更改都成功，要么在事务回滚时所有更改都被撤消。

The database remains in a consistent state at all times — after each commit or rollback, and while transactions are in progress. If related data is being updated across multiple tables, queries see either all old values or all new values, not a mix of old and new values.

数据库始终保持一致的状态——每次提交或回滚之后以及事务正在进行时。如果跨多个表更新相关数据，则查询会看到所有旧值或所有新值，而不是新旧值的混合。

Transactions are protected (isolated) from each other while they are in progress; they cannot interfere with each other or see each other's uncommitted data. This isolation is achieved through the locking mechanism. Experienced users can adjust the isolation level, trading off less protection in favor of increased performance and concurrency, when they can be sure that the transactions really do not interfere with each other.

交易在进行过程中相互保护（隔离）；他们不能互相干扰或看到彼此未提交的数据。这种隔离是通过锁定机制实现的。当有经验的用户可以确定事务确实不会相互干扰时，他们可以调整隔离级别，以减少保护来提高性能和并发性。

The results of transactions are durable: once a commit operation succeeds, the changes made by that transaction are safe from power failures, system crashes, race conditions, or other potential dangers that many non-database applications are vulnerable to. Durability typically involves writing to disk storage, with a certain amount of redundancy to protect against power failures or software crashes during write operations. (In InnoDB, the doublewrite buffer assists with durability.)

事务的结果是持久的：一旦提交操作成功，该事务所做的更改就不会受到电源故障、系统崩溃、竞争条件或许多非数据库应用程序容易遭受的其他潜在危险的影响。 持久性通常涉及写入磁盘存储，并具有一定量的冗余，以防止写入操作期间发生电源故障或软件崩溃。 （在 InnoDB 中，双写缓冲区有助于提高耐用性。）