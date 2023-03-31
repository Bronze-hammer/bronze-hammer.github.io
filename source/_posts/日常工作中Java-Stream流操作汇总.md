---
title: 日常工作中Java.Stream流操作汇总
date: 2023-02-08 10:29:31
tags:
	- Java
	- Stream
categories:
	- 编程语言
---


Java 8 API添加了一个新的抽象称为流Stream，以一种声明的方式处理数据。Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。Stream API可以极大提高Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。    

————《菜鸟教程》

本片文章记录了工作中常用的Stream流操作，方便之后回顾。

<!-- more -->

## 1.对List<User>集合，取User中的一个元素，形成新的List<String>集合

```java
// List<User> userList;
List<String> userNameList = userList.stream.map(User::getUserName).collect(Collectors.toList);
```

## 2.去重

对集合 List<User> userList 内的对象去重（实体类User 使用Lombok 插件的@Data 注解，自动覆写 equals 和 hashCode 方法）
```java
// List<User> userList;
List<User> newUserList = userList.stream().distinct().collect(Collectors.toList());
```

## 3.过滤

过滤集合 List\<SysOptionData\>  optionDataList 对象label为 "zhangsan" 的数据
```java
List<SysOptionData> resultList = optionDataList.stream().filter(
	optionData -> "zhangsan".equals(optionData.getOptionLabel())
).collect(Collectors.toList());

// 返回过滤后的集合第一条数据
List<SysOptionData> resultList = optionDataList.stream().filter(
	optionData -> "zhangsan".equals(optionData.getOptionLabel())
).findFirst(); 
```

## 4.List集合转String字符串

```java
// List<User> 包含User实体的集合，只提取username拼成一个字符串，以“，”隔开
String username = userList.stream().map(User::getUsername()).collect(Collectors.joining(","));  

// List<String> usernameList
String username = usernameList.stream().collect(Collectors.joining(","));   
```

## 5.匹配两个List集合，返回新的List集合

比如有两个集合，List\<User\>、 List\<Address\>

List\<User\> 集合的数据

|  name   | age  | address |
|  ----  | ----  | ---- |
| zhangsan  | 15 | null |
| tom  | 16 | null |
| jom  | 20 | null |


List\<Address\> 集合的数据

|  name   | address  | 
|  ----  | ----  | 
| zhangsan  | China | 
| tom  | USA |
| jary  | Japan |

对比着两个集合，以List\<User\>为主体，按照name字段匹配List\<Address\>集合，把匹配到的Address对象的address字段的值设置到User对象的address字段

```java
List<User> list = userList.stream()
	.map(u -> {
		return addrList.stream()
				.filter(a -> {
					return a.getName().equals(u.getName());
				}).map(a -> {
					u.setAddress(a.getAddress());
					return u;
				}).collect(Collectors.toList());
	})
	.flatMap(List::stream)
	.collect(Collectors.toList());
for (User user : list) {
	System.out.println(user.toString());
}
```

结果输出：

```
User [name=zhangsan, age=15, address=China]
User [name=tom, age=16, address=USA]
```

> 需要注意的是，结果只会返回匹配到的数据

## 6.Stream map和flatMap的区别

### 6.1 stream.map

*Returns a stream consisting of the results of applying the givenfunction to the elements of this stream.*

返回一个流，由将给定函数应用于该流的元素的结果组成。

示例：

```java
List<String> nameList = Stream.of("ZhangSan", "Tom").collect(Collectors.toList());
nameList.stream().map(n -> n + ", welcome").forEach(e -> System.out.println(e));
```

输出：

```
ZhangSan, welcome
Tom, welcome
```

请注意另一种情况：

```java
List<String> nameList = Stream.of("ZhangSan", "Tom").collect(Collectors.toList());
List<String[]> list = nameList.stream().map(n -> n.split("")).collect(Collectors.toList());
for (String[] strings : list) {
	for (int i = 0; i < strings.length; i++) {
		System.out.print(strings[i] + " ");
	}
	System.out.println();
}
```

输出：

```
Z  h  a  n  g  S  a  n  
T  o  m  
```

> map操作就是把一种操作运算，映射到一个序列的每一个元素上。以每个元素为一个单位，运算的结果也是相互独立的，所以返回的是List\<String[]\>，而不是List\<String\>

### 6.2 stream.flatMap

*Returns a stream consisting of the results of replacing each element ofthis stream with the contents of a mapped stream produced by applyingthe provided mapping function to each element.  Each mapped stream is closed after its contentshave been placed into this stream.  (If a mapped stream is nullan empty stream is used, instead.)*

返回一个流，由将提供的映射函数应用到每个元素所产生的映射流的内容替换此流中的每个元素的结果组成。每个映射的流在其内容被放入该流后将被关闭。(如果映射流为null，则使用空流。)

*The flatMap() operation has the effect of applying a one-to-manytransformation to the elements of the stream, and then flattening theresulting elements into a new stream.*

flatMap()操作的效果是对流的元素应用一对多的转换，然后将产生的元素平铺成一个新的流。

示例：

```java
List<String> nameList = Stream.of("ZhangSan", "Tom").collect(Collectors.toList());
List<String> list = nameList.stream()
		.map(n -> n.split(""))
		.flatMap(e -> Arrays.stream(e))
		.collect(Collectors.toList());
System.out.println(list.toString());
```

输出：

```
[Z, h, a, n, g, S, a, n, T, o, m]
```

## 7.List集合转Map

List\<User\> 集合，设置 User.name 作为Map的key，User对象作为value，转换为Map集合

```java
Map<String, User> userMap = userList.stream()
	.collect(Collectors.toMap(User::getName, Function.identity()));

```


## 8.针对List列表，按照指定元素分组，生成新的Map集合

例如对下面List列表的数据做分组

```
new User("zhangsan", 12, "Guangzhou");
new User("lisi", 13, "Shenzhen");
new User("tom", 12, "Beijing");
```

分组操作

```java
Map<Integer, List<User>> map = userList.stream().collect(Collectors.groupingBy(User::getAge));
```

分组结果

```
{
	12=[
		User [name=zhangsan, age=12, address=Guangzhou], 
		User [name=tom, age=12, address=Beijing]
	], 
	13=[
		User [name=lisi, age=13, address=Shenzhen]
	]
}
```

## 9.根据指定元素去重

第1部分已经介绍了`distinct`去重方法，但是这种方法只能针对整个对象去重，如果想只根据对象中的某几个元素去重，可以通过下面的方法

```
User(name=zhangsan, sex=man, ages=1)
User(name=zhangsan, sex=man, ages=2)
User(name=lisi, sex=woman, ages=3)
User(name=wanger, sex=man, ages=4)
```

如果只根据name和sex去重

```java
userList.stream().collect(
    Collectors.collectingAndThen(
        Collectors.toCollection(() -> new TreeSet<>(
        	Comparator.comparing(p -> p.getName()+";"+p.getSex()))
        ) , ArrayList::new)
).forEach(System.out::println);
```

结果：

```
User(name=lisi, sex=woman, ages=3)
User(name=wanger, sex=man, ages=4)
User(name=zhangsan, sex=man, ages=1)
```