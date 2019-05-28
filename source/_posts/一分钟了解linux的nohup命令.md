---
title: 一分钟了解linux的nohup命令
date: 2019-05-28 15:51:22
tags:
- nohup
categories:
- 运维
---

#### 使用&后台运行程序：

结果会输出到终端

使用Ctrl + C发送SIGINT信号，程序继续执行

关闭session发送SIGHUP信号，程序关闭

#### 使用nohup运行程序：

结果默认会输出到nohup.out

使用Ctrl + C发送SIGINT信号，程序关闭

关闭session发送SIGHUP信号，程序继续执行

#### 平日线上经常使用nohup和&配合来启动程序：
同时免疫SIGINT和SIGHUP信号

同时，还有一个最佳实践：
不要将信息输出到终端标准输出，标准错误输出，而要用日志组件将信息记录到日志里

> 转自[架构师之路《一分钟了解nohup和&的功效》](https://mp.weixin.qq.com/s/nyT-FPdIUdJUiUCYVGEnTg)
