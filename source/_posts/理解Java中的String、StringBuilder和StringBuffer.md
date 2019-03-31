---
title: 理解Java中的String、StringBuilder和StringBuffer
date: 2019-03-29 14:00:43
tags:
- Java
- String
- StringBuilder
- StringBuffer
categoties:
- 编程语言
---

String是我们学习和使用Java最常用的类之一，我们经常在网上看一些Java的面试题，大多数都也会提到关于String、StringBuilder、StringBuffer的区别，因此我们有理由要好好了解了解。而想要了解一个类，最好的办法就是看这个类的实现源代码。

<!-- more -->

#### ⚪ 理解String

String类的实现在`jdk1.6.0_14srcjavalangString.java`文件中。

打开这个类文件就会发现String类是被final修饰的：

```java
public final class String
   implements java.io.Serializable, Comparable<String>, CharSequence
{
   /** The value is used for character storage. */
   private final char value[];

   /** The offset is the first index of the storage that is used. */
   private final int offset;

   /** The count is the number of characters in the String. */
   private final int count;

   /** Cache the hash code for the string */
   private int hash; // Default to 0

   /** use serialVersionUID from JDK 1.0.2 for interoperability */
   private static final long serialVersionUID = -6849794470754667710L;

   ......

}
```

从上面可以看出几点：

1）String类是final类，也即意味着String类不能被继承，并且它的成员方法都默认为final方法。在Java中，被final修饰的类是不允许被继承的，并且该类中的成员方法都默认为final方法。在早期的JVM实现版本中，被final修饰的方法会被转为内嵌调用以提升执行效率。而从Java SE5/6开始，就渐渐摈弃这种方式了。因此在现在的Java SE版本中，不需要考虑用final去提升方法调用效率。只有在确定不想让该方法被覆盖时，才将方法设置为final。

2）上面列举出了String类中所有的成员属性，从上面可以看出String类其实是通过char数组来保存字符串的。

下面再继续看String类的一些方法实现：

```java
public String substring(int beginIndex, int endIndex) {
   if (beginIndex < 0) {
       throw new StringIndexOutOfBoundsException(beginIndex);
   }
   if (endIndex > count) {
       throw new StringIndexOutOfBoundsException(endIndex);
   }
   if (beginIndex > endIndex) {
       throw new StringIndexOutOfBoundsException(endIndex - beginIndex);
   }
   return ((beginIndex == 0) && (endIndex == count)) ? this :
       new String(offset + beginIndex, endIndex - beginIndex, value);
   }

public String concat(String str) {
   int otherLen = str.length();
   if (otherLen == 0) {
       return this;
   }
   char buf[] = new char[count + otherLen];
   getChars(0, count, buf, 0);
   str.getChars(0, otherLen, buf, count);
   return new String(0, count + otherLen, buf);
   }

public String replace(char oldChar, char newChar) {
   if (oldChar != newChar) {
       int len = count;
       int i = -1;
       char[] val = value; /* avoid getfield opcode */
       int off = offset;   /* avoid getfield opcode */

       while (++i < len) {
       if (val[off + i] == oldChar) {
           break;
       }
       }
       if (i < len) {
       char buf[] = new char[len];
       for (int j = 0 ; j < i ; j++) {
           buf[j] = val[off+j];
       }
       while (i < len) {
           char c = val[off + i];
           buf[i] = (c == oldChar) ? newChar : c;
           i++;
       }
       return new String(0, len, buf);
       }
   }
   return this;
```

从上面的三个方法可以看出，无论是sub操作、concat操作还是replace操作都不是在原有的字符串上进行的，而是重新生成了一个新的字符串对象。也就是说进行这些操作后，最原始的字符串并没有被改变。

在这一永远记住一点

> 对String对象的任何改变都不影响到原对象，相关的任何change操作都会生成新的对象。

在了解了于String类基础的知识后，下面来看一些在平常使用中容易忽略和混淆的地方。
