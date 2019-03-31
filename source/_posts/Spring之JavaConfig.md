---
title: Spring之JavaConfig
date: 2019-03-31 19:28:32
tags:
- Java
- Spring
categories:
- Java框架
---

### Spring JavaConfig实例

从Spring 3起，JavaConfig功能已经包含在Spring核心模块，它允许开发者将bean定义和在Spring配置XML文件到Java类中。但是，仍然允许使用经典的XML方式来定义bean和配置，JavaConfig是另一种替代解决方案。

我们写一个简单的实例看看，实现输出 hello world

<!-- more -->

接口
```java
package com.pepper.itf;

public interface HelloWorld {

	public void sayHello(String msg);

}
```

接口实现类

```java
package com.pepper.impl;

import com.pepper.itf.HelloWorld;

public class HelloWorldImpl implements HelloWorld {

	@Override
	public void sayHello(String msg) {
		System.out.println(msg);
	}

}
```

JavaConfig类

```Java
package com.pepper.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.pepper.impl.HelloWorldImpl;
import com.pepper.itf.HelloWorld;

@Configuration
public class AppConfig {

	@Bean(name="helloBean")
	public HelloWorld helloWorld() {
		return new HelloWorldImpl();
	}
}

```

使用 AnnotationConfigApplicationContext 加载您的JavaConfig类

```java
package com.pepper.main;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import com.pepper.config.AppConfig;
import com.pepper.itf.HelloWorld;

public class Test {

	public static void main(String[] args) {
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
		HelloWorld hello = (HelloWorld) context.getBean("helloBean");
		String msg = "This is Spring JavaConfig!";
		hello.sayHello(msg);
	}
}

```

运行程序，控制台会输出 ： This is Spring JavaConfig!

> 注意，项目需要导入 spring-aop-5.1.3.RELEASE.jar 包

由此我们可以看出，JavaConfig类

```java
package com.pepper.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.pepper.impl.HelloWorldImpl;
import com.pepper.itf.HelloWorld;

@Configuration
public class AppConfig {

	@Bean(name="helloBean")
	public HelloWorld helloWorld() {
		return new HelloWorldImpl();
	}
}
```

就类似于

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

	<bean id="helloBean" class="com.pepper.impl.HelloWorldImpl">

</beans>
```
