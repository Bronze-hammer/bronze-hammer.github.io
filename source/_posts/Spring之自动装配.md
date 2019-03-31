---
title: Spring之自动装配
date: 2019-03-31 19:30:00
tags:
- Java
- Spring
categories:
- Java框架
---

#### Spring由名称自动装配

Spring有一下几种装配模式：

- 默认情况下，通过ref指定装配bean
- byName - 根据属性名称自动装配
- byType - 根据数据类型自动装配

<!-- more -->

> 还有两种我暂时没理解，以后再说

那么接下来分别说说这几种自动装配

#### 默认设置

新建两个bean

```java
package com.assembly.beans;

public class School {

	private String name = "Changchun college of architecture";

	public String getName() {
		return name;
	}

}
```

```java
package com.assembly.beans;

public class Student {

	private School school;

	public School getSchool() {
		return school;
	}

	public void setSchool(School school) {
		this.school = school;
	}

}
```

默认设置下，我们通过'ref'属性来连接bean

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Spring-Common.xml -->
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">

	<bean id="student" class="com.assembly.beans.Student">
		<property name="school" ref="school" />
	</bean>

	<bean id="school" class="com.assembly.beans.School"></bean>
</beans>
```

#### 通过属性名称自动装配

按属性名称自动装配，就是指如果一个bean的名称与其他bean属性的名称一样，那么将自动装配它。以下面的xml文件为例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Spring-Common.xml -->
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">

	<bean id="student" class="com.assembly.beans.Student" autowire="byName"></bean>

	<bean id="school" class="com.assembly.beans.School">
		<property name="name" value="Changchun college of architecture"></property>
	</bean>
</beans>
```

如果 com.assembly.beans.Student 类中有个属性名称为 'school'，那么第二个bean将自动装配到第一个bean中。

```java
package com.assembly.beans;

public class School {

	private String name;

	//...
}
```

```java
package com.assembly.beans;

public class Student {

	private School school;

	//...
}
```

运行程序

![通过属性名称自动装配](TIM20190212131101.png)

但是如果 Student 类中的属性方法名为 'SCHOOL'，那么将会装配失败

```java
package com.assembly.beans;

public class Student {

	private School SCHOOL;

	//...
}
```

#### 通过属性类型自动装配

按属性类型自动装配是指一个bean的数据类型与其他bean属性的类型相同，则自动装配它。

> 我们以上面 通过属性名称自动装配 的例子来说明

我们把 School 类做一下修改，把变量 name 的数据类型改成布尔类型

```java
package com.assembly.beans;

public class School {

	private Boolean name;

	public Boolean getName() {
		return name;
	}

	public void setName(Boolean name) {
		this.name = name;
	}
}
```

修改XML文件，让Spring按照类型自动装配

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Spring-Common.xml -->
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">

	<bean id="student" class="com.assembly.beans.Student" autowire="byType"></bean>

	<bean id="school" class="com.assembly.beans.School">
		<property name="name" value="Changchun college of architecture!!!"></property>
	</bean>

</beans>
```

再次运行程序，会报如下的错误

```
...
Caused by: java.lang.IllegalArgumentException: Invalid boolean value [Changchun college of architecture!!!]
	at org.springframework.beans.propertyeditors.CustomBooleanEditor.setAsText(CustomBooleanEditor.java:154)
	at org.springframework.beans.TypeConverterDelegate.doConvertTextValue(TypeConverterDelegate.java:466)
	at org.springframework.beans.TypeConverterDelegate.doConvertValue(TypeConverterDelegate.java:439)
	at org.springframework.beans.TypeConverterDelegate.convertIfNecessary(TypeConverterDelegate.java:192)
	at org.springframework.beans.AbstractNestablePropertyAccessor.convertIfNecessary(AbstractNestablePropertyAccessor.java:585)
	... 28 more
```

还有通过注解实现自动装配等的例子，后续会再做补充
