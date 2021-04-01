---
title: Java 虚拟机（一）
date: 2017-04-05 22:59:37
tags:
	- Java
categories: Java
---

## Java 虚拟机

### Java 运行时数据区

Java 虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域。不同的区域都有各自的用途，以及创建和销毁的时间，有的区域随着虚拟机进程的启动而存在，有的则随着线程的启动和结束而建立和销毁。数据区分为5大区域：程序计数器，Java虚拟机栈，本地方法区，Java堆，方法区。
<!-- more -->

![](/images/jvm-data.png)

#### 程序计数器
内存中较小的一块空间，可以当做是当前线程所执行的字节码的行号指示器。简单点其实应该算是空着着当前线程的下一步操作。Java虚拟机的多线程其实是线程轮流分配执行时间的方式来实现的，意思就是一个处理器在任何一个确定的时间，只会执行一条线程的指令。想想如果线程没执行完，cpu的时间没了，另一个线程占去了cpu，那么下次cpu如何知道当前线程执行到哪里了呢？要解决这个问题，是不是的有个区域必须的保持独立性，不能被非当前线程干扰？所以把程序计数器设置成***```线程私有```***，这样就完美解决这个问题了。如果线程执行的是Java方法，计数器就是记录的正在执行的虚拟机字节码指令地址。如果正在执行的是native方法，计数器的值就为空(undefined)。另外非常要注意的点就是程序计数器是***唯一一个没有规定任何OutOfMemoryError***。

#### Java 虚拟机栈
虚拟机栈也是线程私有的，它的生命周期与线程相同。都跟线程同生共死了，当然虚拟机栈也是***线程私有的***。虚拟机栈描叙的是java方法执行的内存模型：每个方法在执行的时候都会创建一个栈帧，栈帧主要用来存储局部变量表、操作数栈、动态链接、方法出口的信息。每一个方法从调用到完成，对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。经常说方法中的局部变量是线程安全的，从这里可以得到解释，局部变量定义在虚拟机栈中，那么它是属于线程私有的，就不会存在多线程中共享变量的情况，所以说局部变量在多线程环境下是线程安全的。栈的局部变量表存放了编译期（注意是`编译`，不是`编辑`）可知的各种基本数据类型（boolean,byte,char,short,int,float,long,double)、对象引用和returnAddress类型。
局部变量表的最小单位为局部变量空间（slot）。其中64位长度的long和double类型的数据会占用2个局部变量空间。局部变量表所需的内存空间在编译期间就完成分配，进入一个方法需要在栈中分配多大的局部变量空间是完全确定的，运行期间不会改变局部变量表的大小。
Java虚拟机规范中，如果这个区域线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverFlowError异常；如果虚拟机栈可以动态扩展，在扩展期间无法申请到足够的内存，就会抛出OutOfMemoryError异常。

#### 本地方法栈
本地方法栈与虚拟机栈所发挥的作用比较类型，区别是虚拟机栈为虚拟机执行Java方法服务，而本地方法栈则为虚拟机使用到的Native方法服务。很多Java的源代码中就有这种native方法。此区域也会抛出StackOverFlowError 和 OutOfMemoryError。

#### Java 堆
Java虚拟机所管理的内存中最大的一块。Java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。Java堆得唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。Java堆是垃圾收集器管理的重要区域，因此也被称为`GC堆`（Garbage Collected Heap）堆中还细分为新生代和老年代。Java堆可以处于物理不连续的内存空间中，只要逻辑上是连续的就可以。如果堆中内存完成实例分配，并且堆也无法扩展的时候，将会抛出OutOfMemoryError异常。

#### 方法区
方法区与Java堆一样，是各个线程共享的内存区域，它用于内存已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。该区域除了和Java堆一样不需要连续的内存和可以选择固定大小或者可扩展外，还可以选择不实现垃圾收集。按照这一块的存在目的，主要是存储加载的类型信息，常量等信息。这块的回收主要是类型的写在和常量池的回收，回收的效益并不一定会很理想。当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。