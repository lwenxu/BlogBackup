---
title: 数据结构Generic
date: 2018-03-02 16:50:14
tags: 数据结构
---

​	接下来我们要处理的是前面实现里另一个 根本性的缺陷 那些实现只适用于字符串，想要实现其他类型数据的队列和栈怎么办呢？ 这个问题就涉及泛型的话题了。<!--more-->

​	有一个广泛采用的捷径是 使用强制类型转换对不同的数据类型重用代码 我们对Object类实现数据结构，Java中所有的类都是Object的 子类，当客户端使用时，就将结果转换为 对应的类型。这个我不想花很多时间来讲 因为我认为这样的解决方案不能令人满意。

​	第二种方法是用的是泛型 这种方法中客户端程序不需要强制类型转换。在编译时就能 发现类型不匹配的错误，而不是在运行时。优秀的模块化编程的指导原则就是我们应当欢迎编译时错误，避免运行时错误。因为如果我们能在编译时 检测到错误，我们给客户交付产品或者部署对一个API的实现时 有把握对于任何客户都是没问题的，然而 直到运行时才会出现的错误可能在某些客户的开发中几年之后出现。

​	基于数组的实现，这种方法不管用。目前很多编程语言 这方面都有问题，而对Java尤其是个难题 我们想做的是用泛型名称item直接声明一个新的数组， 不幸的是，Java不允许创建泛型数组。对于这个问题有各种 技术方面的原因。这里，要行得通我们需要 加入强制类型转换。我们创建Object数组，然后将类型转换为 item数组。我的观点是优秀的代码应该不用强制类型转换。要 尽量避免强制类型转换因为它确实在我们的实现中 留下隐患。但这个情况中我们必须加入这个强制类型转换 我们听到过的教导是蹩脚的强制类型转换让你看你的代码不爽 这样的想法不仅仅你一个人有 我认为像这么简单的代码强制类型转换是讨厌的特性。当我们编译这个程序的 时候，Java会发出警告信息说我们在使用未经检查 或者不安全的操作，详细信息需要使用-Xlint=unchecked参数 重新编译。我们加上这个参数重新编译之后显示 你在代码中加入了一个未经检查的强制类型转换，对此发出 警告，你不应该加入未经检查的强制类型转换。好吧，当你 编译这样的代码的时候看到这个警告信息没事。

​	接下来，是个跟Java有关的 细节问题，关于基本类型。我们用的泛型类型是针对 Object及其子类的。前面讲过，是从Object数组强制类型转换 来的。为了处理基本类型，我们需要使用Java的包装对象类型 如大写的Integer是整型的包装类型等等。