---
title: 总结学习HashMap和HashTable
date: 2023-04-14 14:20:08
tags:
	- HashMap
	- Hashtable
categories:
	- 代码人生
toc: true
---

HashMap 和 Hashtable 都用于以键和值的形式存储数据。两者都使用散列技术来存储唯一密钥。但是HashMap和Hashtable 类之间也是有许多区别。

<!-- more -->

## 1.HashMap是不同步的，即非线程安全；Hashtable是同步的，即线程安全。

HashMap部分源码:

```java

// get
public V get(Object key) {
    HashMap.Node e;
    return (e = this.getNode(hash(key), key)) == null ? null : e.value;
}

...
...


// put
public V put(K key, V value) {
    return this.putVal(hash(key), key, value, false, true);
}

...
...


// remove
public V remove(Object key) {
    HashMap.Node e;
    return (e = this.removeNode(hash(key), key, (Object)null, false, true)) == null ? 
    		null : e.value;
}

...
...


```


HashTable部分源码:

```java
// get
public synchronized V get(Object key) {
    Hashtable.Entry<?, ?>[] tab = this.table;
    int hash = key.hashCode();
    int index = (hash & 2147483647) % tab.length;

    for(Hashtable.Entry e = tab[index]; e != null; e = e.next) {
        if (e.hash == hash && e.key.equals(key)) {
            return e.value;
        }
    }

    return null;
}

...
...


// put
public synchronized V put(K key, V value) {
    if (value == null) {
        throw new NullPointerException();
    } else {
        Hashtable.Entry<?, ?>[] tab = this.table;
        int hash = key.hashCode();
        int index = (hash & 2147483647) % tab.length;

        for(Hashtable.Entry entry = tab[index]; entry != null; entry = entry.next) {
            if (entry.hash == hash && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        this.addEntry(hash, key, value, index);
        return null;
    }
}

...
...


// remove
public synchronized V remove(Object key) {
    Hashtable.Entry<?, ?>[] tab = this.table;
    int hash = key.hashCode();
    int index = (hash & 2147483647) % tab.length;
    Hashtable.Entry<K, V> e = tab[index];

    for(Hashtable.Entry prev = null; e != null; e = e.next) {
        if (e.hash == hash && e.key.equals(key)) {
            if (prev != null) {
                prev.next = e.next;
            } else {
                tab[index] = e.next;
            }

            ++this.modCount;
            --this.count;
            V oldValue = e.value;
            e.value = null;
            return oldValue;
        }

        prev = e;
    }

    return null;
}

...
...

```

## 2.HashMap可以通过Collections.synchronizedMap(Map<K, V> m)实现同步；Hashtable不能实现非同步。

>虽然HashMap不是线程安全的，但是我们可以通过`Collections.synchronizedMap(Map<K, V> m)`实现线程安全.

```java
public class App {

    private static AtomicInteger atomicInteger = new AtomicInteger();

    public static void main(String[] args) throws InterruptedException {
        Map<Integer, Integer> m = new HashMap<>();
        Map<Integer, Integer> map = Collections.synchronizedMap(m);
        //创建线程池
        ExecutorService threadPool = Executors.newFixedThreadPool(10);
        for(int i = 0 ; i < 10000 ; i++) {
            //调用execute()方法创建线程
            threadPool.execute(() -> map.put(atomicInteger.incrementAndGet(), 
            		(int)(Math.random()*100))
        	);
        }
        // 关闭线程池
        threadPool.shutdown();
        threadPool.awaitTermination(1000, TimeUnit.SECONDS);
        System.out.println(map.size());
    }
}
```

## 3.HashMap允许一个空键和多个空值；HashTable不允许任何空键和空值

从HashTable的源码可以看到，如果key或value是null，会抛出`NullPointerException`

```java
/**
 * Maps the specified <code>key</code> to the specified
 * <code>value</code> in this hashtable. Neither the key nor the
 * value can be <code>null</code>. <p>
 *
 * The value can be retrieved by calling the <code>get</code> method
 * with a key that is equal to the original key.
 *
 * @param      key     the hashtable key
 * @param      value   the value
 * @return     the previous value of the specified key in this hashtable,
 *             or <code>null</code> if it did not have one
 * @exception  NullPointerException  if the key or value is
 *               <code>null</code>
 * @see     Object#equals(Object)
 * @see     #get(Object)
 */
public synchronized V put(K key, V value) {
    // Make sure the value is not null
    if (value == null) {
        throw new NullPointerException();
    }

    // Makes sure the key is not already in the hashtable.
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }

    addEntry(hash, key, value, index);
    return null;
}
```

## 4.HashMap是JDK 1.2中引入的新类. Hashtable是JDK 1.0中的类

## 5.HashMap比Hashtable更快.

## 6.HashMap由Iterator实现遍历. Hashtable由Enumerator和Iterator实现遍历.

## 7.HashMap中的迭代器是快速失败机制. Hashtable是安全失败机制.

HashMap不是线程安全的，在遍历HashMap的内容时，如果有其他线程修改了HashMap的内容，那么将抛出`ConcurrentModificationException`。

```java
// iterators

abstract class HashIterator {
    Node<K,V> next;        // next entry to return
    Node<K,V> current;     // current entry
    int expectedModCount;  // for fast-fail
    int index;             // current slot

    HashIterator() {
        expectedModCount = modCount;
        Node<K,V>[] t = table;
        current = next = null;
        index = 0;
        if (t != null && size > 0) { // advance to first entry
            do {} while (index < t.length && (next = t[index++]) == null);
        }
    }

    public final boolean hasNext() {
        return next != null;
    }

    final Node<K,V> nextNode() {
        Node<K,V>[] t;
        Node<K,V> e = next;
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        if (e == null)
            throw new NoSuchElementException();
        if ((next = (current = e).next) == null && (t = table) != null) {
            do {} while (index < t.length && (next = t[index++]) == null);
        }
        return e;
    }

    public final void remove() {
        Node<K,V> p = current;
        if (p == null)
            throw new IllegalStateException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        current = null;
        K key = p.key;
        removeNode(hash(key), key, null, false, false);
        expectedModCount = modCount;
    }
}
```

modCount用于记录修改次数，对HashMap的修改都将增加这个值，在迭代器初始化过程中会将modCount传递给expectedModCount。在迭代中就是根据`modCount != expectedModCount`判断Map是否已被其他线程修改。


Hashtable是[fail-safe 安全失败](https://blog.csdn.net/striner/article/details/86375684)机制

fail-safe:这种遍历基于容器的一个克隆。因此，对容器内容的修改不影响遍历。java.util.concurrent包下的容器都是安全失败的,可以在多线程下并发使用,并发修改。常见的的使用fail-safe方式遍历的容器有ConcerrentHashMap和CopyOnWriteArrayList等。

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发Concurrent Modification Exception。

缺点：基于拷贝内容的优点是避免了ConcurrentModificationException，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。

## 8.HashMap继承AbstractMap类；Hashtable继承Dictionary类.