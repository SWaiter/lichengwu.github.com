---
layout: post
catalog: true
title: "Java 中的语法糖 (Syntactic Sugar)"
description: ""
category: java
subhead: ''

tags: [java, syntactic-sugar]

---

语法糖（Syntactic Sugar），也叫糖衣语法，是英国计算机科学家彼得·约翰·兰达(Peter J. Landin)发明的一个术语。指的是，在计算机语言中添加某种语法，这种语法能使程序员更方便的使用语言开发程序，同时增强程序代码的可读性，避免出错的机会；但是这种语法对语言的功能并没有影响。

Java中的泛型，变长参数，自动拆箱/装箱，条件编译等都是，下面做简单的介绍和分析。

#### 泛型 
与C#中的泛型相比，Java的泛型可以算是“伪泛型”了。在C#中，不论是在程序源码中、在编译后的中间语言，还是在运行期泛型都是真实存在的。Java则不同，Java的泛型只在源代码存在，只供编辑器检查使用，编译后的字节码文件已擦除了泛型类型，同时在必要的地方插入了强制转型的代码。

泛型代码：
 
    public static void main(String[] args) {  
        List<String> stringList = new ArrayList<String>();  
        stringList.add("oliver");  
        System.out.println(stringList.get(0));  
    }  
 
 
将上面的代码的字节码反编译后：
  
    public static void main(String args[])  
    {  
        List stringList = new ArrayList();  
        stringList.add("oliver");  
        System.out.println((String)stringList.get(0));  
    }  
 

#### 自动拆箱/装箱 

自动拆箱/装箱是在编译期，依据代码的语法，决定是否进行拆箱和装箱动作。

装箱过程：把基本类型用它们对应的包装类型进行包装，使基本类型具有对象特征。

拆箱过程：与装箱过程相反，把包装类型转换成基本类型。

需要注意的是：包装类型的“==”运算在没有遇到算数运算符的情况下不会自动拆箱，而其包装类型的equals()方法不会处理数据类型转换，所以：

    Integer a = 1;  
    Integer b = 1;  
    Long c = 1L;  
    System.out.println(a == b);  
    System.out.println(c.equals(a));  
 
这样的代码应该尽量避免自动拆箱与装箱。

#### 循环历遍（foreach） 
语法：
 
    List<Integer> list = new ArrayList<Integer>();  
    for(Integer num : list){  
        System.out.println(num);  
    }  
 
Foreach要求被历遍的对象要实现Iterable接口，由此可想而知，foreach迭代也是调用底层的迭代器实现的。反编译上面源码的字节码：
 
    List list = new ArrayList();  
    Integer num;  
    Integer num;  
    for (Iterator iterator = list.iterator(); iterator.hasNext(); System.out.println(num)){  
        num = (Integer) iterator.next();  
    }  
 
#### 条件编辑 
很多编程语言都提供了条件编译的途径，C，C++中使用#ifdef。Java语言并没有提供这种预编译功能，但是Java也能实现预编译。

    if(true){  
        System.out.println("oliver");  
    }else{  
        System.out.println("lee");  
    }  
 
这段代码的字节码反编译后只有一条语句：
  
    System.out.println("oliver");  
 
在编译器中，将会把分支不成立的代码消除，这一动作发生在编译器解除语法糖阶段。
所以说，可以利用条件语句来实现预编译。

#### 枚举 
枚举类型其实并不复杂，在JVM字节码文件结构中，并没有“枚举”这个类型。

其实源程序的枚举类型，会在编译期被编译成一个普通了类。利用继承和反射，这是完全可以做到的。

看下面一个枚举类：

    public enum EnumTest {  
        OLIVER,LEE;  
    }  
 
反编译字节码后：

    public final class EnumTest extends Enum {  
        private EnumTest(String s, int i) {  
            super(s, i);  
        }  
        public static EnumTest[] values() {  
            EnumTest aenumtest[];  
            int i;  
            EnumTest aenumtest1[];  
            System.arraycopy(aenumtest = ENUM$VALUES, 0,  
                    aenumtest1 = new EnumTest[i = aenumtest.length], 0, i);  
            return aenumtest1;  
        }  
        public static EnumTest valueOf(String s) {  
            return (EnumTest) Enum.valueOf(EnumTest, s);  
        }  
        public static final EnumTest OLIVER;  
        public static final EnumTest LEE;  
        private static final EnumTest ENUM$VALUES[];  
        static {  
            OLIVER = new EnumTest("OLIVER", 0);  
            LEE = new EnumTest("LEE", 1);  
            ENUM$VALUES = (new EnumTest[] { OLIVER, LEE });  
        }  
    }  
 
至于更多细节，可以参考父类Enum。

#### 变长参数 
变长参数允许我们传入到方法的参数是不固定个数。

对于这个方法：

    public void foo(String str,Object...args){  
    }  
 
我们可以这样调用：
 
    foo("oliver");  
    foo("oliver",new Object());  
    foo("oliver",new Integer(1),"sss");  
    foo("oliver",new ArrayList(),new Object(),true,1);  
 
参数args可以是任意多个。
其实，在编译阶段，args是会被编译成Object \[\] args。
  
    public transient void foo(String s, Object aobj[])  
    {  
    }  
 
这样，变长参数就可以实现了。

但是要注意的是，变长参数必须是方法参数的最后一项。

----

除了上面介绍的语法糖，还有内部类，断言以及JDK7的switch支持字符串，自动关闭资源（在try中定义和关闭）等。

感兴趣的同学可以反编译字节码了解它们的本质。

无疑，语法糖方便了程序员的开发，提高了开发效率，提升了语法的严谨也减少了编码出错误的几率。我们不仅仅在平时的编码中依赖语法糖，更要看清语法糖背后程序代码的真实结构，这样才能更好的利用它们。


