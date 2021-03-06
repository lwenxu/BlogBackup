---
title: Java位操作
date: 2017-08-09 09:34:32
tags: Java
categories:
  -Java
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;无论说是在哪一门计算机语言，位操作运算对于计算机来说肯定是最高效的，因为计算机的底层是按就是二进制，而位操作就是为了节省开销，加快程序的执行速度，以及真正的实现对数的二进制操作。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用位操作，很多代码看起来会很简洁，并且执行速度也会随之提高。在大多数编程语言中都会有 << 和 >>
这两个符号向左的就是左移，反之则是右移这个符号的左边就是需要操作的数，而右边就代表了对这个数移动多少位。
<!--more-->
#### 1.具体位操作
1. 左移( << )：
左移几位就是将这个数再乘以2的几次方，例如说 4 << 2  其结果就是16，也就是将这个数化作为2进制的数然后向左移动两位，最右边的空位就补0.
2. 右移( >> )：
右移就刚好相反，但是也不是完全一样，他是向右移动 n 位，如果说这个数本来就是正的，那么和左移刚好相反就直接除以 2 的 n 次方位，但是如果是负数的话在这个数向右移动 n 位后我们在前面的空位补的是 0 。也就是右移的话是与数相关的问题。右移一个很明显的应用就是在二分法的时候我们就可以直接右移一位，显然速度会提高。
3. 超级右移( >>> )：
刚刚说了右移其实还是需要按照情况来的，有时候就不一定是正数，我们就可能补 1 ，但是我们期望结果就是这个数除以 2 的 n 次方，我们就可以使用这个无视正负号的右移操作 >>> ，也就是说他是在任何情况下都是给最高位添加 0 。
4. 与操作( & ):
与操作就是把两个数转化为二进制的数，然后再把这两个数，从最低位每位对其，同 1 结果为 1 否则全为 0。
5. 或操作( | ):
操作同上只是这个是同 0 为 0，其他都是1。
6. 取反操作( ~ ):
二进制的0 变 1 ， 1 变 0。
7. 异或( ^ ):
异或有一条很重要的性质，用的非常多就是一个数异或同一个数两次结果还是那个数。

上面的与或操作会发现他们有单符号的还有双符号的，不要搞混了单符号的不仅仅就是位操作，他们还是逻辑操作，而双符号的仅仅就是逻辑操作。并且他们有区别例如 & 和 && 当他们都作为逻辑操作的，前者就是对一个表达式一直判断完毕才会出现他的值，而后者则是判断一半如果知道为假或真他就不再判断了，这也就是我们看到的大多数的 if 判断中是用的双与，而非单与。

#### 2.实际应用：
1. 第一个就是两个数交换，这个一般有三种方式：  
第一个：临时变量  
```Java
int i=3,j=8,temp=0;
temp=i;
i=j;
j=temp;
```
第二个：使用加减法
```Java
int i=3,j=8;
i=j+i;
j=i-j;
i=i-j;
```
第三个：位操作
```Java
int i=3,j=8;
i=i^j;
j=i^j;
i=i^j;
```
这个地方就是用了异或的重要性质

2. 第二个就是进制转换了：
基本思路就是先把数转为二进制的数，然后如果要 16 进制那么就4位取，8进制3位取，但是又怎么取这个4位或者3位呢，这里与操作就能派上用场取四位我们可以直接与上 15 ，三位就是 7 了，例如：
```Java
int num=60;
int n1=num & 15;
int tmp=num >>> 4;
int n2=tmp & 15;
System.out.println("n1: "+n1+" n2 "+n2);
```
