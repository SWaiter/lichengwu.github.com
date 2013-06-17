---
layout: post
title: "Garbage First(G1)����"
description: ""
category: jvm
subhead: ''
tags: [jvm, java, G1]

---

#### ���ܣ�

Garbage First(G1)�������ڶ�CPU�ʹ��ڴ�������϶������ռ��ṩ��ʵʱĿ��(soft real-time goal )�͸�������(high throughput )����JDK 6u14��ʼ���Ѿ���Hotspot�����飬�����ڵ�DK7��Ȼû���߳�ʵ���ң�
  
    #java -version
    java version "1.7.0_03"
    Java(TM) SE Runtime Environment (build 1.7.0_03-b05)
    Java HotSpot(TM) 64-Bit Server VM (build 22.1-b02, mixed mode)
  
    #java -server -XX:+PrintFlagsFinal | grep UseG1GC
    bool UseG1GC                                   = false           {product}
 
#### ����region��

��G1�У�heap��ƽ���ֳ����ɸ���С��ȵ�����(region)��ÿ��region����һ��������remembered set (RS)��RS�����ݽṹ��hash table�������������card table (heap��ÿ512byteӳ����card table 1byte)���򵥵�˵RS������ڵ���region��live objects��ָ�롣��region�����ݷ����仯ʱ�����ȷ�ӳ��card table�е�һ������card�ϣ�RSͨ��ɨ���ڲ���card table��֪region���ڴ�ʹ������ʹ�����

#### �����ڴ���䣺

����G1��Ҫ��ע�ڶ�CPU���̣߳������ڴ������� thread-local allocation buffers (TLABs)������ÿ�������̶߳���һ���Լ���buffers����������󣬵�buffers������߲�����ʱ��ȥ��������һ���ڴ�����Լ���thread-local���档����������ڴ���䱻��С����˽�е�buffers���棬�����˲��������ڴ��ѹ����

��region�������󣬷����ڴ���̻߳�����ѡ��һ���µ�region����region����֯��һ��linked list���棬�������Կ����ҵ��µ�region��

���ڴ����ķ��䲻����TLABs���еģ�������TLABs֮�⡣��һ������Ĵ�С����region��3/4��ʱ�����������Ϊ�Ǿ޴��(humongous )���޴�Ķ��󱻷��䵽���������(heap regions )����Щ����ֻ�����޴����(humongous object )��

#### ִ�й��̣�

* ��ʼ��� :Initial Marking
* ������� :Concurrent Marking
* ���ձ�� :Final Marking
* ���������� :Live Data Counting and Cleanup

G1ִ�еĵ�һ�׶��ǳ�ʼ���(Initial Marking ),����׶���STW(Stop the World )�ģ�����mutator threads����ֹͣ����ǳ���GC Root��ʼֱ�ӿɴ�Ķ���Ȼ������mutator threads�������������벢�����(Concurrent Marking )�׶Ρ�����׶δ�GC Root��ʼ��heap�еĶ����ǣ�����߳���Ӧ�ó����̲߳���ֱ�ӣ���ʱ�ϳ��������������ɺ󣬿�ʼ���ձ��(Final Marking )�׶Ρ�����׶���Ҫ�Ǳ����Щ�ڲ�����ǽ׶η����仯�Ķ���ͬ�����ձ��ҲҪSTW�����Ƕ������̲߳������У��ܿ�Ϳ�����ɡ����һ���׶λ��ÿ������(region)�Ļ��ճɱ��ͼ�ֵ�������򣬸����û�ָ����ͣ��ʱ�䣬ѡ���Ե��ռ�ĳЩ����Ķ��󣬲�ͳ��ÿ����������������

##### ��CMS�Աȣ�

������˵��G1��CMSһ������һ�����ʱ���ռ�����ͬ������������������������֮��õ��˺ܺõ�Ȩ�⡣
G1��CMS�Ա���һ�²�ͬ��

**1.�ִ���** CMS�У��ѱ���ΪPermGen��YoungGen��OldGen����YoungGen�ַ�������survivo������G1�У��ѱ�ƽ���ֳɼ�������(region)����ÿ�������У���ȻҲ���������ϴ��ĸ�������ռ���������������Ϊ��λ�ռ��ġ�

**2.�㷨��** �����CMS�ġ����??????�����㷨��G1��ʹ��ѹ���㷨����֤�������������Ƭ���ռ��׶Σ�G1�Ὣĳ��������Ķ��󿽱�����������Ȼ�����������������ա�

**3.ͣ��ʱ��ɿأ�** Ϊ������ͣ��ʱ�䣬G1������Ԥ��ͣ��ģ�ͣ��������û����õ�ͣ��ʱ�䷶Χ�ڣ�G1��ѡ���ʵ�����������ռ���ȷ��ͣ��ʱ�䲻�����û�ָ��ʱ�䡣

{% include JB/setup %}
