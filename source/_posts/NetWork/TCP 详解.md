---
title: TCP 总结
date: 2018-03-25 09:40:24
tags: 
categories:
  -计算机网络
---

# TCP 总结

>计算机网络中比较中要的无非就是 TCP/IP 协议栈，以及应用层的 HTTP 和 HTTPS 。
>前几天一直炒的的比较火的就是 HTTP/2.0 了，但是其实 HTTP/2.0 早在2015年的时候就已经出来了，并且这个版本是基于 Google 公司的 SPDY 协议发布的，其实说白了就是用的 SPDY 做了一点修改。
>好了今天的主题是 TCP 就不过多的介绍 HTTP/2.0 了,以后会专门写一篇关于 HTTP/2.0 的文章，介绍一下他的新特性。

<!--more-->
## 1.引言
&emsp;&emsp;我们都知道 TCP 是位于传输层的协议，他还有一个兄弟就是 UDP ，他们两共同构成了传输层。显然他们之间有很大的区别要不然的话在传输层只需要一个就好了。

&emsp;&emsp;其中最重要的区别就是一个面向连接另外一个不是，这个区别就导致了他们是否能够保证稳定传输，显然不面向连接的 UDP 是没办法保证可靠传输的，他只能靠底层的网络层和链路层来保证。我们都知道网络层采用的是不可靠的 IP 协议。好吧，网络层也保证不了可靠传输，所以 UDP 保证可靠传输只能依靠链路层了。

&emsp;&emsp;而 TCP 就好说了他不仅仅有底层的链路层的支持，还有自己的面向链接服务来保证可靠传输。当然 TCP也不仅仅就是比 UDP 多了一个可靠传输，前面也说到了这只是他们之间一个重要的区别。其实他的三个重要特性就是它们之间的区别。
 
&emsp;&emsp;* 可靠传输
&emsp;&emsp;* 流量控制
&emsp;&emsp;* 拥塞控制

## 2.可靠传输
TCP 主要是`确认重传机制` `数据校验                                                                                                                                                                                                                                                      ` `数据合理分片和排序` `流量控制` `拥塞控制`依靠来完成可靠传输的 , 下面详细介绍这几种保证可靠传输的方式。

### 1. 确认和重传
确认重传，简单来说就是接收方收到报文以后给发送方一个 ACK 回复，说明自己已经收到了发送方发过来的数据。如果发送方等待了一个特定的时间还没有收到接收方的 ACK 他就认为数据包丢了，接收方没有收到就会重发这个数据包。

好的，上面的机制还是比较好理解的，但是我们会发现一个问题，那就是如果接收方已经收到了数据然后返回的 ACK 丢失，发送方就会误判导致重发。而此时接收方就会收到冗余的数据，但是接收方怎么能判定这个数据是冗余的还是新的数据呢？

这就涉及到了 TCP 的另外一个机制就是采用序号和确认号，也就是每次发送数据的时候这个报文段里面包括了当前报文段的序号和对上面的报文的确认号，这样我们的接收方可以根据自己接受缓存中已经有的数据来确定是否接受到了重复的报文段。这时候如果出现上面所说的 ACK 丢失，导致接受重复的报文段时客户端丢弃这个冗余的报文段。

好现在我们大致了解了确认重传机制，但是还有些东西还没有弄清楚，也就是 TCP 真正的实现究竟是怎样的。

1. 确认是每发一个报文段就确认一次还是一次确认多个呢？

