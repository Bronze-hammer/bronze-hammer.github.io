---
title: Arrays.binarySearch()方法详解
toc: true
date: 2023-09-05 17:14:19
tags:
categories:
---

背景：我想校验一个指定的String字符串，是否存在于另一个String数组中，选择`Arrays.binarySearch()`方法实现，代码如下：

```java
String[] item = {"0","1","16","1591","1594","1596"};
if (Arrays.binarySearch(item, "1591") > 0) {
    System.out.println("exists");
} else {
    System.out.println("not exists");
}
```

运行结果：

```
not exists
```

很直观的能看到item数组里面存在字符串`1591`，为什么程序运行的结果却是找不到该元素呢？

<!-- more -->

首先来看一下源码：

```java
/**
 * Searches the specified array for the specified object using the binary
 * search algorithm. The array must be sorted into ascending order
 * according to the
 * {@linkplain Comparable natural ordering}
 * of its elements (as by the
 * {@link #sort(Object[])} method) prior to making this call.
 * If it is not sorted, the results are undefined.
 * (If the array contains elements that are not mutually comparable (for
 * example, strings and integers), it <i>cannot</i> be sorted according
 * to the natural ordering of its elements, hence results are undefined.)
 * If the array contains multiple
 * elements equal to the specified object, there is no guarantee which
 * one will be found.
 *
 * @param a the array to be searched
 * @param key the value to be searched for
 * @return index of the search key, if it is contained in the array;
 *         otherwise, <tt>(-(<i>insertion point</i>) - 1)</tt>.  The
 *         <i>insertion point</i> is defined as the point at which the
 *         key would be inserted into the array: the index of the first
 *         element greater than the key, or <tt>a.length</tt> if all
 *         elements in the array are less than the specified key.  Note
 *         that this guarantees that the return value will be &gt;= 0 if
 *         and only if the key is found.
 * @throws ClassCastException if the search key is not comparable to the
 *         elements of the array.
 */
public static int binarySearch(Object[] a, Object key) {
    return binarySearch0(a, 0, a.length, key);
}
```

注意，注释上提到了两个重点：

1. 使用二分查找算法在指定数组中搜索指定对象；
2. 在调用此方法之前，必须根据元素的自然顺序(如通过sort(Object[])方法)将数组按升序排序。

也就是说，数组不是遍历每一个元素，与目标值做对比，校验是否相同，而是通过二分查找算法，先找到数组中间的元素，与目标值做比较：如果目标值大于中间值，则继续比较数组后半部分的元素；如果目标值小于中间值，则继续比较数组前半部分的元素；如果等于，那么就直接返回中间元素的数组下标。因此在调用此方法之前，要先对数组进行升序排序。

```java
private static int binarySearch0(Object[] a, int fromIndex, int toIndex,
                                     Object key) {
    int low = fromIndex;
    int high = toIndex - 1;

    while (low <= high) {
        int mid = (low + high) >>> 1;
        @SuppressWarnings("rawtypes")
        Comparable midVal = (Comparable)a[mid];
        @SuppressWarnings("unchecked")
        int cmp = midVal.compareTo(key);

        if (cmp < 0)
            low = mid + 1;
        else if (cmp > 0)
            high = mid - 1;
        else
            return mid; // key found
    }
    return -(low + 1);  // key not found.
}
```

源码中，是中间值通过`(low + high) >>> 1` 的方式获取的。

这是一个在二分查找算法中常见的代码片段。low 和 high 通常表示搜索范围的下界和上界。mid 是下界和上界的中间值，通过 (low + high) >>> 1 计算得出。

这是由于在Java中，+ 运算符对于整数是按照整数算术运算（即舍去小数点）来执行的。这可能会导致整数的溢出。例如，如果 low 是 -1000000000，而 high 是 1000000000，那么 low + high 的结果将会是 -999999999 + 1000000000，这将导致整数溢出。

