---
title: 处理SpringCloudConfig客户端启动无法读取到配置参数
date: 2023-02-01 17:42:28
tags:
	- Java
	- Spring Cloud Config
categories: 
	- 编程语言
---


自己部署了一个Spring Cloud微服务项目，实践Spring Cloud Config分布式配置组件，按照Spring Cloud Config 资料[Config：Spring Cloud分布式配置组件](http://c.biancheng.net/springcloud/config.html) 先后创建了Eureka注册中心服务、 Spring Cloud Config Server服务、 Spring Cloud Config Client客户端，在最后启动 Spring Client Config Client 客户端时，客户端始终无法访问 Config Server服务，读取上传在Gitee上的配置文件的内容。

在Baidu、 Google搜索了大量资料，问题是最终解决了，但是这其中的原因，还需要继续探讨。

<!-- more -->

Eureka注册中心和Spring Cloud Config Server的配置内容就不多讲，可参考[Eureka：Spring Cloud服务注册与发现组件](http://c.biancheng.net/springcloud/eureka.html) 和[Config：Spring Cloud分布式配置组件](http://c.biancheng.net/springcloud/config.html)，启动了 Config Server 服务，并用浏览器访问，上传在Gitee上的参数文件的内容是可以正常获取到的

{% asset_img  微信截图_20230201175150.png %}

我们重点说一下Spring Cloud Config Client的配置，yml文件配置如下：

```
server:
  port: 3355 #端口号
spring:
  application:
    name: spring-cloud-config-client #服务名
  cloud:
    config:
      label: master #分支名称
      name: application  #配置文件名称，application-dev.yml 中的 config
      profile: dev  #环境名  application-dev.yml 中的 dev
      #这里不要忘记添加 http:// 否则无法读取
      uri: http://localhost:3344 #Spring Cloud Config 服务端（配置中心）地址

eureka:
  client: #将客户端注册到 eureka 服务列表内
    service-url:
      defaultZone: http://localhost:9900/eureka
```

新增Controller类，用于测试配置文件内容的读取

```
@RestController
@RequestMapping("/config/client")
public class ConfigClientController {

    @Value("${server.port}")
    private String serverPort;

    @Value("${config.info}")
    private String configInfo;

    @Value("${config.version}")
    private String configVersion;

    @GetMapping(value = "/getConfig")
    public String getConfig() {
        return "info：" + configInfo + "<br/>version：" 
        	+ configVersion + "<br/>port：" + serverPort;
    }

}

```

Spring Cloud Config Client客户端在启动的时候控制台报错：

```
Caused by: java.lang.IllegalArgumentException: Could not resolve placeholder 'config.info' in value "${config.info}"
	at org.springframework.util.PropertyPlaceholderHelper.parseStringValue(PropertyPlaceholderHelper.java:180) ~[spring-core-5.3.23.jar:5.3.23]
	at org.springframework.util.PropertyPlaceholderHelper.replacePlaceholders(PropertyPlaceholderHelper.java:126) ~[spring-core-5.3.23.jar:5.3.23]
	at org.springframework.core.env.AbstractPropertyResolver.doResolvePlaceholders(AbstractPropertyResolver.java:239) ~[spring-core-5.3.23.jar:5.3.23]
	at org.springframework.core.env.AbstractPropertyResolver.resolveRequiredPlaceholders(AbstractPropertyResolver.java:210) ~[spring-core-5.3.23.jar:5.3.23]
	at org.springframework.context.support.PropertySourcesPlaceholderConfigurer.lambda$processProperties$0(PropertySourcesPlaceholderConfigurer.java:191) ~[spring-context-5.3.23.jar:5.3.23]
	at org.springframework.beans.factory.support.AbstractBeanFactory.resolveEmbeddedValue(AbstractBeanFactory.java:936) ~[spring-beans-5.3.23.jar:5.3.23]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1332) ~[spring-beans-5.3.23.jar:5.3.23]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1311) ~[spring-beans-5.3.23.jar:5.3.23]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.resolveFieldValue(AutowiredAnnotationBeanPostProcessor.java:656) ~[spring-beans-5.3.23.jar:5.3.23]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:639) ~[spring-beans-5.3.23.jar:5.3.23]
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:119) ~[spring-beans-5.3.23.jar:5.3.23]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:399) ~[spring-beans-5.3.23.jar:5.3.23]
	... 16 common frames omitted

Disconnected from the target VM, address: '127.0.0.1:54601', transport: 'socket'

Process finished with exit code 1

```

遂检查配置文件，对照资料教程看是不是自己写错了。在检查 Config Client 模块的配置文件时发现，资料上创建的配置文件名称是 `bootstrap.yml` 而非 `application.yml`

遂把配置文件名改为 bootstrap.yml， 重新启动，发现没有报之前的错误了。但是服务也没有正常运行起来，而是直接停止了，控制台输出：

```
2023-02-01 20:42:52.002  INFO 15096 --- [           main] com.netflix.discovery.DiscoveryClient    : Starting heartbeat executor: renew interval is: 30
2023-02-01 20:42:52.005  INFO 15096 --- [           main] c.n.discovery.InstanceInfoReplicator     : InstanceInfoReplicator onDemand update allowed rate per min is 4
2023-02-01 20:42:52.012  INFO 15096 --- [           main] com.netflix.discovery.DiscoveryClient    : Discovery Client initialized at timestamp 1675255372011 with initial instances count: 2
2023-02-01 20:42:52.015  INFO 15096 --- [           main] o.s.c.n.e.s.EurekaServiceRegistry        : Registering application SPRING-CLOUD-CONFIG-CLIENT with eureka with status UP
2023-02-01 20:42:52.016  INFO 15096 --- [           main] com.netflix.discovery.DiscoveryClient    : Saw local status change event StatusChangeEvent [timestamp=1675255372016, current=UP, previous=STARTING]
2023-02-01 20:42:52.018  INFO 15096 --- [nfoReplicator-0] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_SPRING-CLOUD-CONFIG-CLIENT/DESKTOP-A5PHDVG:spring-cloud-config-client:3355: registering service...
2023-02-01 20:42:52.033  INFO 15096 --- [           main] c.s.c.ConfigClientApplication            : Started ConfigClientApplication in 10.224 seconds (JVM running for 10.92)
2023-02-01 20:42:52.040  INFO 15096 --- [ionShutdownHook] o.s.c.n.e.s.EurekaServiceRegistry        : Unregistering application SPRING-CLOUD-CONFIG-CLIENT with eureka with status DOWN
2023-02-01 20:42:52.040  INFO 15096 --- [ionShutdownHook] com.netflix.discovery.DiscoveryClient    : Saw local status change event StatusChangeEvent [timestamp=1675255372040, current=DOWN, previous=UP]
2023-02-01 20:42:52.042  INFO 15096 --- [ionShutdownHook] com.netflix.discovery.DiscoveryClient    : Shutting down DiscoveryClient ...
2023-02-01 20:42:52.076  INFO 15096 --- [nfoReplicator-0] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_SPRING-CLOUD-CONFIG-CLIENT/DESKTOP-A5PHDVG:spring-cloud-config-client:3355 - registration status: 204
2023-02-01 20:42:52.077  INFO 15096 --- [nfoReplicator-0] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_SPRING-CLOUD-CONFIG-CLIENT/DESKTOP-A5PHDVG:spring-cloud-config-client:3355: registering service...
2023-02-01 20:42:52.080  INFO 15096 --- [nfoReplicator-0] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_SPRING-CLOUD-CONFIG-CLIENT/DESKTOP-A5PHDVG:spring-cloud-config-client:3355 - registration status: 204
2023-02-01 20:42:52.081  INFO 15096 --- [ionShutdownHook] com.netflix.discovery.DiscoveryClient    : Unregistering ...
2023-02-01 20:42:52.085  INFO 15096 --- [ionShutdownHook] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_SPRING-CLOUD-CONFIG-CLIENT/DESKTOP-A5PHDVG:spring-cloud-config-client:3355 - deregister  status: 200
2023-02-01 20:42:52.091  INFO 15096 --- [ionShutdownHook] com.netflix.discovery.DiscoveryClient    : Completed shut down of DiscoveryClient
Disconnected from the target VM, address: '127.0.0.1:58233', transport: 'socket'

Process finished with exit code 0

```


其中有一句 `Registering application SPRING-CLOUD-CONFIG-CLIENT with eureka with status UP`，

查询资料，在这篇文章[SpringCloud中Client向Eureka注册中心注册服务成功后不久就Unregistering（Unregistering application 服务名 with eureka with）](https://blog.csdn.net/m0_57545353/article/details/125411539)中有提出解决办法

> 虽然 Config Client 子模块依赖的父模块中，pom文件已经引入了spring-boot-web 依赖，但是依旧要在 Config Client 子模块的pom文件上加上 spring-boot-web 依赖
> ```
> <dependency>
>   <groupId>org.springframework.boot</groupId>
>   <artifactId>spring-boot-starter-web</artifactId>
> </dependency>
> ```


再次启动，服务启动成功

```
23-02-01 20:51:37.092  INFO 16848 --- [           main] c.n.discovery.InstanceInfoReplicator     : InstanceInfoReplicator onDemand update allowed rate per min is 4
2023-02-01 20:51:37.098  INFO 16848 --- [           main] com.netflix.discovery.DiscoveryClient    : Discovery Client initialized at timestamp 1675255897097 with initial instances count: 1
2023-02-01 20:51:37.100  INFO 16848 --- [           main] o.s.c.n.e.s.EurekaServiceRegistry        : Registering application SPRING-CLOUD-CONFIG-CLIENT with eureka with status UP
2023-02-01 20:51:37.100  INFO 16848 --- [           main] com.netflix.discovery.DiscoveryClient    : Saw local status change event StatusChangeEvent [timestamp=1675255897100, current=UP, previous=STARTING]
2023-02-01 20:51:37.103  INFO 16848 --- [nfoReplicator-0] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_SPRING-CLOUD-CONFIG-CLIENT/DESKTOP-A5PHDVG:spring-cloud-config-client:3355: registering service...
2023-02-01 20:51:37.138  INFO 16848 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 3355 (http) with context path ''
2023-02-01 20:51:37.139  INFO 16848 --- [           main] .s.c.n.e.s.EurekaAutoServiceRegistration : Updating port to 3355
2023-02-01 20:51:37.164  INFO 16848 --- [nfoReplicator-0] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_SPRING-CLOUD-CONFIG-CLIENT/DESKTOP-A5PHDVG:spring-cloud-config-client:3355 - registration status: 204
2023-02-01 20:51:38.241  INFO 16848 --- [           main] c.s.c.ConfigClientApplication            : Started ConfigClientApplication in 12.417 seconds (JVM running for 13.157)

```

Eureka注册中心

{% asset_img 微信截图_20230201205231.png %}

浏览器调用接口

{% asset_img 微信截图_20230201205330.png %}


那么，为什么Spring Cloud Config Client 的配置文件为什么要用 bootstrap.yml， 而不是 application ？ 这里有一篇文章有说明[SpringCloud Config - client连接server的设置写在application.yml, 导致属性无法解析](https://www.cnblogs.com/frankcui/p/15256664.html)


Bootstrap.yml (bootstrap.properties) 是在application.yml (application.properties)之前加载的。它通常用于“使用SpringCloud Config Server时，应在bootstrap.yml中指定spring.application.name和spring.cloud.config.server.git.uri”以及一些加密/解密信息。

 

Spring Cloud会创建一个`Bootstrap Context`（由bootstrap.yml加载），作为Spring应用的`Application Context`（由application.yml加载）的父上下文。初始化的时候，`Bootstrap Context`负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的`Environment`。`Bootstrap`属性有高优先级，默认情况下，它们不会被本地配置覆盖。

 

例如，当使用SpringCloud Config时，通常从服务器加载“真正的”配置数据。为了获取URL（和其他连接配置，如密码等），您需要一个较早的或“bootstrap”配置。因此，您将配置服务器属性放在bootstrap.yml中，该属性用于加载实际配置数据（通常覆盖application.yml [如果存在]中的内容）。

---


补充：

在刚开始启动Spring Cloud Config Client 时，控制台提示：

```
Description:

No spring.config.import property has been defined

Action:

Add a spring.config.import=configserver: property to your configuration.
	If configuration is not required add spring.config.import=optional:configserver: instead.
	To disable this check, set spring.cloud.config.enabled=false or 
	spring.cloud.config.import-check.enabled=false.

Disconnected from the target VM, address: '127.0.0.1:58966', transport: 'socket'

Process finished with exit code 1
```


stackoverflow上有篇文章[No spring.config.import property has been defined](https://stackoverflow.com/questions/67507452/no-spring-config-import-property-has-been-defined)中给出解决办法，在pom文件中加上依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```