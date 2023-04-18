---
title: ArrayList去重方式总结
toc: true
date: 2023-04-17 19:37:05
tags:
categories:
	- 代码人生
feature: title.png
---

我在日常工作中常用Stream方式去重，满足了工作上业务的需求即可，并没有深入了解和尝试其他方式的去重操作，这对于个人的成长是很有局限性的，遂借此机会整理ArrayList的去重方法。

<!-- more -->

## 1.使用迭代器遍历去重


```java
// Java program to remove duplicates from ArrayList

import java.util.*;

public class GFG {

	// Function to remove duplicates from an ArrayList
	public static <T> ArrayList<T> removeDuplicates(ArrayList<T> list)
	{

		// Create a new ArrayList
		ArrayList<T> newList = new ArrayList<T>();

		// Traverse through the first list
		for (T element : list) {

			// If this element is not present in newList
			// then add it
			if (!newList.contains(element)) {

				newList.add(element);
			}
		}

		// return the new list
		return newList;
	}

	// Driver code
	public static void main(String args[])
	{

		// Get the ArrayList with duplicate values
		ArrayList<Integer>
			list = new ArrayList<>(
				Arrays
					.asList(1, 10, 1, 2, 2, 3, 3, 10, 3, 4, 5, 5));

		// Print the Arraylist
		System.out.println("ArrayList with duplicates: "
						+ list);

		// Remove duplicates
		ArrayList<Integer>
			newList = removeDuplicates(list);

		// Print the ArrayList with duplicates removed
		System.out.println("ArrayList with duplicates removed: "
						+ newList);
	}
}

```

输出：

```
ArrayList with duplicates: [1, 10, 1, 2, 2, 3, 3, 10, 3, 4, 5, 5]
ArrayList with duplicates removed: [1, 10, 2, 3, 4, 5]
```

## 2.使用LinkedHashSet

可以将ArrayList转换成不允许值重复的Set集合，因此LinkedHashSet是最好的选择，因为它不允许重复.

>HashSet也能实现同样的去重效果，但是HashSet与LinkedHashSet的不同之处在于，LinkedHashSet同时保留了插入顺序。

```java
// Java program to remove duplicates from ArrayList

import java.util.*;

public class GFG {

	// Function to remove duplicates from an ArrayList
	public static <T> ArrayList<T> removeDuplicates(ArrayList<T> list)
	{

		// Create a new LinkedHashSet
		Set<T> set = new LinkedHashSet<>();

		// Add the elements to set
		set.addAll(list);

		// Clear the list
		list.clear();

		// add the elements of set
		// with no duplicates to the list
		list.addAll(set);

		// return the list
		return list;
	}

	// Driver code
	public static void main(String args[])
	{

		// Get the ArrayList with duplicate values
		ArrayList<Integer>
			list = new ArrayList<>(
				Arrays
					.asList(1, 10, 1, 2, 2, 3, 10, 3, 3, 4, 5, 5));

		// Print the Arraylist
		System.out.println("ArrayList with duplicates: "
						+ list);

		// Remove duplicates
		ArrayList<Integer>
			newList = removeDuplicates(list);

		// Print the ArrayList with duplicates removed
		System.out.println("ArrayList with duplicates removed: "
						+ newList);
	}
}

```

输出：

```
ArrayList with duplicates: [1, 10, 1, 2, 2, 3, 10, 3, 3, 4, 5, 5]
ArrayList with duplicates removed: [1, 10, 2, 3, 4, 5]
```

>更简单的方法可以这样写
```java
Set<String> set = new LinkedHashSet<>(yourList);
yourList.clear();
yourList.addAll(set);
```

## 3.使用Java 8版本中的Stream.distinct()方法

distinct()方法根据equals()方法返回的结果返回一个没有重复元素的新Stream，可用于进一步处理。

```java
// Java program to remove duplicates from ArrayList

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

// Program to remove duplicates from a List in Java 8
class GFG
{
	public static void main(String[] args)
	{
		// input list with duplicates
		List<Integer> list = new ArrayList<>(
			Arrays.asList(1, 10, 1, 2, 2, 3, 10, 3, 3, 4, 5, 5));
			// Print the Arraylist
		System.out.println("ArrayList with duplicates: "
						+ list);

		// Construct a new list from the set constucted from elements
		// of the original list
		List<Integer> newList = list.stream().distinct().collect(Collectors.toList());

		// Print the ArrayList with duplicates removed
		System.out.println("ArrayList with duplicates removed: "
						+ newList);
	}
}

```

输出：

```
ArrayList with duplicates: [1, 10, 1, 2, 2, 3, 10, 3, 3, 4, 5, 5]
ArrayList with duplicates removed: [1, 10, 2, 3, 4, 5]
```