2. 还有上面所说的发送方等待一个特定的时间，这个时间究竟等多长比较合适？
3. 重传的时候是只重传那个没收到的报文还是重传那个报文段及它以后的报文段？
#### 1.累计确认/单停等协议
这就是我们要解决的第一个问题就是如何确认。这里涉及到两种确认方式，分别称为`累计确认（捎带确认）` 和 `单停等协议` 。
###### 单停等协议
![](http://ojrw3x8j2.bkt.clouddn.com/18-3-25/9290971.jpg)

用一张图来快速理解，就是每发送一次数据，就进行一次确认。等发送方收到了 ACK 才能进行下一次的发送。
##### 累计确认
![](http://ojrw3x8j2.bkt.clouddn.com/18-3-25/8684982.jpg)

一样的也是采用的 ACK 机制，但是注意一点的是，并非对于每一个报文段都进行确认，而仅仅对最后一个报文段确认，捎带的确认了上图中的 203 号及以前的报文。
<font color=#A52A2A>
**总结：从上面可以看到累计确认的效率更加高，首先他的确认包少一些那么也就是在网络中出现的大部分是需要传输的数据，而不是一半的数据一半的 ACK ，然后我们在第二张图中可以看到我们是可以连续发送多个报文段的（究竟一次性能发多少这个取决于发送窗口，而发送窗口又是由接受窗口和拥塞窗口一起来决定的。），一次性发多个数据会提高网络的吞吐量以及效率这个可以证明，比较简单这里不再赘述！**

**结论：显然怎么看都是后者比较有优势，TCP 的实现者自然也是采用的累计确认的方式！**
</font>

#### 2. 超时时间计算
上文中的那个特定的时间就是超时时间，为什么有这个值呢? 其实在发送端发送的时候就为数据启动了一个定时器，这个定时器的初始值就是超时时间。

超时时间的计算其实有点麻烦，主要是我们很难确定一个确定的值，太长则进行了无意义的等待，太短就会导致冗余的包。TCP 的设计者们设计了一个计算超时时间的公式，这个公式概念比较多，有一点点麻烦，不过没关系我们一点点的来。

首先我们自己思考如何设计一个超时时间的计算公式，超时时间一般肯定是和数据的传输时间有关系的，他必然要大于数据的往返时间（数据在发送端接收端往返一趟所用的时间）。好，那么我们就从往返时间下手，可是又有一个问题就是往返时间并不是固定的我们有如何确定这个值呢？自然我们会想到我们可以取一小段时间的往返时间的平均值来代表这一时间点的往返时间，也就是微积分的思想！

好了我们找到了往返时间（RTT），接下来的超时时间应该就是往返时间再加上一个数就能得到超时时间了。这个数也应该是动态的，我们就选定为往返时间的波动差值，也就是相邻两个往返时间的差。

下面给出我们所预估的超时时间（TimeOut）公式：

```
TimeOut = AvgRTT2 + | AvgRTT2 - AvgRTT1 |
```

很好，看到这里其实你已经差不多理解了超时时间的计算方式了，只不过我们这个公式不够完善，但是思路是对的。我们这时候来看看 TCP 的实现者们采用的方式。

```
RTT_New = (1-a)RTT_Current + a*Avg_RTT (计算平均 RTT，a 通常取0.125)
DevRTT = (1-b)DevRTT + b|RTT_New - Avg_RTT|  （计算差值，b 通常取0.25）
TimeOut = RTT_New + 4*DevRTT （计算超时时间） 
```

好的，这就是 TCP 实现的超时时间的方式，但是在实际的应用中并不是一直采用的这种方式。假如说我们现在网络状态非常的差，一直在丢包我们根本没必要这样计算，而是采用直接把原来的超时时间加倍作为新的超时时间。

<font color=#A52A2A>
**总结：好的现在我们知道了在两种情况下的超时时间的计算方式，正常的情况下我们采用的上面的比较复杂的计算公式，也就是 `RTT+波动值` 否则直接加倍**
</font>

#### 3. 快速重传
上面我们看到在发送方等待一个超时重传时间后会开始重传，但是我们计算的超时重传时间也不定就很准，也就是说我们经常干的一件事就会是等待，而且一般等的时间还挺长。那么可不可以优化一下呢？

当然，在 TCP 实现中是做了优化的，也就是这里说到的快速重传机制。他的原理就是在发送方收到三个冗余的 ACK 的时候，就开始重传那个报文段。那么为什么是三个冗余的 ACK 呢？注意三个冗余的 ACK 其实是四个 ACK 。我们先了解一下发送 ACK 策略，这个是 `RFC 5681 文档` 规定的。

1. 第一种情况收到一个期望的有序的数据时，最多延时 500ms 发送一个 ACK 表示该数据及以前的数据都收到了。

2. 第二种情况是收到一个期望的有序的数据时，前面的有序数据等待发送 ACK 的时候立即发送一个 ACK 捎带确认前面那个数据，也就是第一个数据还在延时的时候又来一个那么久两个一起确认。
 
3. 第三种情况，收到比期望序号大的数据的时候立即发送冗余 ACK ，ACK 确认的值就是中间缺少的第一个序号的值。

4. 收到能部分填充或者完全填充中间缺少的数据的，如果这个报文是起始于缺少的数据的低端就立即发送一个 ACK。

好的，那么现在我们可以看到如果出现了三个冗余的 ACK 他只可能是发生了两次情况三，也就是发送了两个比期望值大的数据。但是注意出现情况三有两种可能，一个是丢包，另外一个是乱序到达。
比如说我们现在是数据乱序到达的，我们来看一下。

第一种乱序情况
![](http://ojrw3x8j2.bkt.clouddn.com/18-3-25/14623789.jpg)

另外一种乱序
![](http://ojrw3x8j2.bkt.clouddn.com/18-3-25/60926878.jpg)

丢包情况
![](http://ojrw3x8j2.bkt.clouddn.com/18-3-25/17661680.jpg)


<font color=#A52A2A>
**结论: 很显然我们可以看到，如果发生了乱序有可能会出现三次冗余 ACK，但是如果发现了丢包必然会有三次冗余 ACK 发生，只是 ACK 数量可能更多但是不会比三次少**
</font>

#### 4.数据重传方式
在我们发现丢包以后我们需要重传，但是我们重传的方式也有两种方式可以选择分别是 `GBN` 和 `SR` 翻译过来就是 `拉回重传` 和 `选择重传` 。好其实我们已经能从名字上面看出来他们的作用方式了，拉回重传就是哪个地方没收到那么就从那个地方及以后的数据都重新传输，这个实现起来确实很简单，就是把发送窗口和接受窗口移回去，但是同样的我们发现这个方式不实用干了很多重复的事，效率低。

那么选择重传就是你想到的谁丢了，就传谁。不存在做无用功的情况。

<font color=#A52A2A>
**结论:  TCP 实际上使用的是两者的结合，称为选择确认，也就是允许 TCP 接收方有选择的确认失序的报文段，而不是累计确认最后一个正确接受的有序报文段。也就是跳过重传那些已经正确接受的乱序报文段。**
</font>
### 2. 数据校验
![](http://ojrw3x8j2.bkt.clouddn.com/18-3-25/40689669.jpg)

&emsp;&emsp;数据校验，其实这个比较简单就是头部的一个校验，然后进行数据校验的时候计算一遍 checkSum 比对一下。
### 3. 数据合理分片和排序
&emsp;&emsp;在 UDP 中，UDP 是直接把应用层的数据往对方的端口上 “扔” ，他基本没有任何的处理。所以说他发给网络层的数据如果大于1500字节,也就是大于MTU。这个时候发送方 IP 层就需要分片。把数据报分成若干片，使每一片都小于MTU.而接收方IP层则需要进行数据报的重组。这样就会多做许多事情,而更严重的是 ，由于UDP的特性,当某一片数据传送中丢失时 ， 接收方便无法重组数据报，将导致丢弃整个UDP数据报。

&emsp;&emsp;而在 TCP 中会按MTU合理分片，也就是在 TCP 中有一个概念叫做最大报文段长度（MSS）它规定了 TCP 的报文段的最大长度，注意这个不包括 TCP 的头，也就是他的典型值就是 1460 个字节（TCP 和 IP 的头各占用了 20 字节）。并且由于 TCP 是有序号和确认号的，接收方会缓存未按序到达的数据，根据序号重新排序报文段后再交给应用层。

### 4. 流量控制
&emsp;&emsp;流量控制一般指的就是在接收方接受报文段的时候，应用层的上层程序可能在忙于做一些其他的事情，没有时间处理缓存中的数据，如果发送方在发送的时候不控制它的速度很有可能导致接受缓存溢出，导致数据丢失。

&emsp;&emsp;相对的还有一种情况是由于两台主机之间的网络比较拥塞，如果发送方还是以一个比较快的速度发送的话就可能导致大量的丢包，这个时候也需要发送方降低发送的速度。

&emsp;&emsp;虽然看起来上面的两种情况都是由于可能导致数据丢失而让发送主机降低发送速度，但是一定要把这两种情况分开，因为前者是属于`流量控制` 而后者是 `拥塞控制` ，那将是我们后面需要讨论的事情。不要把这两个概念混了。

&emsp;&emsp;其实说到流量控制我们就不得不提一下滑动窗口协议，这个是流量控制的基础。由于 TCP 连接是一个全双工的也就是在发送的时候也是可以接受的，所以在发送端和接收端同时维持了发送窗口和接收窗口。这里为了方便讨论我们就按照单方向来讨论。

&emsp;&emsp;接收方维持一个接受窗口，发送方一个发送窗口。发送的时候要知道接受窗口还有多少空间，也就是发送的数据量不能超过接受窗口的大小，否则就溢出了。而当我们收到一个接收方的 ACK 的时候我们就可以移动接受窗口把那些已经确认的数据滑动到窗口之外，发送窗口同理把确认的移出去。这样一直维持两个窗口大小，当接收方不能在接受数据的时候就把自己的窗口大小调整为 0 发送窗口就不会发送数据了。但是有一个问题，这个时候当接收窗口再调大的时候他不会主动通知发送方，这里采用的是发送方主动询问。

&emsp;&emsp;还是画个图看的比较直观：
![](http://ojrw3x8j2.bkt.clouddn.com/18-3-25/32650197.jpg)

### 5. 拥塞控制
&emsp;&emsp;拥塞控制一般都是由于网络中的主机发送的数据太多导致的拥塞，一般拥塞的都是一些负载比较高的路由，这时候为了获得更好的数据传输稳定性，我们必须采用拥塞控制，当然也为了减轻路由的负载防止崩溃。

&emsp;&emsp;这里主要介绍两个拥塞控制的方法，一个是慢开始，另外一个称为快恢复。

#### 1.慢开始
1. 一开始我们不知道网络中的拥塞情况，我们就发一个数据包
2. 如果没有发生拥塞我们成倍的增加发送的数据的数量。
3. 当然我们也不能到无休止的增加，这里有一个慢开始门限，到达门限则加法增加，每次加一。
4. 这时候如果遇到了拥塞，我们直接跳到第一步，也就是从头开始，并且把慢开始门限调整为拥塞时候的数据量的一半再次开始。

#### 2.快恢复
1. 一开始我们不知道网络中的拥塞情况，我们就发一个数据包
2. 如果没有发生拥塞我们成倍的增加发送的数据的数量。
3. 当然我们也不能到无休止的增加，这里有一个慢开始门限，到达门限则加法增加，每次加一。
4. 这时候如果遇到了拥塞，这里就是唯一和慢开始不一样的地方，直接从新的慢开始门限加法增长。

![](http://ojrw3x8j2.bkt.clouddn.com/18-3-25/14662652.jpg)


## 3.连接管理
### 1. 建立连接3次握手
1. 客户端像服务端发起连接，首先向服务端发送一个特殊的报文，这个报文的 SYN 位被置 1 ，然后生成一个随机的序号填入到 TCP 的头部。这个报文段称为 SYN 报文，用于请求连接。

2. 服务器接收到客户端的 SYN 报文以后，也要生成一个特殊的报文段来允许客户端的接入，这个报文是设置一个自己的初始序号，SYN 设置为 1，ACK 设置为 SYN 报文序号加一。这个报文段称之为 SYNACK 报文。并且根据 SYN 报文的参数来分配本地变量，但是也是由于这么早的分配变量就有一种 SYN 洪泛攻击。注意一下，上面的这两个报文段都没有数据部分。

3. 在客户端收到 SYNACK 报文段时候需要对客户端分配变量，然后对服务器的允许进行确认。这时候 SYN 位要置位 0 ，并且可以携带数据，也就是这时候是已经开始了数据传输的。

![](http://ojrw3x8j2.bkt.clouddn.com/18-3-25/3147427.jpg)


那么问题来了，为什么需要序号呢？为什么又是三次握手而不是两次？以及什么是 SYN 洪泛攻击？

1. 序号存在的目的是为了能否区分多个 TCP 连接，毕竟是一个服务器，多个客户端，不然各个 TCP 连接就会变得非常混乱。

2. 其实我们单方向来看其实就是两次握手，之所以是三次握手是因为 TCP 是双工的，中间那次的 SYNACK 其实试一次合并。

3. SYN 洪泛攻击就是让客户端乱遭一些 IP 然后和服务器简历 TCP 连接，由于服务器收不到 ACK 但是他分配了变量，导致一直在消耗服务器资源。这个解决方法就是采用 SYNCookie 这个 Cookie 其实就是服务器在发送 SYNACK 的头部的 seq 序号的值，那么客户端必须返回一个比 Cookie 大一的 ACK 回来才是正确的，否则不分配变量，也就是变量延时分配。


### 2.释放链接四次挥手
1. 首先客户端发起终止会话的请求，FIN=1
2. 服务器接收到后相应客户端 ACK=1
3. 服务器发送完毕终止会话 FIN=1
4. 客户端回应 ACK=1

![](http://ojrw3x8j2.bkt.clouddn.com/18-3-25/59663362.jpg)

这里需要说明一下的是最后的那个长长的 TIME_WAIT 状态一般是为了客户端能够发出 ACK 一般他的值是 1分钟 或者2分钟


## 4.总结
&emsp;&emsp;好了，今天真的写了不少，主要就是把 TCP 的可靠传输以及连接管理讲清楚了，以及里面的一下细节问题，真的很花时间。然后其他没有涉及到的就是关于 TCP 的头并没有详细的去分析，这个东西其实也不是很难，但是现在篇幅真的已经很大就先这样，头里面的都是固定的不需要太多的理解。

