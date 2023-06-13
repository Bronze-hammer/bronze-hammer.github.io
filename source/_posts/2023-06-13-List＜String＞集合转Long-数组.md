---
title: 'List＜String＞集合转Long[]数组'
toc: true
date: 2023-06-13 15:54:46
tags:
	- List
	- String
	- Long
categories:
	- 代码人生
---


```java
List<String> list = new ArrayList<>();
Long[] item = list.toArray(new Long[0]);
```
`List<String>`直接以toArray的方式转换`Long数组`是错误的，运行后报错：

```
Exception in thread "main" java.lang.ArrayStoreException
```

查看`java.util.List.toArray(T[])`方法，注释中明确写到：

> @throws  ArrayStoreException if the runtime type of the specified array is not a supertype of the runtime type of every element in this list
> 如果指定数组的运行时类型不是此列表中每个元素的运行时类型的超类型

明显Long类型不是String类型的超类。

可以通过下面的方式实现

```java
List<String> stringList = Arrays.asList("1", "2", "3");
Long[] item = list.stream().map(Long::valueOf).toArray(Long[]::new);
```

<!-- more -->

**引申：**

如何实现 `List<String>` 转 `List<Long>` ?

实现方式：

```java
List<String> stringList = Arrays.asList("1", "2", "3");
List<Long> longList = stringList.stream().map(Long::valueOf).collect(Collectors.toList());
```

-----

`java.util.List.toArray(T[])` 源码：

```java
/**
     * Returns an array containing all of the elements in this list in
     * proper sequence (from first to last element); the runtime type of
     * the returned array is that of the specified array.  If the list fits
     * in the specified array, it is returned therein.  Otherwise, a new
     * array is allocated with the runtime type of the specified array and
     * the size of this list.
     *
     * <p>If the list fits in the specified array with room to spare (i.e.,
     * the array has more elements than the list), the element in the array
     * immediately following the end of the list is set to <tt>null</tt>.
     * (This is useful in determining the length of the list <i>only</i> if
     * the caller knows that the list does not contain any null elements.)
     *
     * <p>Like the {@link #toArray()} method, this method acts as bridge between
     * array-based and collection-based APIs.  Further, this method allows
     * precise control over the runtime type of the output array, and may,
     * under certain circumstances, be used to save allocation costs.
     *
     * <p>Suppose <tt>x</tt> is a list known to contain only strings.
     * The following code can be used to dump the list into a newly
     * allocated array of <tt>String</tt>:
     *
     * <pre>{@code
     *     String[] y = x.toArray(new String[0]);
     * }</pre>
     *
     * Note that <tt>toArray(new Object[0])</tt> is identical in function to
     * <tt>toArray()</tt>.
     *
     * @param a the array into which the elements of this list are to
     *          be stored, if it is big enough; otherwise, a new array of the
     *          same runtime type is allocated for this purpose.
     * @return an array containing the elements of this list
     * @throws ArrayStoreException if the runtime type of the specified array
     *         is not a supertype of the runtime type of every element in
     *         this list
     * @throws NullPointerException if the specified array is null
     */
    <T> T[] toArray(T[] a);
```