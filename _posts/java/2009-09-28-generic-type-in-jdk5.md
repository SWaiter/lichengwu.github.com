---
layout: post
catalog: true
title: "JDK5.0新特性---泛型"
description: ""
category: java
subhead: ''
tags: [java,  generic-type]

---
泛型是Java SE 1.5的新特性，泛型的本质是`参数化类型`，也就是说所操作的数据类型被指定为一个参数。这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口、泛型方法。

Java语言引入泛型的好处是安全简单。在Java SE 1.5之前，没有泛型的情况的下，通过对类型`Object`的引用来实现参数的“任意化”，“任意化”带来的缺点是要做显式的强制类型转换，而这种转换是要求开发者对实际参数类型可以预知的情况下进行的。对于强制类型转换错误的情况，编译器可能不提示错误，在运行的时候才出现异常，这是一个安全隐患。
泛型的好处是在编译的时候检查类型安全，并且所有的强制转换都是自动和隐式的，提高代码的重用率。

##### 泛型在使用中还有一些规则和限制：

<> 泛型的参数类型可以使用extends语句，例如```<T extends superclass>```。习惯上成为`有界类型`。

<> 同一种泛型可以对应多个版本【因为参数类型是不确定的】，不同版本的泛型类实例是不兼容的。

<> 泛型的类型参数只能是类类型（包括自定义类），不能是简单类型。

<> 泛型的类型参数可以有多个。

<> 泛型的参数类型还可以是通配符类型。例如`Class<?> classType = Class.forName(java.lang.String)`。

泛型还有接口、方法等等，内容很多，需要花费一番功夫才能理解掌握并熟练应用。在此给出我曾经了解泛型时候写出的两个例子（根据看的印象写的），实现同样的功能，一个使用了泛型，一个没有使用，通过对比，可以很快学会泛型的应用，学会这个基本上学会了泛型大部分的内容。

    publicclassGenericity<T>//泛型T  
    {  
        private T a;  
        public void setA(T a)  
        {  
            this.a=a;  
        }  
        public T getA()  
        {  
            return a;  
        }  
        public static void main(String [] args)  
        {  
            //Integer类型  
            Genericity<Integer>intVal=newGenericity<Integer>();  
            intVal.setA(newInteger(5));  
            System.out.println(intVal.getA());  
            //String类型  
            Genericity<String>strVal=newGenericity<String>();  
            strVal.setA("Gunct");  
            System.out.println(strVal.getA());  
        }  
    }   

