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

两者的生态对比：

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


Spring Cloud 的功能很明显比 Dubbo 更加强大，涵盖面更广，而且作为 Spring 的旗舰项目，它也能够与 Spring Framework、Spring Boot、Spring Data、Spring Batch 等其他 Spring 项目完美融合，这些对于微服务而言是至关重要的。

使用 Dubbo 构建的微服务架构就像组装电脑，各环节选择自由度很高，但是最终结果很有可能因为一条内存质量不行就点不亮了，总是让人不怎么放心，但是如果使用者是一名高手，那这些都不是问题。



## 2.2 dubbo和Feign远程调用的差异

Feign是SpringCloud中的远程调用方式，基于成熟Http协议，所有接口都采用Rest风格。因此接口规范更统一，而且只要符合规范，实现接口的微服务可以采用任意语言或技术开发。但受限于http协议本身的特点，请求和响应格式臃肿，其通信效率相对会差一些。

Dubbo框架默认采用Dubbo自定义通信协议，与Http协议一样底层都是TCP通信。但是Dubbo协议自定义了Java数据序列化和反序列化方式、数据传输格式，因此Dubbo在数据传输性能上会比Http协议要好一些。

不过这种性能差异除非是达极高的并发量级，否则无需过多考虑。




## 2.3 Eureka和Zookeeper注册中心的区别

SpringCloud和Dubbo都支持多种注册中心，不过目前主流来看SpringCloud用Eureka较多，Dubbo则以Zookeeper为主。两者存在较大的差异：

- 从集群设计来看：Eureka集群各节点平等，没有主从关系，因此可能出现数据不一致情况；ZK为了满足一致性，必须包含主从关系，一主多从。集群无主时，不对外提供服务
- CAP原则来看：Eureka满足AP原则，为了保证整个服务可用性，牺牲了集群数据的一致性；而Zookeeper满足CP原则，为了保证各节点数据一致性，牺牲了整个服务的可用性。
- 服务拉取方式来看：Eureka采用的是服务主动拉取策略，消费者按照固定频率（默认30秒）去Eureka拉取服务并缓存在本地；ZK中的消费者首次启动到ZK订阅自己需要的服务信息，并缓存在本地。然后监听服务列表变化，以后服务变更ZK会推送给消费者。

**扩展：**

首先，Eureka和Zookeeper都是服务治理框架，但是设计上有一定的差别。

先看下CAP原则：C-数据一致性；A-服务可用性；P-服务对网络分区故障的容错性，这三个特性在任何分布式系统中不能同时满足，最多同时满足两个。

Eureka满足AP，Zookeeper满足CP

当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，但不能接受服务直接down掉不可用。也就是说，服务注册功能对可用性的要求要高于一致性。但是Zookeeper和Eureka在一致性与可用性间做出了不同的选择。

Zookeeper：Zookeeper的设计追求数据的一致性，不保证服务的可用性。当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。问题在于，选举leader的时间太长，30 ~ 120s, 且选举期间整个zk集群都是不可用的，这就导致在选举期间注册服务瘫痪。在云部署的环境下，因网络问题使得zk集群失去master节点是较大概率会发生的事，虽然服务能够最终恢复，但是漫长的选举时间导致的注册长期不可用是不能容忍的。

Eureka：Eureka追求的是服务的可用性，从而牺牲了数据的一致性。Eureka各个节点都是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册或时如果发现连接失败，则会自动切换至其它节点，只要有一台Eureka还在，就能保证注册服务可用(保证可用性)，只不过查到的信息可能不是最新的(不保证强一致性)。除此之外，Eureka还有一种自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况。

1. Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务
2. Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上(即保证当前节点依然可用)
3. 当网络稳定时，当前实例新的注册信息会被同步到其它节点中

因此， Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zookeeper那样使整个注册服务瘫痪。

Eureka集群各节点平等，Zookeeper中有主从之分

1. 如果Zookeeper集群中部分宕机，可能会导致整个集群因为选主而阻塞，服务不可用
2. Eureka集群宕机部分，不会对其它机器产生影响

Eureka的服务发现需要主动去拉取，Zookeeper服务发现是监听机制

1. Eureka中获取服务列表后会缓存起来，每隔30秒重新拉取服务列表
2. Zookeeper则是监听节点信息变化，当服务节点信息变化时，客户端立即就得到通知


## 2.4 SpringCloud中的常用组件有哪些？

Spring Cloud的子项目很多，比较常见的都是Netflix开源的组件：

- Spring Cloud Config

集中配置管理工具，分布式系统中统一的外部配置管理，默认使用Git来存储配置，可以支持客户端配置的刷新及加密、解密操作。

- Spring Cloud Netflix

Netflix OSS 开源组件集成，包括Eureka、Hystrix、Ribbon、Feign、Zuul等核心组件。

Eureka：服务治理组件，包括服务端的注册中心和客户端的服务发现机制；
Ribbon：负载均衡的服务调用组件，具有多种负载均衡调用策略；
Hystrix：服务容错组件，实现了断路器模式，为依赖服务的出错和延迟提供了容错能力；
Feign：基于Ribbon和Hystrix的声明式服务调用组件；
Zuul：API网关组件，对请求提供路由及过滤功能。
Spring Cloud Bus
用于传播集群状态变化的消息总线，使用轻量级消息代理链接分布式系统中的节点，可以用来动态刷新集群中的服务配置。

- Spring Cloud Consul

基于Hashicorp Consul的服务治理组件。

- Spring Cloud Security

安全工具包，对Zuul代理中的负载均衡OAuth2客户端及登录认证进行支持。

- Spring Cloud Sleuth

Spring Cloud应用程序的分布式请求链路跟踪，支持使用Zipkin、HTrace和基于日志（例如ELK）的跟踪。

- Spring Cloud Stream

轻量级事件驱动微服务框架，可以使用简单的声明式模型来发送及接收消息，主要实现为Apache Kafka及RabbitMQ。

- Spring Cloud Task

用于快速构建短暂、有限数据处理任务的微服务框架，用于向应用中添加功能性和非功能性的特性。

- Spring Cloud Zookeeper

基于Apache Zookeeper的服务治理组件。

- Spring Cloud Gateway

API网关组件，对请求提供路由及过滤功能。

- Spring Cloud OpenFeign

基于Ribbon和Hystrix的声明式服务调用组件，可以动态创建基于Spring MVC注解的接口实现用于服务调用，在Spring Cloud 2.0中已经取代Feign成为了一等公民。

----
未完待续