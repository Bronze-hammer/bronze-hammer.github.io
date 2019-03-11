---
title: Oracle去重而保存时间最新的数据
date: 2019-01-21 10:03:52
tags:
- Oracle
categories:
- 数据库
comments: true
---

#### Oracle去重而保存时间最新的数据

> 本篇文章仅仅是我工作上的一个随笔记录，写的不是很详细，如果不了解我所做的工作的业务内容，可能会看的一头雾水。大家可以仅做为一种参考思路！

场景描述：
    单据表 fdcpm_pay_apply 中的财务处理状态的值一直是从财务中间表 mid_fina_x 中获取的
现在我们在 fdcpm_pay_apply 中添加一个 ifinastatus 的字段用来直接存储财务处理状态的值，并且以后就按照这种方式存储，所以需要把财务中间表中已有的财务处理状态给刷到对应单据的 ifinastatus字段中。

##### 查看表中重复的数据有哪些(去掉重复，只显示单据编码vbillcode)

```oracle
SELECT vbillcode FROM mid_fina WHERE NVL(dr,0)=0 GROUP BY vbillcode HAVING COUNT(*)>1;
```

##### 通过上步查询出来的vbillcode，可以查询出所有重复的数据

```oracle
SELECT * FROM mid_fina WHERE NVL(dr,0)=0 AND vbillcode IN(
    SELECT vbillcode FROM mid_fina WHERE NVL(dr,0)=0 GROUP BY vbillcode HAVING COUNT(*)>1
);
```
<!-- more -->

##### 那么我们为了以防万一，复制一张一模一样的表来进行下一步的处理

```oracle
CREATE TABLE mid_fina_x AS (SELECT * FROM mid_fina WHERE NVL(dr,0)=0);
```

##### 查询数据中vbillcode相等，但是修改时间比较早的数据

```oracle
SELECT * FROM mid_fina_x mf WHERE mf.ts < (
   SELECT MAX(mx.ts) FROM mid_fina_x mx WHERE mf.vbillcode=mx.vbillcode AND NVL(mx.dr,0)=0
) AND NVL(mf.dr,0)=0;
```

##### 删除这些重复数据中的旧数据，只保留最新的那一条
```oracle
DELETE FROM mid_fina_x mf WHERE mf.ts < (
   SELECT MAX(mx.ts) FROM mid_fina_x mx WHERE mf.vbillcode=mx.vbillcode AND NVL(mx.dr,0)=0
) AND NVL(mf.dr,0)=0;
```

> 需要注意的是，按上述方法删除重复数据，只会删除时间比最新时间早的数据，如果有两条数据，他们的vbillcode和ts修改时间都是一样的，那么这两条数据都不会被删除，因此我们需要再次过来一次重复数据

前一次过滤是通过ts修改时间，那么这次我们可以选择其他有序的字段进行过滤，毕竟，如果几条数据的vbillcode和ts修改时间都相同的情况下，我们只需要取其中的某一条（不管那一条）数据的paystatus财务处理状态的值

```oracle
DELETE FROM mid_fina_x mf WHERE mf.pk_pa_payrefinfo < (
   SELECT MAX(mx.pk_pa_payrefinfo) FROM mid_fina_x mx WHERE mf.vbillcode=mx.vbillcode AND NVL(mx.dr,0)=0
) AND NVL(mf.dr,0)=0;
```

那么我们现在得到的 mid_fina_x 表就是没有重复数据的数据表了。

![总共的数据行数](TIM20190121172517.png)

![去除重复数据后的数据总数](TIM20190121173044.png)
