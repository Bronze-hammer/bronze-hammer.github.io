---
title: Spring之松耦合
date: 2019-03-31 19:17:39
tags:
- Java
- Spring
categories:
- Java框架
---

### Spring松耦合的个人理解和代码实例

理解Spring的松耦合概念，那么我们先来看看一个不使用Sring的实例代码

先看一下整个测试项目案例的结构

<!-- more -->

![项目结构](TIM20190124191037.png)

#### 正常方式

创建一个接口，这个接口指定车辆的行驶速度

```java
package com.car.run;

public interface SpeedCar {

	public void runSpeed();

}

```

现在我们指定两辆车的行驶速度：一辆F1赛车，行驶最高速度为300 km/h, 一辆客运大巴，速度为 80 km/h。

```java
//F1赛车
package com.car.impl;

import com.car.run.SpeedCar;

public class F1Racing implements SpeedCar {

	@Override
	public void runSpeed() {
		System.out.println("The maximum speed of the F1 is 300 km/h.");
	}

}
```

```java
//客运大巴
package com.car.impl;

import com.car.run.SpeedCar;

public class PassengerBus implements SpeedCar {

	@Override
	public void runSpeed() {
		System.out.println("The maximum speed of a passenger bus is 80 km/h.");
	}

}

```

按正常方式我们直接调用

```java
package com.car.main;

import com.car.impl.PassengerBus;
import com.car.run.SpeedCar;

public class Test {

	public static void main(String[] args) {
		SpeedCar speedcar = new PassengerBus();
		speedcar.runSpeed();
	}
}

```
目前来看，其实这种方法也挺简单的，无非就是new 出一个客运大巴的对象 speedcar，然后通过speedcar来输出客运大巴的最高时速。但是我们new出的对象speedcar完全依赖的是客运大巴，或者说对客运大巴的耦合度特别高，它也仅仅是指向客运大巴PassengerBus，如果我们要输出F1赛车的最高时速，那么还需要再次new出F1赛车的对象。

结果显示

![客运大巴](TIM20190124185051.png)

#### 通过辅助类实现

```java
package com.car.helper;

import com.car.impl.PassengerBus;
import com.car.run.SpeedCar;

public class SpeedCarHelper {

	SpeedCar speedcar;

	public SpeedCarHelper(){
		speedcar = new PassengerBus();
	}

	public void getSpeed() {
		speedcar.runSpeed();
	}
}

```

改造我们上一个方法中主方法实现内容

```java
package com.car.main;

import com.car.helper.SpeedCarHelper;

public class Test {

	public static void main(String[] args) {
		SpeedCarHelper helper = new SpeedCarHelper();
		helper.getSpeed();
	}

}
```

那么实际上这种方法也是一种高耦合，helper依赖与辅助类，而这个辅助类又依赖与客运大巴PassengerBus。

结果显示

![客运大巴](TIM20190124185051.png)

#### Spring实现

最后，我们通过Spring的依赖注入，实现生成松散的耦合。

改造辅助类

```java
package com.car.helper;

import com.car.run.SpeedCar;

public class SpeedCarHelper {

	SpeedCar speedcar;

	public void setSpeedcar(SpeedCar speedcar) {
		this.speedcar = speedcar;
	}

	public void getSpeed() {
		speedcar.runSpeed();
	}
}
```

创建 Spring bean 的配置文件，并在这里声明所有的Java对象的依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Spring-Common.xml -->
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">

	<bean id="speedCarHelper" class="com.car.helper.SpeedCarHelper">
		<property name="speedcar" ref="F1Racing" />
	</bean>

	<bean id="F1Racing" class="com.car.impl.F1Racing" />
	<bean id="PassengerBus" class="com.car.impl.PassengerBus" />

</beans>
```

改造主方法

```java
package com.car.main;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.car.helper.SpeedCarHelper;

public class Test {

	public static void main(String[] args) {
		ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
		SpeedCarHelper helper = (SpeedCarHelper) context.getBean("speedCarHelper");
		helper.getSpeed();
	}

}

```
结果显示

![F1赛车](TIM20190124190413.png)

现在我们通过beans.xml文件生成指定的输出，如果在后续的工作中我们希望输出客运大巴的最高时速，直接更改xml文件即可，不用修改代码，这样就降低了代码的耦合度。
