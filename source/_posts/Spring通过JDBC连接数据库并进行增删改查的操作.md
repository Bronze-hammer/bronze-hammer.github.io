---
title: Spring通过JDBC连接数据库并进行增删改查的操作
date: 2019-03-31 19:27:48
tags:
- Java
- Spring
categories:
- Java框架
---

### Spring通过JDBC连接MySQL数据库并进行增删改查的操作

首先我们在自己的MySQL数据库中建一张学生成绩表，做为我们的测试用例。

```sql
CREATE TABLE `students` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(50) DEFAULT NULL,
  `math` int(50) DEFAULT NULL,
  `english` int(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8
```

<!-- more-->

随便向里面插入一条数据

```sql
INSERT INTO students ('name', 'math', 'english') values ('zhangsan', '80', '75');
```

接下来开始新建我们的实例项目，先看看项目目录结构

![项目结构](TIM20190128190724.png)

> 注意，必要的jar包不要少了

#### 首先对应我们建的学生成绩表新建一个学生成绩模型

```java
package com.jdbc.model;

public class Student {

	private int id;
	private String name;
	private int math;
	private int english;

	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getMath() {
		return math;
	}
	public void setMath(int math) {
		this.math = math;
	}
	public int getEnglish() {
		return english;
	}
	public void setEnglish(int english) {
		this.english = english;
	}
	public Student(int id, String name, int math, int english) {
		super();
		this.id = id;
		this.name = name;
		this.math = math;
		this.english = english;
	}
}

```

#### 新建成绩查询的接口，并具体实现它

```java
package com.jdbc.itf;

import com.jdbc.model.Student;

public interface StudentDAO {

	public Student findByStudentId(int id);

}

```

```java
package com.jdbc.impl;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

import javax.sql.DataSource;

import com.jdbc.itf.StudentDAO;
import com.jdbc.model.Student;

public class JdbcStudentDAO implements StudentDAO {

	private DataSource data_source;

	public void setData_source(DataSource data_source) {
		this.data_source = data_source;
	}

	@Override
	public Student findByStudentId(int id) {
		String querySQL = "SELECT  * FROM students WHERE id = ?";
		Connection conn = null;
		try {
			conn = data_source.getConnection();
			PreparedStatement ps = conn.prepareStatement(querySQL);
			ps.setInt(1, id);
			ResultSet rs = ps.executeQuery();
			Student student = null ;
			if(rs.next()) {
				student = new Student(
						rs.getInt("id"),
						rs.getString("name"),
						rs.getInt("math"),
						rs.getInt("english"));
			}
			rs.close();
			ps.close();
			return student;
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			if(conn != null) {
				try {
					conn.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
		}
		return null;
	}

}

```

我们注意到，在实现类的开始定义了一个 DataSource 类型的变量 data_source ，用来连接我们的数据库。那么 data_source 具体定义的是什么呢？这个需要我们通过Spring的xml文件向其注入属性。

#### 新建xml文件

> mysql数据库的ip地址和账号密码请修改为自己的

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">

	<bean id="studentDAO" class="com.jdbc.impl.JdbcStudentDAO">
		<property name="data_source" ref="dataSource" />
	</bean>

	<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
		<property name="url" value="jdbc:mysql://IP:3306/my?useSSL=false" />
		<property name="username" value="root" />
		<property name="password" value="root" />
	</bean>

</beans>
```

#### 功能实现

一切准备好了，那么我们通过一个主文件来运行它

```java
package com.jdbc.main;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.jdbc.itf.StudentDAO;
import com.jdbc.model.Student;

public class Application {

	public static void main(String[] args) {
		ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
		StudentDAO studentDao = (StudentDAO) context.getBean("studentDAO");
		Student std = studentDao.findByStudentId(1);
		System.out.println("Id:"+std.getId());
		System.out.println("Name:"+std.getName());
		System.out.println("Math:"+std.getMath());
		System.out.println("English:"+std.getEnglish());
	}
}

```

结果显示：

![](TIM20190128191601.png)

以上例子仅仅是查询的，那么下面的例子是对增删该查的总体汇总。

首先在接口中把剩余的方法补充上

```java
package com.jdbc.itf;

import com.jdbc.model.Student;

public interface StudentDAO {

	//查询
	public Student findByStudentId(int id);
	//新增
	public void insertNewStudent(Student studnet);
	//删除
	public void deleteByStudentId(int id);
	//更改
	public void updateByStudentId(Student student);

}
```

在接口实现类中实现它们

```java
package com.jdbc.impl;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

import javax.sql.DataSource;

import com.jdbc.itf.StudentDAO;
import com.jdbc.model.Student;

public class JdbcStudentDAO implements StudentDAO {

	private DataSource data_source;

	public void setData_source(DataSource data_source) {
		this.data_source = data_source;
	}

	@Override
	public Student findByStudentId(int id) {
		String querySQL = "SELECT  * FROM students WHERE id = ?";
		Connection conn = null;
		try {
			conn = data_source.getConnection();
			PreparedStatement ps = conn.prepareStatement(querySQL);
			ps.setInt(1, id);
			ResultSet rs = ps.executeQuery();
			Student student = null ;
			if(rs.next()) {
				student = new Student(
						rs.getInt("id"),
						rs.getString("name"),
						rs.getInt("math"),
						rs.getInt("english"));
			}
			rs.close();
			ps.close();
			return student;
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			if(conn != null) {
				connClose(conn);
			}
		}
		return null;
	}

	@Override
	public void insertNewStudent(Student studnet) {
		String insertSQL = "INSERT INTO students (name, math, english) VALUES (?, ?, ?)";
		Connection conn = null;
		try {
			conn = data_source.getConnection();
			PreparedStatement ps = conn.prepareStatement(insertSQL);
			ps.setString(1, studnet.getName());
			ps.setInt(2, studnet.getMath());
			ps.setInt(3, studnet.getEnglish());
			ps.executeUpdate();
			ps.close();
		} catch(Exception e) {
			e.printStackTrace();
		} finally {
			connClose(conn);
		}
	}

	@Override
	public void deleteByStudentId(int id) {
		Connection conn = null;
		try {
			String deleteSQL = "DELETE FROM students where id=?";
			conn = data_source.getConnection();
			PreparedStatement ps = conn.prepareStatement(deleteSQL);
			ps.setInt(1, id);
			ps.executeUpdate();
			ps.close();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			connClose(conn);
		}

	}

	@Override
	public void updateByStudentId(Student student) {
		Connection conn = null;
		try {
			String updateSQL = "UPDATE students SET name=?, math=?, english=? WHERE id=?";
			conn = data_source.getConnection();
			PreparedStatement ps = conn.prepareStatement(updateSQL);
			ps.setString(1, student.getName());
			ps.setInt(2, student.getMath());
			ps.setInt(3, student.getEnglish());
			ps.setInt(4, student.getId());
			ps.executeUpdate();
			ps.close();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			connClose(conn);
		}

	}

	private void connClose(Connection conn) {
		if (conn != null) {
			try {
				conn.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
	}

}

```

> updateByStudentId方法实现对数据的修改不是最优的，后续会对它进行进一步的优化
