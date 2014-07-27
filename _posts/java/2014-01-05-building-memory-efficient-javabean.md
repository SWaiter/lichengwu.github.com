---
layout: post
title: "创建节约内存的JavaBean"
description: "如果编写节约内存的java对象"
category: java
subhead: '如果编写节约内存的java对象'
tags: [java,  jvm]
---


编写Java代码的时候，大多数情况下，我们很少关注一个Java对象究竟有多大(占据多少内存)，更多的是关注业务与逻辑。但是殊不知，在我们不经意间，大量的内存被无形地浪费了。

### 一个Java对象到底有多大？

想要精确计算一个Java对象占用的内存，首先要了解Java对象的结构表示。

#### Java对象结构

一个Java对象在Heap的表示，可以分为三部分：

* [Object Header](http://cr.openjdk.java.net/~stefank/6989984.1/raw_files/new/src/share/vm/oops/markOop.hpp)
* Class Pointer
* Fields

每个普通Java对象在堆(heap)中都有一个头信息(object header)，头信息是必不可少的，记录着对象的状态。

![](http://i1298.photobucket.com/albums/ag53/lichengwu/Object_Header_zpsbaadc065.png)

32位与64位占用空间不同，在32位中：

`hash(25)+age(4)+lock(3)=32bit`

64位中：

`unused(25+1)+hash(31)+age(4)+lock(3)=64bit`

我们知道，在Java中，一切皆对象；每个类都有一个父类，`Class Pointer`就是当前对象父类的一个指针，在32位系统中，这个指针为4byte；在64位系统中，如果开启指针压缩(-XX:+UseCompressedOops)或者JVM堆的最大值小于32G，这个指针也是4byte，否则是8byte。

关于字段(Fields)，这里指的是类的实例字段；也就是说不包括静态字段，因为这个字段是共享内存的，只会存在一份。

下面以32位系统为例子，计算一下java.lang.Integer到底占用多大内存：

Object Header 和 Pointer 都是固定的，4+4=8byte；再看看字段，只有这一个，表示数值：

    /**
     * The value of the <code>Integer</code>.
     *
     * @serial
     */
    private final int value;

一个int在java中占据4byte，所以Integer的大小为4+4+4=12byte。

这个结果对吗？不对！还有一点没有说：`在java，对象占用的heap大小是8位对齐的`，上面的12byte没有对齐，所以需要补位4byte。结果是16byte！

另外，在Java中还有一种特殊的对象，`数组`！没错，这个对象有点特殊，它比其他对象多了一个属性：长度(length)。所以我们计算数组长度的时候，需要额外加上一个长度的字段，即一个int的大小。

例如：int[] arr = new int[10];

arr的占用heap大小为：

    4(object header)+4(pointer)+4(length)+4*10(10个int大小)=52byte
由于需要8位对齐，所以最终大小为`56byte`。

### 节约内存原则
在了解了对象的内存使用情况后，我们可以简单算一笔帐。一个java.lang.Integer占用16byte，而一个int占用4byte，4:1的比例！也就是说整数的类类型是基本类型内存的4倍！由此我们得出第一个节约内存的原则：

####(1)尽量使用基本类型，而不是包装类型。

数据库建表的时候字段类型需要仔细推敲，同样JavaBean中的属性字段类型也需要仔细斟酌。不要吝啬使用short，byte，boolean，如果短类型能放下数据，尽量不要使用更长的类型。一个long比一个int才多4byte，但是你要想，如果内存中有100W个long，那就白白浪费了约4MB空间，不要小看这一点点的空间浪费，因为随便一个跑着在线应用的JVM中，对象都能达到上千万！内存是节省出来的。所以：

####(2)斟酌字段类型，在满足容量前提下，尽量用小字段。

你知道一个ArrayList&lt;Integer&gt;集合，如果里面放了10个数字，占用多少内存吗？让我们算算：

ArrayList&lt;Integer&gt;中有两个字段：

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer.
     */
    private transient Object[] elementData;

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;

Object Header占4byte，Pointer占4byte，一个int字段(size)占4byte，elementData数组本身占12(4+4+4)，数组中10个Integer对象占10×16。所以整个集合空间大小为4+4+4+12+160=184byte。

如果我们用int[]代替集合呢，12+4×10=52byte，对其后56byte。

集合跟数组的比例是184:56，超过3:1了！

所以我们的第三个建议是：

####(3)如果可能，尽量用数组，少用集合。

`数组中是可以使用基本类型的，但是集合中只能放包装类型！`

如果实在需要使用集合，推荐一个比较节约内存的集合工具，[fastutil](http://fastutil.di.unimi.it/)。这里面包含了JKD集合中绝大部分的实现，而且比较省内存。

####(4)小技巧。

在上面的三个原则基础上，提供两个小技巧。

* 时间用long/int表示，不用Date或者String。
* 短字符串如果能穷举或者转换成ascii表示，可以用long或者int表示。

小技巧跟具体的场景是数据有关系，可以根据实际情况进行激进优化节省内存。

### 总结
性能和可读性向来就有些矛盾，在这里也是，为了节约内存，不得不进行取舍，代码丑陋了一些，可读性差了一些，还好能省下一些内存。上面的原则在确实需要节约内存的时候，不妨可以试试！

{% include JB/setup %}


