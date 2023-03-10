---
title: ArrayList.retainAll()方法解析
date: 2023-03-09 13:57:09
tags:
	- ArrayList
	- retainAll()
categories:
	- 代码人生
---

在工作中，用java.util.ArrayList.retainAll(Collection<?>)方法判断两个list集合是否有交集（两个list是否有相同的元素）。如果两个集合有相同元素，那么retainAll返回true。但是如果两个集合的元素完全相同，返回的结果却是false,而如果两个list集合的元素都不一样，retainAll却返回true。 这是怎么回事呢？

<!-- more -->

先来看看`ArrayList.retainAll(Collection<?>)` 源码

```java

/**
 * Retains only the elements in this list that are contained in the
 * specified collection.  In other words, removes from this list all
 * of its elements that are not contained in the specified collection.
 *
 * @param c collection containing elements to be retained in this list
 * @return {@code true} if this list changed as a result of the call
 * @throws ClassCastException if the class of an element of this list
 *         is incompatible with the specified collection
 * (<a href="Collection.html#optional-restrictions">optional</a>)
 * @throws NullPointerException if this list contains a null element and the
 *         specified collection does not permit null elements
 * (<a href="Collection.html#optional-restrictions">optional</a>),
 *         or if the specified collection is null
 * @see Collection#contains(Object)
 */
public boolean retainAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, true);
}

private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    int r = 0, w = 0;
    boolean modified = false;
    try {
        for (; r < size; r++)
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r];
    } finally {
        // Preserve behavioral compatibility with AbstractCollection,
        // even if c.contains() throws.
        if (r != size) {
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r;
        }
        if (w != size) {
            // clear to let GC do its work
            for (int i = w; i < size; i++)
                elementData[i] = null;
            modCount += size - w;
            size = w;
            modified = true;
        }
    }
    return modified;
}
```

retainAll注释的第一句已经基本交代了方法的功能

> Retains only the elements in this list that are contained in the specified collection.  In other words, removes from this list all of its elements that are not contained in the specified collection.
> 
> 仅保留此列表中包含在指定集合中的元素。换句话说，从此列表中删除未包含在指定集合中的所有元素。

```java
for (; r < size; r++)
	if (c.contains(elementData[r]) == complement)
		elementData[w++] = elementData[r];
```

遍历此列表 `elementData` 中的元素，判断是否与 `c`集合有相同元素，把相同的元素覆盖在此列表的第一个地址上，以此类推。以`w` 记录相同元素的个数。

回到刚开始的问题上，为什么两个集合的元素都一样，结果返回 false， 两个集合元素都不一样，结果返回 true 呢？

```java
if (w != size) {
    // clear to let GC do its work
    for (int i = w; i < size; i++)
        elementData[i] = null;
    modCount += size - w;
    size = w;
    modified = true;
}
```

如果两个集合的元素完全一样，`w == size`，这样会跳过判断，直接返回 modified 的初始值 false; 如果两个集合的元素都不一样，`w != size`满足条件，modified 被赋值为true，返回结果即为true。

所以单纯用ArrayList.retainAll()方法，根据返回值true/false,判断两个集合是否有相同元素，是不准确的。

可以使用另一个方法 `java.util.Collections.disjoint(Collection<?> c1, Collection<?> c2)`，两个指定collection中没有相同的元素，则返回true。如果其中一个集合为null，则抛出NullPointerException。

源码：

```java
/**
 * Returns {@code true} if the two specified collections have no
 * elements in common.
 *
 * <p>Care must be exercised if this method is used on collections that
 * do not comply with the general contract for {@code Collection}.
 * Implementations may elect to iterate over either collection and test
 * for containment in the other collection (or to perform any equivalent
 * computation).  If either collection uses a nonstandard equality test
 * (as does a {@link SortedSet} whose ordering is not <em>compatible with
 * equals</em>, or the key set of an {@link IdentityHashMap}), both
 * collections must use the same nonstandard equality test, or the
 * result of this method is undefined.
 *
 * <p>Care must also be exercised when using collections that have
 * restrictions on the elements that they may contain. Collection
 * implementations are allowed to throw exceptions for any operation
 * involving elements they deem ineligible. For absolute safety the
 * specified collections should contain only elements which are
 * eligible elements for both collections.
 *
 * <p>Note that it is permissible to pass the same collection in both
 * parameters, in which case the method will return {@code true} if and
 * only if the collection is empty.
 *
 * @param c1 a collection
 * @param c2 a collection
 * @return {@code true} if the two specified collections have no
 * elements in common.
 * @throws NullPointerException if either collection is {@code null}.
 * @throws NullPointerException if one collection contains a {@code null}
 * element and {@code null} is not an eligible element for the other collection.
 * (<a href="Collection.html#optional-restrictions">optional</a>)
 * @throws ClassCastException if one collection contains an element that is
 * of a type which is ineligible for the other collection.
 * (<a href="Collection.html#optional-restrictions">optional</a>)
 * @since 1.5
 */
public static boolean disjoint(Collection<?> c1, Collection<?> c2) {
    // The collection to be used for contains(). Preference is given to
    // the collection who's contains() has lower O() complexity.
    Collection<?> contains = c2;
    // The collection to be iterated. If the collections' contains() impl
    // are of different O() complexity, the collection with slower
    // contains() will be used for iteration. For collections who's
    // contains() are of the same complexity then best performance is
    // achieved by iterating the smaller collection.
    Collection<?> iterate = c1;

    // Performance optimization cases. The heuristics:
    //   1. Generally iterate over c1.
    //   2. If c1 is a Set then iterate over c2.
    //   3. If either collection is empty then result is always true.
    //   4. Iterate over the smaller Collection.
    if (c1 instanceof Set) {
        // Use c1 for contains as a Set's contains() is expected to perform
        // better than O(N/2)
        iterate = c2;
        contains = c1;
    } else if (!(c2 instanceof Set)) {
        // Both are mere Collections. Iterate over smaller collection.
        // Example: If c1 contains 3 elements and c2 contains 50 elements and
        // assuming contains() requires ceiling(N/2) comparisons then
        // checking for all c1 elements in c2 would require 75 comparisons
        // (3 * ceiling(50/2)) vs. checking all c2 elements in c1 requiring
        // 100 comparisons (50 * ceiling(3/2)).
        int c1size = c1.size();
        int c2size = c2.size();
        if (c1size == 0 || c2size == 0) {
            // At least one collection is empty. Nothing will match.
            return true;
        }

        if (c1size > c2size) {
            iterate = c2;
            contains = c1;
        }
    }

    for (Object e : iterate) {
        if (contains.contains(e)) {
           // Found a common element. Collections are not disjoint.
            return false;
        }
    }

    // No common elements were found.
    return true;
}

```