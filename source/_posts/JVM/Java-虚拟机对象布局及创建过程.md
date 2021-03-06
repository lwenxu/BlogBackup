---
title: Java 虚拟机对象布局及创建过程
date: 2017-12-10 21:38:50
tags: Java 虚拟机
categories:
	-Java 
	-虚拟机
---



# 一. 对象创建过程

### 一、类加载

​	这个阶段其实主要做的就是类的加载验证，以及类的准备等等一系列的工作。为后面的类的初始化和解析做铺垫。

<!--more-->

### 二、类解析

​	这个阶段主要对上面已经加载入内存的类进行词法语法的解析，形成语法树等操作。

### 三、类初始化

​	类初始化执行的就是 \<clinit> 方法，以及在类中的各种常量的初始化（放入常量池中的常量）。

### 四、分配内存

​	虚拟机对所需要创建的对象分配内存，在类的解析的时候其实已经知道实例变量需要的堆空间了。此时只需要调用操作系统的系统调用为当前的对象分配内存即可。

​	但是为对象分配内存如果是在大型环境下，肯定是并发的也就是可能会出现正当当前指针分配内存的时候另外一个线程把指针指向了别处，这样一来自然就会出现多线程的安全问题。这里 jvm 通常采用两种方式来解决：

* 使用操作系统的CAS算法来分配内存，保证分配内存动作的原子性，CAS 也就是 Compare And Save 其算法的基本思想就是拿线程需要改变的变量写入内存之前从内存中读取的值 A 和该变量的最初从内存读取的值 V 进行比较如果他们相等就可以将当前的修改后的值 B 写入内存，否则则放弃当前的操作重新开始尝试。
* 另外一种就是使用了线程的临时分配表叫做 TLAB 表，每个线程获得一块这样的内存这样一来每个线程就拿到了相互独立的内存空间，只有当 TLAB 表分配没有剩余的时候才使用 CAS 同步操作。

### 五、内存的初始化

​	虚拟机对这块刚分配的内存进行初始化的时候是非常暴力的或者说简单的，直接使用了 

```c++
memset( bf , 0 ) ;
```

这是真正的 0 值，所以说对象在分配内存以后其实就是可用的，里面的引用类型就是 NULL 而普通变量的内容则是 0 。这个操作其实不一定到现在才进行，如果说我们采用的 TLAB 的方式的话我们只需要在为线程分配 TLAB 表的时候执行那个 memset 函数即可完成内存初始化。

### 六、对象进行必要的初始化

​	在经过上面的暴力的 0 值初始化之后对象还是不能使用的，因为对象不是一块充满数据的内存，对象里面有一些特殊的标记信息，标识当前对象的状态，这个就叫做对象头也称作 MarkWord ，这里对对象的必要的初始化说的就是对对象头进行设置。比如说 GC 分代年龄，hashCode，偏向锁，偏向 ID，时间戳，线程持有的锁等等。

​	至此 jvm 对一个对象的创建才算是真正的完成，但是这个对于 java 程序员来说对象的创建才刚刚开始，因为此时才开始执行用户代码，这个要取决于字节码中是否有 invokespecial 指令，这个指令就会去执行实例的 \<init>

方法来初始化对象。



# 二、对象的内存布局