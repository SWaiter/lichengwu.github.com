---
layout: post
catalog: true
title: "JNI入门 一个JNI的HelloWorld 程序"
description: ""
category: java
subhead: ''
tags: [.net,  java,  JNI]

---

JNI是java本地编程接口。是 `Java Native Interface`的英文缩写。他能够使java代码与用其他编程语言编写的应用程序和库进行互操作。（其他编程语言大多是c,c++和汇编语言。）


下面来写一个间的HelloWorld程序。首先启动Eclipse 新建一个java工程：


![](/images/java/1_zps15ec1abd.png)

新建一个class 注意：必须在类中声明一个`native`方法:

    package org.gunct;
    public class JniDemo
    {
        /**
         * sayHello
         * 使用JNI时 在java类中必须声明一个native方法
         * 在java中不用实现这个方法，只声明就行了
         * 
         */
        public native void sayHello();
        public static void main(String[] args)
        {
        }
    }

 

接下来是c/c++的本地代码实现了:
写在c/c++中 sayHello()函数的声明。这里需要用到jdk提供的一个工具`javah.exe`,在jdk安装目录/bin下面。

![](/images/java/2_zps789b7160.png)

下进行下面操作时，确保你的系统已经设置好了java的环境变量,使用javah命令生成包含c/c++的头文件

javah的用法:

![](/images/java/3_zps26951292.png)

首先进入刚才写的JniDemo对应的class路径
然后运行命令


![](/images/java/4_zps53383591.png)

回到目录会发现已经生成好了头文件


![](/images/java/5_zps04bab569.png)

接下来启动Microsoft Visual Studio新建一个win32控制台项目


![](/images/java/8_zps3676a008.png)

为项目创建dll


![](/images/java/6_zps63ffd0c3.png)

创建C++文件


![](/images/java/8_zps3676a008.png)

将刚才生成的头文件剪切到c++项目中，因为在c++编译过程中要用到jdk提供的头文件jni.h（jdk目录/include下面）和jni\_md.h（jdk目录/include/win32下面），把这两个文件也拷贝到c++项目中。

![](/images/java/9_zpsec35a514.png)

回到Microsoft Visual Studio项目中，将这三个头文件添加到项目中


![](/images/java/10_zps24def977.png)
 
接下来编写sayHello.cpp 代码如下：


    #include <iostream>
    #include "org_gunct_JniDemo.h"
    using namespace std;
    //重写方法
    JNIEXPORT void JNICALL Java_org_gunct_JniDemo_sayHello(JNIEnv * env, jobject obj)
        cout<<"Hello World"<<endl;
    }

编译后在Release目录看到jniCode.dll文件


![](/images/java/11_zpsf3faa6fc.png)

将这个文件拷贝到C:/WINDOWS/system32下面

编写代码：

    package org.gunct;
    public class JniDemo
    {
        /**
         * sayHello
         * 使用JNI时 在java类中必须声明一个native方法
         * 在java中不用实现这个方法，只声明就行了
         */
        public native void sayHello();
        public static void main(String[] args)
        {
            System.loadLibrary("jniCode");
            JniDemo d = new JniDemo();
            d.sayHello();
        }
    }


运行后输出结果：

![](/images/java/12_zpse71ce87d.png)

##### 使用JNI的弊端 ：

由于c/c++不是跨平台的，所以使用JNI后的java程序就不能跨平台了。

Java是强类型的语言，c/c++不是。
所以应该尽量少使用JNI。


