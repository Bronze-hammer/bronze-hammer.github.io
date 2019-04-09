---
title: Oracle的一些故障处理
date: 2019-04-08 13:17:55
tags:
- oracle
categories:
- 数据库
comments: true
---

这里汇总了一些在使用Oracle过程中遇到的问题及解决办法，一方面做为笔记帮助自己以后更快速处理问题，一方面分享处理供大家互相学习。

<!-- more -->

### ⚪SP2-0667: Message file sp1<lang>.msb not found

![](TIM20190408103118.png)

出错原因：
crontab里面的脚本，通常读取的是默认的环境变量，PATH里面不包含oracle数据库的路径。

解决办法：
```
vim ~/.bashrc
```

把一下内容填写其中

```
export ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
```

> 注意，ORACLE_HOME的路径要是你计算机中oracle真实的安装地址

环境变量设置完成，执行 `source ~/.bashrc` 使其生效。

![](TIM20190408122029.png)

### ⚪ORA-12162: TNS:net service name is incorrectly specified

![](TIM20190408121707.png)

一般出现这种错误，基本都是环境变量配置有问题，要么是没有配置正确的ORACLE_SID、ORACLE_HOME，要么是监听配置环境变量和.bash_profile环境变量配置不一致。

这里检查发现，是操作系统环境变量没有配置ORACLE_SID

![](TIM20190408122343.png)

因此，我们配置一下 ~/.bashrc ，在其中添加ORACLE_SID

```
export ORACLE_SID=erp
```

> 注意：ORACLE_SID的值要根据自己安装oracle时设置的为准

![](TIM20190408122811.png)

![](TIM20190408122556.png)