而使用 >>>（无符号右移运算符）则可以避免这个问题。位运算中，右移运算符 >> 对于负数会将移位后的左侧填充部分填充为该数的符号位（即负数的话填充为1，正数的话填充为0）。而 >>> 是无符号右移运算符，无论该数是正数还是负数，都会将左侧填充部分填充为0。

所以 (low + high) >>> 1 的结果就是 low + high 的值除以2的整数部分，无论 low 和 high 的值是多少。

拿到数组的中间元素后，通过`int cmp = midVal.compareTo(key)`的方法比较中间元素和目标值。

compareTo() 方法按字典顺序比较两个字符串（比较基于字符串中每个字符的 Unicode 值）。


接下来回到最开始的问题中，通过调试发现，中间元素`16`与目标值`1591`比较，结果cmp=1，也就是说比较的结果居然是`16`大于`1591`。

{% asset_image 微信截图_20230905182124.png %}

由于比较的结果大于0，因此 `mid-1`，接下来会拿数组左边部分的值与目标值做对比，而`16`左边的几个元素不存在`1591`，因此最终结果是在数组中找不到与目标值一致的元素。

这是因为，`16`和`1591`都是字符串类型，而非数值类型，字符串类型通过`compareTo()` 方法进行比较，是比较两个字符串相应位置字符的Unicode值。

```java
/**
 * Compares two strings lexicographically.
 * The comparison is based on the Unicode value of each character in
 * the strings. The character sequence represented by this
 * {@code String} object is compared lexicographically to the
 * character sequence represented by the argument string. The result is
 * a negative integer if this {@code String} object
 * lexicographically precedes the argument string. The result is a
 * positive integer if this {@code String} object lexicographically
 * follows the argument string. The result is zero if the strings
 * are equal; {@code compareTo} returns {@code 0} exactly when
 * the {@link #equals(Object)} method would return {@code true}.
 * <p>
 * This is the definition of lexicographic ordering. If two strings are
 * different, then either they have different characters at some index
 * that is a valid index for both strings, or their lengths are different,
 * or both. If they have different characters at one or more index
 * positions, let <i>k</i> be the smallest such index; then the string
 * whose character at position <i>k</i> has the smaller value, as
 * determined by using the &lt; operator, lexicographically precedes the
 * other string. In this case, {@code compareTo} returns the
 * difference of the two character values at position {@code k} in
 * the two string -- that is, the value:
 * <blockquote><pre>
 * this.charAt(k)-anotherString.charAt(k)
 * </pre></blockquote>
 * If there is no index position at which they differ, then the shorter
 * string lexicographically precedes the longer string. In this case,
 * {@code compareTo} returns the difference of the lengths of the
 * strings -- that is, the value:
 * <blockquote><pre>
 * this.length()-anotherString.length()
 * </pre></blockquote>
 *
 * @param   anotherString   the {@code String} to be compared.
 * @return  the value {@code 0} if the argument string is equal to
 *          this string; a value less than {@code 0} if this string
 *          is lexicographically less than the string argument; and a
 *          value greater than {@code 0} if this string is
 *          lexicographically greater than the string argument.
 */
public int compareTo(String anotherString) {
    int len1 = value.length;
    int len2 = anotherString.value.length;
    int lim = Math.min(len1, len2);
    char v1[] = value;
    char v2[] = anotherString.value;

    int k = 0;
    while (k < lim) {
        char c1 = v1[k];
        char c2 = v2[k];
        if (c1 != c2) {
            return c1 - c2;
        }
        k++;
    }
    return len1 - len2;
}
```

> 引申：但是要注意的是，这只适用于两个字符都属于基本字母或数字的情况。如果字符包含其他字符（比如特殊字符、标点符号等），或者涉及非数字字符，那么结果可能不会如你所预期。例如，如果'A'和'中'（Unicode值为65296）进行相减，结果将会是-38321，这显然不是我们期望的结果。因此，在进行这种操作时，一定要确保字符的取值范围和你的预期相符。

至此，文章开头 `1591`为什么在目标数组中 `{"0","1","16","1591","1594","1596"}` 匹配不到的问题，原因就是如此。