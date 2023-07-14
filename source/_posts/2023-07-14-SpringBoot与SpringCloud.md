---
title: SpringBoot与SpringCloud
toc: true
date: 2023-07-14 13:33:33
tags:
	- SpringBoot
	- SpringCloud
categories:
	- 代码人生
---


# 1. SpringBoot

## 1.1 SpringBoot的作用

SpringBoot是一个快速构建项目并简化项目配置的工具，内部集成了Tomcat及大多数第三方应用和Spring框架的默认配置。与我们学习的SpringMVC和SpringCloud并无冲突,SpringBoot提供的这些默认配置，大大简化了SpringMVC、SpringCloud等基于Spring的Web应用的开发。

## 1.2.SpringBoot的自动配置原理（如何实现）？

### SpringBoot的自动配置是如何实现的？

一般我们的SpringBoot项目启动类都会添加`@SpringBootApplication`注解，而这个注解的其中一个二级注解是`@EnableAutoConfiguration`注解。而`@EnableAutoConfiguration`注解通过`@Import`注解，以`ImportSelector`接口的方法来导入classpath下的`META-INF/spring.factories`文件，这些文件中会指定需要加载的一些类名称。

这些类一般都加了`@Configuration`注解，并且完成了对某框架（例如Redis、SpringMVC）的默认配置，当这些类符合条件时，就会被实例化，其中的配置生效，那么自动配置自然生效了。

<!-- more -->

### 满足怎样的条件配置才会生效？

一般提供默认配置的类都会添加`@ConditionalOnXxx`这样的注解，例如：`@ConditionalOnClass`，`@ConditionalOnProperties`等。`@ConditionalOnClass`表示只有classpath中存在某些指定的类时，条件满足，此时该配置类才会生效。例如Redis的默认配置其实早就有了，但是只有你引入redis的starter依赖，才满足了条件，触发自动配置。

### 那如果我需要覆盖这些默认配置呢？

有两种方式可以覆盖默认配置：

- SpringBoot提供默认配置时，会在提供的Bean上加注解@ConditionalOnMissingBean，意思是如果这个Bean不存在时条件满足，那么我们只要配置了相同的Bean，那么SpringBoot提供的默认配置就会失效
- SpringBoot提供默认配置时，一些关键属性会通过读取application.yml或者application.properties文件来获取，因此我们可以通过覆盖任意一个文件中的属性来覆盖默认配置。

## 1.3 自定义SpringBoot的stater？

项目中某些中间件的客户端（如Redis、ElasticSearch）会进行二次封装，并通过starter方式提供jar包，供大家使用。

一般定义starter包括下面几个子工程：

- xxx-spring-boot-starter：pom格式，管理当前starter中需要的各种依赖
- xxx-spring-boot-autoconfigure：jar格式，自动配置的核心代码

以elasticsearch为例来说说autoconfigure中包含哪些

- elasticsearch的工具类
- 属性加载的类，一般通过@ConfigurationProperties注解读取yaml文件中的es地址
- 添加了@Configuration的配置类，作用是初始化elasticsearch工具类，初始化elasticsearch客户端，初始化一些其它必备的实例。
- resource下定义META-INF文件夹，并且文件夹下定义spring.factories文件，文件中是key-value形式:key是EnableAutoConfiguration这个注解的全路径名,value是我们自定义自动配置类（加了@Configuration的类），如果有多个以","隔开

## 1.4 SpringBoot项目的启动流程

SpringBoot项目启动第一步就是创建SpringApplication的实例，并且调用SpringApplication.run()这个方法。

创建SpringApplication实例主要完成三件事情：

- 记录当前启动类字节码
- 判断当前项目类型，普通Servlet、响应式WebFlux、NONE
- 加载/META-INF/spring.factories文件，初始化ApplicationContextInitializer和ApplicationListener实例

而后的run()方法则会创建spring容器，流程如下：

- 准备监听器，监听Spring启动的各个过程
- 创建并配置环境参数Environment
- 创建ApplicationContext
- prepareContext()：初始化ApplicationContext，准备运行环境
- refreshContext(context)：准备Bean工厂，调用一个BeanDefinition和BeanFactory的后处理器，初始化各种Bean，初始化tomcat
- afterRefresh()：拓展功能，目前为空
- 发布容器初始化完毕的事件

## 1.5 SpringBoot的配置加载优先级

SpringBoot参数配置方式很多，比较常用参数配置方式按照优先级从高到低分别是：

- 在命令行中传入的参数
- java 的系统属性，可以通过System.getProperties()获得的内容
- 操作系统的环境变量
- 针对不同{profile}环境的配置文件内容，例如 applicaiton-{profile}.yaml
- application.yml或application.proerties文件
- 在@Configration注解修改的类中，通过@PropertySource注解定义的属性

# 2 SpringCloud

## 2.1 SpringCloud和Dubbo的区别

两者都是现在主流的微服务框架，但却存在不少差异：

- 初始定位不同：SpringCloud定位为微服务架构下的一站式解决方案；Dubbo 是 SOA 时代的产物，它的关注点主要在于服务的调用和治理
- 生态环境不同：SpringCloud依托于Spring平台，具备更加完善的生态体系；而Dubbo一开始只是做RPC远程调用，生态相对匮乏，现在逐渐丰富起来。
- 调用方式：SpringCloud是采用Http协议做远程调用，接口一般是Rest风格，比较灵活；Dubbo是采用Dubbo协议，接口一般是Java的Service接口，格式固定。但调用时采用Netty的NIO方式，性能较好。
- 组件差异比较多，例如SpringCloud注册中心一般用Eureka，而Dubbo用的是Zookeeper

SpringCloud生态丰富，功能完善，更像是品牌机，Dubbo则相对灵活，可定制性强，更像是组装机。

| 功能         | Dubbo            | SpringCloud                        |
| ------------ | ---------------- | ---------------------------------- |
| 服务注册中心 | Zookeeper        | Eureka(主流）、Consul、zookeeper   |
| 服务调用方式 | RPC基于Dubbo协议 | REST API 基于Http协议              |
| 服务监控     | Dubbo-Monitor    | Spring Boot Admin                  |
| 熔断器       | 不完善           | Spring Cloud Netflix Hystrix       |
| 服务网关     | 无               | Spring Cloud Netflix Zuul、Gateway |
| 分布式配置   | 无               | Spring Cloud Config                |
| 服务跟踪     | 无               | Spring Cloud Sleuth+Zipkin(一般)   |
| 数据流       | 无               | Spring Cloud Stream                |
| 批量任务     | 无               | Spring Cloud Task                  |
| 信息总线     | 无               | Spring Cloud Bus                   |



----
未完待续