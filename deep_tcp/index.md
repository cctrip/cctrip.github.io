# 理解TCP协议


## 1. 介绍

TCP协议是个复杂的东西，但是对学习网络知识也好，排查网络问题也好，优化网络性能也是，又是一个不得不啃的东西。自己也反反复复的看了不少文章，书籍和RFC文档，这次，想总结一下自己这些年对TCP的理解，方便后续查阅。

### 1.1 TCP是什么？

我们知道，IP协议是不可靠的，它不保证IP数据报文能成功达到目的地，它只提供最好的传输服务。而TCP的出现就是为了在IP协议不可靠的传输上面提供可靠的传输服务。

> [wiki](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)上的定义，TCP是在通过IP网络进行通信的主机上运行的应用程序之间，提供可靠，有序且经过错误检查的八位位组（字节）流交付。

在[RFC 793](https://tools.ietf.org/html/rfc793)中，定义了为了实现TCP服务，需要做什么：

* **Basic data transfer**

  *TCP*通过将一定量的字节打包成在*internet*系统上传输的分片，能够在用户之间在两个方向上传输连续的字节流。通常情况下，*TCPs*决定什么时候阻塞以及前推数据。有时候用户需要确保他们所提交给*TCP*的所有数据都被传输。基于这个目的，定义了*push*功能。为了确认提交给*TCP*的数据报确实被传送了，发送用户指示数据必须被推给接收用户。*Push*导致了*TCPs*立即前推和投递数据给接收者。确切的*push*点对接收用户可能不可见，且*push*功能不提供一个记录边界标识。

* **Reliability**

  为确保可靠性，TCP必须能解决由通讯网络造成的数据损坏、丢失、重复、或者乱序问题。TCP通过给每个字节分配序列号，并要求接收者必须确认(ACK)的机制来达到这个目的。(确认机制)，如果在超时时间间隔内没有收到ACK，将重传该”数据“。（重传机制）。对于接收方而言，序列号用来对可能接收到的乱序的、重复的数据段进行正确的排序并且消除重复数据。对于损坏的数据而言，通过在每个传输的数据段中添加checksum，并且在接收端进行检查来进行丢弃处理。

* **Flow control**

  TCP为接受方提供了一种管理发送方发送数据量大小的机制。通过在每次返回确认信息（ACK）的时候增加一个窗口，这个窗口表示接收方最后一次成功接收之后还可以接收的字节数量。

* **Multiplexing**

  为了允许在一个单独的主机里多个进程同时使用*TCP*通信机制，*TCP*提供了一套地址和端口。从*internet*通信层同网络和宿主地址连接，这形成了一个*socket*。一对*socket*标识了一个连接。也就是说，一个*socket*可能同时被使用在多个连接中。绑定端口到进程被每个主机单独处理。是，将常用的进程（如“*logger*”或者时间服务）隶属于众所皆知的*socket*被证明是有用的。这些服务就可以通过已知的地址获取到。建立和学习其它进程的端口地址可能包括更加动态的机制。

* **Connections**

  上面描述的可靠性和流量控制机制要求所有的TCP为每个数据流发起和维护某些状态信息。这些信息的结合体，包括*sockets*，系列号，和窗口大小，被称为一个连接。每个连接被一套指定两端的socket唯一指定。当两个进程需要通信的时候，他们的*TCPs*必须首先建立一个连接（在每一端初始化状态信息）。当通信完成的时候，连接终止或者关闭以释放资源用于其它用途。由于连接必须在不可靠的主机和不可靠的*internet*通信系统上建立，一个带有基于时钟的系列号的握手机制被用来避免连接的错误初始化。

* **Precedence and Security**

  *TCP*用户可以指示通信的安全性和优先级。当这些特性不需要的时候，规定采用缺省值。

### 1.2 数据封装

![](encapsulation.png)

首先，我们要知道，真实数据是通过层层封装后传输出去的，每一层都会添加自己的协议信息到header里面加上数据形成新的数据"包"，方便对方解析。接下来，我们来了解TCP的相关数据信息。

***

## 2. TCP基本信息

### 2.1 Header

![](tcp-header.png)

在TCP传输中，是以段(segment)为一个单位进行传输的，一个TCP段由header和data部分组成，header由10个固定字段(共20字节)和一个可选扩展字段组成：

- **Source port (16 bits)**

  发送端口标识	.

- **Destination port (16 bits)**

  接收端口标识

- **Sequence number (32 bits)**

  两种作用：

  * 当SYN标志位为1时，这是一个初始序列号，实际第一个数据字节的序列号和相应ACK中的已确认数字就是该序列号加1。
  * 当SYN标志位为0时，这是当前会话中该段的第一个数据字节的累积序列号。

- **Acknowledgment number (32 bits)**

  如果设置了ACK标志，则此字段的值是ACK发送方期望的下一个序列号。这确认接收到所有先前的字节（如果有）。两端发送的第一个ACK会确认另一端的初始序列号本身，但不会确认任何数据。

- **Data offset (4 bits)**

  指定TCP头的大小（以32 bit为单位） 

- **Reserved (3 bits)**

  预留位，设置为0

- **Flags (9 bits)**

  * NS (1 bit): ECN-nonce 隐藏保护
  * CWR (1 bit): 发送主机设置了减少拥塞窗口（CWR）标志，以指示它接收到设置了ECE标志的TCP段，并且已经在拥塞控制机制中做出了响应。
  * ECE (1 bit): ECN-Echo具有双重作用，具体取决于SYN标志的值:
    * 如果将SYN标志设置为（1），则表明TCP对等方具有ECN功能。
    * 如果清除了SYN标志（0），则表示在正常传输期间收到了TCP发送者在IP报头中设置了拥塞经历标志（ECN = 11）的数据包。

  - URG (1 bit):表示紧急指针字段有效
  - ACK (1 bit): 表示“确认”字段是有效的。客户端发送的初始SYN数据包之后的所有数据包都应设置此标志。
  - PSH (1 bit): 推送功能。要求将缓冲的数据推送到接收的应用程序。
  - RST (1 bit): 重置连接
  - SYN (1 bit): 同步序列号。仅从两端发送的第一个数据包应设置此标志。其他一些标志和字段会根据此标志更改含义，一些仅在设置时有效，而其他一些则在清除时有效。
  - FIN (1 bit): 发送者的最后一个数据包，表示开始关闭连接。

- **Window size (16 bits)**

  接收窗口的大小，它指定此段的发送方当前愿意接收的窗口大小单位。

- **Checksum (16 bits)**

  16位校验和字段用于TCP报头，有效负载和IP伪报头的错误校验。伪标头由源IP地址，目标IP地址，TCP协议的协议号（6）以及TCP标头和有效载荷的长度（以字节为单位）组成。

- **Urgent pointer (16 bits)**

  如果设置了URG标志，则此16位字段是指示最后一个紧急数据字节的序列号的偏移量。

- **Options (Variable 0–320 bits, in units of 32 bits)**

  可选字段

- **Padding**

  TCP header padding用于确保TCP标头在32位边界上结束并开始数据。填充由零组成。

***

### 2.2 状态机

![](tcpfsm.png)

状态机是一个很重要的东西，它能让我们了解到一个TCP传输过程处于哪个阶段，可能出现了什么问题。

#### 2.2.1 三次握手

![](3way.png)

一个TCP连接的开始由三次握手组成：

1. 三次握手的前提是Server要进入`LISTEN`状态
2. client往server发起请求，将自己TCP header中的SYN标志位设置为1，seq num为 4567，发送过后，自己进入`SYN-SENT`状态
3. server接收到SYN包后，给client回复包，将自己TCP header中的ack num设为4567+1，seq num为12998，并将SYN和ACK标志位设为1，发送给client，自己进入`SYN-RECEIVED`状态
4. client收到ack包后，将自己的ack num设为12998+1，并将ACK标志位设为1，发送给server，此时进去`ESTABLISHED`状态
5. server收到ack后，自己也进入了`ESTABLISHED`状态。

#### 2.2.2 数据传输

数据传输过程中，client和server两端的状态都为`ESTABLISHED`

这里的传输会涉及到流控，影响着数据传输的效率。

#### 2.2.3 四次挥手

![](close.png)

当所有数据传输结束后，就需要结束一个TCP连接了。

注：client表示发起结束请求的一方，真实情况下可能会是客户端发起，也可能是服务端发起。

1. client将自己的TCP header中的FIN标志位设置为1，往server发起结束连接的请求，自己进入`FIN-WAIT-1`状态
2. server端接收到FIN的请求后，返回一个ack包，然后等待应用的关闭连接，此时自己进入`CLOSE-WAIT`状态
3. client接收到回复的ack后，进入`FIN-WAIT-2`状态，等待server端的FIN包
4. 应用允许关闭连接后，server端往client发送一个FIN包，此时自己进入了`LAST-ACK`状态，等待最终的ack包
5. client接收到fin包后，回复一个ack，自己进入了`TIME_WAIT`状态，默认情况下等待2个MSL后关闭socket
6. server端接收到ack后，这条连接就完全关闭了。

***

## 3. TCP细节

上面所说的数据传输是基于完美状态下，但我们知道，IP传输是不可靠的，因此会出现各种问题，而TCP需要针对这些问题进行处理。

### 3.1 TCP重传和确认

因为IP传输是不可靠的，那么就有可能产生丢包或者损坏，所以TCP要保证所有的包都可以到达，因此必须要有重传机制。但是要怎么保证重传是高效的呢？

注意，接收端给发送端的Ack确认只会确认最后一个连续的包，比如，发送端发了1,2,3,4,5一共五份数据，接收端收到了1，2，于是回ack 3，然后收到了4（注意此时3没收到），此时的TCP会怎么办？我们要知道，因为正如前面所说的，**SeqNum和Ack是以字节数为单位，所以ack的时候，不能跳着确认，只能确认最大的连续收到的包**，不然，发送端就以为之前的都收到了。

#### 3.1.1 超时重传

一种是不回ack，死等3，当发送方发现收不到3的ack超时后，会重传3。一旦接收方收到3后，会ack 回 4——意味着3和4都收到了。

但是，这种方式会有比较严重的问题，那就是因为要死等3，所以会导致4和5即便已经收到了，而发送方也完全不知道发生了什么事，因为没有收到Ack，所以，发送方可能会悲观地认为也丢了，所以有可能也会导致4和5的重传。

对此有两种选择：

- 一种是仅重传timeout的包。也就是第3份数据。
- 另一种是重传timeout后所有的数据，也就是第3，4，5这三份数据。

这两种方式有好也有不好。第一种会节省带宽，但是慢，第二种会快一点，但是会浪费带宽，也可能会有无用功。但总体来说都不好。因为都在等timeout，timeout可能会很长。

**timeout计算**

Timeout的设置对于重传非常重要。

- 设长了，重发就慢，丢了老半天才重发，没有效率，性能差；
- 设短了，会导致可能并没有丢就重发。于是重发的就快，会增加网络拥塞，导致更多的超时，更多的超时导致更多的重发。

而且，这个超时时间在不同的网络的情况下，根本没有办法设置一个死的值。只能动态地设置。 为了动态地设置，TCP引入了RTT——Round Trip Time，也就是一个数据包从发出去到回来的时间。这样发送端就大约知道需要多少的时间，从而可以方便地设置Timeout——RTO（Retransmission TimeOut），以让我们的重传机制更高效。 听起来似乎很简单，好像就是在发送端发包时记下t0，然后接收端再把这个ack回来时再记一个t1，于是RTT = t1 – t0。没那么简单，这只是一个采样，不能代表普遍情况。

当前最新的timetout计算算法叫Jacobson / Karels Algorithm（参看[RFC6289](http://tools.ietf.org/html/rfc6298)）。这个算法引入了最新的RTT的采样和平滑过的SRTT的差距做因子来计算。 公式如下：（其中的DevRTT是Deviation RTT的意思）

```mathematica
SRTT = SRTT + α* (RTT – SRTT)          —— 计算平滑RTT

DevRTT = (1-β)*DevRTT + β*(|RTT-SRTT|) ——计算平滑RTT和真实的差距（加权移动平均）

RTO= µ \* SRTT + ∂ \*DevRTT            —— 神一样的公式
```

（其中：在Linux下，α = 0.125，β = 0.25， μ = 1，∂ = 4 ——这就是算法中的“调得一手好参数”，nobody knows why, it just works…） 最后的这个算法在被用在今天的TCP协议中（Linux的源代码在：[tcp_rtt_estimator](http://lxr.free-electrons.com/source/net/ipv4/tcp_input.c?v=2.6.32#L609)）。

***

#### 3.1.2 快速重传机制

上面说了超时重传总体来说并不是一个号办法，于是，TCP引入了一种叫**Fast Retransmit** 的算法，**不以时间驱动，而以数据驱动重传**。也就是说，如果，包没有连续到达，就ack最后那个可能被丢了的包，如果发送方连续收到3次相同的ack，就重传。Fast Retransmit的好处是不用等timeout了再重传。

比如：如果发送方发出了1，2，3，4，5份数据，第一份先到送了，于是就ack回2，结果2因为某些原因没收到，3到达了，于是还是ack回2，后面的4和5都到了，但是还是ack回2，因为2还是没有收到，于是发送端收到了三个ack=2的确认，知道了2还没有到，于是就马上重转2。然后，接收端收到了2，此时因为3，4，5都收到了，于是ack回6。示意图如下：

![img](FASTIncast021.png)

Fast Retransmit只解决了一个问题，就是timeout的问题，它依然面临一个艰难的选择，就是，是重传之前的一个还是重传所有的问题。对于上面的示例来说，是重传#2呢还是重传#2，#3，#4，#5呢？因为发送端并不清楚这连续的3个ack(2)是谁传回来的？也许发送端发了20份数据，是#6，#10，#20传来的呢。这样，发送端很有可能要重传从2到20的这堆数据（这就是某些TCP的实际的实现）。可见，这是一把双刃剑。

***

#### 3.1.3 SACK 方法

另外一种更好的方式叫：**Selective Acknowledgment (SACK)**（参看[RFC 2018](http://tools.ietf.org/html/rfc2018)），这种方式需要在TCP头里加一个SACK的东西，ACK还是Fast Retransmit的ACK，SACK则是汇报收到的数据碎版。参看下图：

![img](tcp_sack.jpg)

这样，在发送端就可以根据回传的SACK来知道哪些数据到了，哪些没有到。于是就优化了Fast Retransmit的算法。当然，这个协议需要两边都支持。在 Linux下，可以通过**tcp_sack**参数打开这个功能（Linux 2.4后默认打开）。

这里还需要注意一个问题——**接收方Reneging，所谓Reneging的意思就是接收方有权把已经报给发送端SACK里的数据给丢了**。这样干是不被鼓励的，因为这个事会把问题复杂化了，但是，接收方这么做可能会有些极端情况，比如要把内存给别的更重要的东西。**所以，发送方也不能完全依赖SACK，还是要依赖ACK，并维护Time-Out，如果后续的ACK没有增长，那么还是要把SACK的东西重传，另外，接收端这边永远不能把SACK的包标记为Ack。**

注意：SACK会消费发送方的资源，试想，如果一个攻击者给数据发送方发一堆SACK的选项，这会导致发送方开始要重传甚至遍历已经发出的数据，这会消耗很多发送端的资源。详细的东西请参看《[TCP SACK的性能权衡](http://www.ibm.com/developerworks/cn/linux/l-tcp-sack/)》

***

#### 3.1.4 Duplicate SACK – 重复收到数据的问题

Duplicate SACK又称D-SACK，**其主要使用了SACK来告诉发送方有哪些数据被重复接收了**。[RFC-2883 ](http://www.ietf.org/rfc/rfc2883.txt)里有详细描述和示例。下面举几个例子（来源于[RFC-2883](http://www.ietf.org/rfc/rfc2883.txt)）

D-SACK使用了SACK的第一个段来做标志，

- 如果SACK的第一个段的范围被ACK所覆盖，那么就是D-SACK

- 如果SACK的第一个段的范围被SACK的第二个段覆盖，那么就是D-SACK

**示例一：ACK丢包**

下面的示例中，丢了两个ACK，所以，发送端重传了第一个数据包（3000-3499），于是接收端发现重复收到，于是回了一个SACK=3000-3500，因为ACK都到了4000意味着收到了4000之前的所有数据，所以这个SACK就是D-SACK——旨在告诉发送端我收到了重复的数据，而且我们的发送端还知道，数据包没有丢，丢的是ACK包。

```shell
Transmitted  Received    ACK Sent

Segment      Segment     (Including SACK Blocks)

3000-3499    3000-3499   3500 (ACK dropped)

3500-3999    3500-3999   4000 (ACK dropped)

3000-3499    3000-3499   4000, SACK=3000-3500
```

 **示例二，网络延误**

下面的示例中，网络包（1000-1499）被网络给延误了，导致发送方没有收到ACK，而后面到达的三个包触发了“Fast Retransmit算法”，所以重传，但重传时，被延误的包又到了，所以，回了一个SACK=1000-1500，因为ACK已到了3000，所以，这个SACK是D-SACK——标识收到了重复的包。

这个案例下，发送端知道之前因为“Fast Retransmit算法”触发的重传不是因为发出去的包丢了，也不是因为回应的ACK包丢了，而是因为网络延时了。

```shell
Transmitted    Received    ACK Sent

Segment        Segment     (Including SACK Blocks)

500-999        500-999     1000

1000-1499      (delayed)

1500-1999      1500-1999   1000, SACK=1500-2000

2000-2499      2000-2499   1000, SACK=1500-2500

2500-2999      2500-2999   1000, SACK=1500-3000

1000-1499      1000-1499   3000

1000-1499   3000, SACK=1000-1500
```

可见，引入了D-SACK，有这么几个好处：

1）可以让发送方知道，是发出去的包丢了，还是回来的ACK包丢了。

2）是不是自己的timeout太小了，导致重传。

3）网络上出现了先发的包后到的情况（又称reordering）

4）网络上是不是把我的数据包给复制了。

 **知道这些东西可以很好得帮助TCP了解网络情况，从而可以更好的做网络上的流控**。

Linux下的tcp_dsack参数用于开启这个功能（Linux 2.4后默认打开）

***

### 3.2 滑动窗口

TCP必需要知道网络实际的数据处理带宽或是数据处理速度，这样才不会引起网络拥塞，导致丢包。

所以，TCP引入了一些技术和设计来做网络流控，Sliding Window是其中一个技术。 前面我们说过，**TCP头里有一个字段叫Window，又叫Advertised-Window，这个字段是接收端告诉发送端自己还有多少缓冲区可以接收数据**。**于是发送端就可以根据这个接收端的处理能力来发送数据，而不会导致接收端处理不过来**。 为了说明滑动窗口，我们需要先看一下TCP缓冲区的一些数据结构：

![img](sliding_window.jpg)

上图中，我们可以看到：

- 接收端LastByteRead指向了TCP缓冲区中读到的位置，NextByteExpected指向的地方是收到的连续包的最后一个位置，LastByteRcved指向的是收到的包的最后一个位置，我们可以看到中间有些数据还没有到达，所以有数据空白区。

- 发送端的LastByteAcked指向了被接收端Ack过的位置（表示成功发送确认），LastByteSent表示发出去了，但还没有收到成功确认的Ack，LastByteWritten指向的是上层应用正在写的地方。

于是：

- 接收端在给发送端回ACK中会汇报自己的AdvertisedWindow = MaxRcvBuffer – LastByteRcvd – 1;

- 而发送方会根据这个窗口来控制发送数据的大小，以保证接收方可以处理。

下面我们来看一下发送方的滑动窗口示意图：

![img](tcpswwindows.png)

上图中分成了四个部分，分别是：（其中那个黑模型就是滑动窗口）

- \#1已收到ack确认的数据。
- \#2发还没收到ack的。
- \#3在窗口中还没有发出的（接收方还有空间）。
- \#4窗口以外的数据（接收方没空间）

下面是个滑动后的示意图（收到36的ack，并发出了46-51的字节）：

![img](tcpswslide.png)

下面我们来看一个接受端控制发送端的图示：

![img](tcpswflow.png)

##### Zero Window

上图，我们可以看到一个处理缓慢的Server（接收端）是怎么把Client（发送端）的TCP Sliding Window给降成0的。此时，你一定会问，如果Window变成0了，TCP会怎么样？是不是发送端就不发数据了？是的，发送端就不发数据了，你可以想像成“Window Closed”，那你一定还会问，如果发送端不发数据了，接收方一会儿Window size 可用了，怎么通知发送端呢？

解决这个问题，TCP使用了Zero Window Probe技术，缩写为ZWP，也就是说，发送端在窗口变成0后，会发ZWP的包给接收方，让接收方来ack他的Window尺寸，一般这个值会设置成3次，第次大约30-60秒（不同的实现可能会不一样）。如果3次过后还是0的话，有的TCP实现就会发RST把链接断了。

**注意**：只要有等待的地方都可能出现DDoS攻击，Zero Window也不例外，一些攻击者会在和HTTP建好链发完GET请求后，就把Window设置为0，然后服务端就只能等待进行ZWP，于是攻击者会并发大量的这样的请求，把服务器端的资源耗尽。（关于这方面的攻击，大家可以移步看一下[Wikipedia的SockStress词条](http://en.wikipedia.org/wiki/Sockstress)）

另外，Wireshark中，你可以使用tcp.analysis.zero_window来过滤包，然后使用右键菜单里的follow TCP stream，你可以看到ZeroWindowProbe及ZeroWindowProbeAck的包。

##### Silly Window Syndrome

Silly Window Syndrome翻译成中文就是“糊涂窗口综合症”。正如你上面看到的一样，如果我们的接收方太忙了，来不及取走Receive Windows里的数据，那么，就会导致发送方越来越小。到最后，如果接收方腾出几个字节并告诉发送方现在有几个字节的window，而我们的发送方会义无反顾地发送这几个字节。

要知道，我们的TCP+IP头有40个字节，为了几个字节，要达上这么大的开销，这太不经济了。

另外，你需要知道网络上有个MTU，对于以太网来说，MTU是1500字节，除去TCP+IP头的40个字节，真正的数据传输可以有1460，这就是所谓的MSS（Max Segment Size）注意，TCP的RFC定义这个MSS的默认值是536，这是因为 [RFC 791](http://tools.ietf.org/html/rfc791)里说了任何一个IP设备都得最少接收576尺寸的大小（实际上来说576是拨号的网络的MTU，而576减去IP头的20个字节就是536）。

**如果你的网络包可以塞满MTU，那么你可以用满整个带宽，如果不能，那么你就会浪费带宽**。（大于MTU的包有两种结局，一种是直接被丢了，另一种是会被重新分块打包发送） 你可以想像成一个MTU就相当于一个飞机的最多可以装的人，如果这飞机里满载的话，带宽最高，如果一个飞机只运一个人的话，无疑成本增加了，也而相当二。

所以，**Silly Windows Syndrome这个现像就像是你本来可以坐200人的飞机里只做了一两个人**。 要解决这个问题也不难，就是避免对小的window size做出响应，直到有足够大的window size再响应，这个思路可以同时实现在sender和receiver两端。

- 如果这个问题是由Receiver端引起的，那么就会使用 David D Clark’s 方案。在receiver端，如果收到的数据导致window size小于某个值，可以直接ack(0)回sender，这样就把window给关闭了，也阻止了sender再发数据过来，等到receiver端处理了一些数据后windows size 大于等于了MSS，或者，receiver buffer有一半为空，就可以把window打开让send 发送数据过来。

- 如果这个问题是由Sender端引起的，那么就会使用著名的 [Nagle’s algorithm](http://en.wikipedia.org/wiki/Nagle's_algorithm)。这个算法的思路也是延时处理，他有两个主要的条件：1）要等到 Window Size>=MSS 或是 Data Size >=MSS，2）收到之前发送数据的ack回包，他才会发数据，否则就是在攒数据。

另外，Nagle算法默认是打开的，所以，对于一些需要小包场景的程序——**比如像telnet或ssh这样的交互性比较强的程序，你需要关闭这个算法**。你可以在Socket设置TCP_NODELAY选项来关闭这个算法（关闭Nagle算法没有全局参数，需要根据每个应用自己的特点来关闭）

```c
setsockopt(sock_fd, IPPROTO_TCP, TCP_NODELAY, (**char** *)&value,sizeof(**int**));
```

另外，网上有些文章说TCP_CORK的socket option是也关闭Nagle算法，这不对。**TCP_CORK其实是更新激进的Nagle算汉，完全禁止小包发送，而Nagle算法没有禁止小包发送，只是禁止了大量的小包发送**。最好不要两个选项都设置。

***

### 3.3 拥塞处理

上面我们知道了，TCP通过Sliding Window来做流控（Flow Control），但是TCP觉得这还不够，因为Sliding Window需要依赖于连接的发送端和接收端，其并不知道网络中间发生了什么。TCP的设计者觉得，一个伟大而牛逼的协议仅仅做到流控并不够，因为流控只是网络模型4层以上的事，TCP的还应该更聪明地知道整个网络上的事。

具体一点，我们知道TCP通过一个timer采样了RTT并计算RTO，但是，**如果网络上的延时突然增加，那么，TCP对这个事做出的应对只有重传数据，但是，重传会导致网络的负担更重，于是会导致更大的延迟以及更多的丢包，于是，这个情况就会进入恶性循环被不断地放大。试想一下，如果一个网络内有成千上万的TCP连接都这么行事，那么马上就会形成“网络风暴”，TCP这个协议就会拖垮整个网络。**这是一个灾难。

所以，TCP不能忽略网络上发生的事情，而无脑地一个劲地重发数据，对网络造成更大的伤害。对此TCP的设计理念是：**TCP不是一个自私的协议，当拥塞发生的时候，要做自我牺牲。就像交通阻塞一样，每个车都应该把路让出来，而不要再去抢路了。**

关于拥塞控制的论文请参看《[Congestion Avoidance and Control](http://ee.lbl.gov/papers/congavoid.pdf)》(PDF)

拥塞控制主要是四个算法：**1）慢启动**，**2）拥塞避免**，**3）拥塞发生**，**4）快速恢复**。

#### 3.3.1 慢热启动算法 – Slow Start

首先，我们来看一下TCP的慢热启动。慢启动的意思是，刚刚加入网络的连接，一点一点地提速，不要一上来就像那些特权车一样霸道地把路占满。新同学上高速还是要慢一点，不要把已经在高速上的秩序给搞乱了。

慢启动的算法如下(cwnd全称Congestion Window)：

1）连接建好的开始先初始化cwnd = 1，表明可以传一个MSS大小的数据。

2）每当收到一个ACK，cwnd++; 呈线性上升

3）每当过了一个RTT，cwnd = cwnd*2; 呈指数让升

4）还有一个ssthresh（slow start threshold），是一个上限，当cwnd >= ssthresh时，就会进入“拥塞避免算法”（后面会说这个算法）

所以，我们可以看到，如果网速很快的话，ACK也会返回得快，RTT也会短，那么，这个慢启动就一点也不慢。下图说明了这个过程。

![img](tcp_slow_start.jpg)

这里，我需要提一下的是一篇Google的论文《[An Argument for Increasing TCP’s Initial Congestion Window](http://static.googleusercontent.com/media/research.google.com/zh-CN//pubs/archive/36640.pdf)》Linux 3.0后采用了这篇论文的建议——把cwnd 初始化成了 10个MSS。 而Linux 3.0以前，比如2.6，Linux采用了[RFC3390](http://www.rfc-editor.org/rfc/rfc3390.txt)，cwnd是跟MSS的值来变的，如果MSS< 1095，则cwnd = 4；如果MSS>2190，则cwnd=2；其它情况下，则是3。

#### 3.3.2 拥塞避免算法 – Congestion Avoidance

前面说过，还有一个ssthresh（slow start threshold），是一个上限，当cwnd >= ssthresh时，就会进入“拥塞避免算法”。一般来说ssthresh的值是65535，单位是字节，当cwnd达到这个值时后，算法如下：

1）收到一个ACK时，cwnd = cwnd + 1/cwnd

2）当每过一个RTT时，cwnd = cwnd + 1

这样就可以避免增长过快导致网络拥塞，慢慢的增加调整到网络的最佳值。很明显，是一个线性上升的算法。

#### 3.3.3 拥塞状态时的算法

前面我们说过，当丢包的时候，会有两种情况：

1）等到RTO超时，重传数据包。TCP认为这种情况太糟糕，反应也很强烈。

- - sshthresh =  cwnd /2
  - cwnd 重置为 1
  - 进入慢启动过程

2）Fast Retransmit算法，也就是在收到3个duplicate ACK时就开启重传，而不用等到RTO超时。

- - TCP Tahoe的实现和RTO超时一样。

- - TCP Reno的实现是：
    - cwnd = cwnd /2
    - sshthresh = cwnd
    - 进入快速恢复算法——Fast Recovery

上面我们可以看到RTO超时后，sshthresh会变成cwnd的一半，这意味着，如果cwnd<=sshthresh时出现的丢包，那么TCP的sshthresh就会减了一半，然后等cwnd又很快地以指数级增涨爬到这个地方时，就会成慢慢的线性增涨。我们可以看到，TCP是怎么通过这种强烈地震荡快速而小心得找到网站流量的平衡点的。

#### 3.3.4 快速恢复算法 – Fast Recovery

**TCP Reno**

这个算法定义在[RFC5681](http://tools.ietf.org/html/rfc5681)。快速重传和快速恢复算法一般同时使用。快速恢复算法是认为，你还有3个Duplicated Acks说明网络也不那么糟糕，所以没有必要像RTO超时那么强烈。 注意，正如前面所说，进入Fast Recovery之前，cwnd 和 sshthresh已被更新：

- cwnd = cwnd /2
- sshthresh = cwnd

然后，真正的Fast Recovery算法如下：

- cwnd = sshthresh  + 3 * MSS （3的意思是确认有3个数据包被收到了）
- 重传Duplicated ACKs指定的数据包
- 如果再收到 duplicated Acks，那么cwnd = cwnd +1
- 如果收到了新的Ack，那么，cwnd = sshthresh ，然后就进入了拥塞避免的算法了。

如果你仔细思考一下上面的这个算法，你就会知道，**上面这个算法也有问题，那就是——它依赖于3个重复的Acks**。注意，3个重复的Acks并不代表只丢了一个数据包，很有可能是丢了好多包。但这个算法只会重传一个，而剩下的那些包只能等到RTO超时，于是，进入了恶梦模式——超时一个窗口就减半一下，多个超时会超成TCP的传输速度呈级数下降，而且也不会触发Fast Recovery算法了。

通常来说，正如我们前面所说的，SACK或D-SACK的方法可以让Fast Recovery或Sender在做决定时更聪明一些，但是并不是所有的TCP的实现都支持SACK（SACK需要两端都支持），所以，需要一个没有SACK的解决方案。而通过SACK进行拥塞控制的算法是FACK（后面会讲）

**TCP New Reno**

于是，1995年，TCP New Reno（参见 [RFC 6582](http://tools.ietf.org/html/rfc6582) ）算法提出来，主要就是在没有SACK的支持下改进Fast Recovery算法的——

- 当sender这边收到了3个Duplicated Acks，进入Fast Retransimit模式，开发重传重复Acks指示的那个包。如果只有这一个包丢了，那么，重传这个包后回来的Ack会把整个已经被sender传输出去的数据ack回来。如果没有的话，说明有多个包丢了。我们叫这个ACK为Partial ACK。

- 一旦Sender这边发现了Partial ACK出现，那么，sender就可以推理出来有多个包被丢了，于是乎继续重传sliding window里未被ack的第一个包。直到再也收不到了Partial Ack，才真正结束Fast Recovery这个过程

我们可以看到，这个“Fast Recovery的变更”是一个非常激进的玩法，他同时延长了Fast Retransmit和Fast Recovery的过程。

##### 算法示意图

下面我们来看一个简单的图示以同时看一下上面的各种算法的样子：

![img](https://coolshell.cn/wp-content/uploads/2014/05/tcp.fr_-1024x359.jpg)

 

##### FACK算法

FACK全称Forward Acknowledgment 算法，论文地址在这里（PDF）[Forward Acknowledgement: Refining TCP Congestion Control](http://conferences.sigcomm.org/sigcomm/1996/papers/mathis.pdf) 这个算法是其于SACK的，前面我们说过SACK是使用了TCP扩展字段Ack了有哪些数据收到，哪些数据没有收到，他比Fast Retransmit的3 个duplicated acks好处在于，前者只知道有包丢了，不知道是一个还是多个，而SACK可以准确的知道有哪些包丢了。 所以，SACK可以让发送端这边在重传过程中，把那些丢掉的包重传，而不是一个一个的传，但这样的一来，如果重传的包数据比较多的话，又会导致本来就很忙的网络就更忙了。所以，FACK用来做重传过程中的拥塞流控。

- 这个算法会把SACK中最大的Sequence Number 保存在**snd.fack**这个变量中，snd.fack的更新由ack带秋，如果网络一切安好则和snd.una一样（snd.una就是还没有收到ack的地方，也就是前面sliding window里的category #2的第一个地方）

- 然后定义一个**awnd = snd.nxt – snd.fack**（snd.nxt指向发送端sliding window中正在要被发送的地方——前面sliding windows图示的category#3第一个位置），这样awnd的意思就是在网络上的数据。（所谓awnd意为：actual quantity of data outstanding in the network）

- 如果需要重传数据，那么，**awnd = snd.nxt – snd.fack + retran_data**，也就是说，awnd是传出去的数据 + 重传的数据。

- 然后触发Fast Recovery 的条件是： ( **( snd.fack – snd.una ) > (3\*MSS)** ) || (dupacks == 3) ) 。这样一来，就不需要等到3个duplicated acks才重传，而是只要sack中的最大的一个数据和ack的数据比较长了（3个MSS），那就触发重传。在整个重传过程中cwnd不变。直到当第一次丢包的snd.nxt<=snd.una（也就是重传的数据都被确认了），然后进来拥塞避免机制——cwnd线性上涨。

我们可以看到如果没有FACK在，那么在丢包比较多的情况下，原来保守的算法会低估了需要使用的window的大小，而需要几个RTT的时间才会完成恢复，而FACK会比较激进地来干这事。 但是，FACK如果在一个网络包会被 reordering的网络里会有很大的问题。

***

## 4. TCP优化

> 善意提醒：Linux本身的默认配置能应对大多数场景，因此一般情况下其实是不需要做优化的，如果确实有场景需要针对TCP优化，请先搞懂参数真实的功能再优化，不要随便踩坑。。。

### 4.1 三次握手下的问题和解决方案

* **关于建连接时SYN超时**。试想一下，如果server端接到了clien发的SYN后回了SYN-ACK后client掉线了，server端没有收到client回来的ACK，那么，这个连接处于一个中间状态，即没成功，也没失败。于是，server端如果在一定时间内没有收到的TCP会重发SYN-ACK。在Linux下，默认重试次数为5次，重试的间隔时间从1s开始每次都翻售，5次的重试时间间隔为1s, 2s, 4s, 8s, 16s，总共31s，第5次发出后还要等32s都知道第5次也超时了，所以，总共需要 1s + 2s + 4s+ 8s+ 16s + 32s = 2^6 -1 = 63s，TCP才会把断开这个连接。
* **关于SYN Flood攻击**。一些恶意的人就为此制造了SYN Flood攻击——给服务器发了一个SYN后，就下线了，于是服务器需要默认等63s才会断开连接，这样，攻击者就可以把服务器的syn连接的队列耗尽，让正常的连接请求不能处理。对于正常的请求，你应该调整三个TCP参数可供你选择，第一个是：tcp_synack_retries 可以用他来减少重试次数；第二个是：tcp_max_syn_backlog，可以增大SYN连接数；第三个是：tcp_abort_on_overflow 处理不过来干脆就直接拒绝连接了。
* **关于ISN的初始化**。ISN是不能hard code的，不然会出问题的——比如：如果连接建好后始终用1来做ISN，如果client发了30个segment过去，但是网络断了，于是 client重连，又用了1做ISN，但是之前连接的那些包到了，于是就被当成了新连接的包，此时，client的Sequence Number 可能是3，而Server端认为client端的这个号是30了。全乱了。[RFC793](http://tools.ietf.org/html/rfc793)中说，ISN会和一个假的时钟绑在一起，这个时钟会在每4微秒对ISN做加一操作，直到超过2^32，又从0开始。这样，一个ISN的周期大约是4.55个小时。因为，我们假设我们的TCP Segment在网络上的存活时间不会超过Maximum Segment Lifetime（缩写为MSL – [Wikipedia语条](http://en.wikipedia.org/wiki/Maximum_Segment_Lifetime)），所以，只要MSL的值小于4.55小时，那么，我们就不会重用到ISN。

### 4.2 四次挥手下的问题和解决方案

* **关于ISN的初始化**。ISN是不能hard code的，不然会出问题的——比如：如果连接建好后始终用1来做ISN，如果client发了30个segment过去，但是网络断了，于是 client重连，又用了1做ISN，但是之前连接的那些包到了，于是就被当成了新连接的包，此时，client的Sequence Number 可能是3，而Server端认为client端的这个号是30了。全乱了。[RFC793](http://tools.ietf.org/html/rfc793)中说，ISN会和一个假的时钟绑在一起，这个时钟会在每4微秒对ISN做加一操作，直到超过2^32，又从0开始。这样，一个ISN的周期大约是4.55个小时。因为，我们假设我们的TCP Segment在网络上的存活时间不会超过Maximum Segment Lifetime（缩写为MSL – [Wikipedia语条](http://en.wikipedia.org/wiki/Maximum_Segment_Lifetime)），所以，只要MSL的值小于4.55小时，那么，我们就不会重用到ISN。

- **关于 MSL 和 TIME_WAIT**。通过上面的ISN的描述，相信你也知道MSL是怎么来的了。我们注意到，在TCP的状态图中，从TIME_WAIT状态到CLOSED状态，有一个超时设置，这个超时设置是 2*MSL（[RFC793](http://tools.ietf.org/html/rfc793)定义了MSL为2分钟，Linux设置成了30s）为什么要这有TIME_WAIT？为什么不直接给转成CLOSED状态呢？主要有两个原因：1）TIME_WAIT确保有足够的时间让对端收到了ACK，如果被动关闭的那方没有收到Ack，就会触发被动端重发Fin，一来一去正好2个MSL，2）有足够的时间让这个连接不会跟后面的连接混在一起（你要知道，有些自做主张的路由器会缓存IP数据包，如果连接被重用了，那么这些延迟收到的包就有可能会跟新连接混在一起）。你可以看看这篇文章《[TIME_WAIT and its design implications for protocols and scalable client server systems](http://www.serverframework.com/asynchronousevents/2011/01/time-wait-and-its-design-implications-for-protocols-and-scalable-servers.html)》
- **关于TIME_WAIT数量太多**。从上面的描述我们可以知道，TIME_WAIT是个很重要的状态，但是如果在大并发的短链接下，TIME_WAIT 就会太多，这也会消耗很多系统资源。只要搜一下，你就会发现，十有八九的处理方式都是教你设置两个参数，一个叫**tcp_tw_reuse**，另一个叫**tcp_tw_recycle**的参数，这两个参数默认值都是被关闭的，后者recyle比前者resue更为激进，resue要温柔一些。另外，如果使用tcp_tw_reuse，必需设置tcp_timestamps=1，否则无效。这里，你一定要注意，**打开这两个参数会有比较大的坑——可能会让TCP连接出一些诡异的问题**（因为如上述一样，如果不等待超时重用连接的话，新的连接可能会建不上。正如[官方文档](https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt)上说的一样“**It should not be changed without advice/request of technical experts**”）。

- - **关于tcp_tw_reuse**。官方文档上说tcp_tw_reuse 加上tcp_timestamps（又叫PAWS, for Protection Against Wrapped Sequence Numbers）可以保证协议的角度上的安全，但是你需要tcp_timestamps在两边都被打开（你可以读一下[tcp_twsk_unique](http://lxr.free-electrons.com/ident?i=tcp_twsk_unique)的源码 ）。我个人估计还是有一些场景会有问题。

- - **关于tcp_tw_recycle**。如果是tcp_tw_recycle被打开了话，会假设对端开启了tcp_timestamps，然后会去比较时间戳，如果时间戳变大了，就可以重用。但是，如果对端是一个NAT网络的话（如：一个公司只用一个IP出公网）或是对端的IP被另一台重用了，这个事就复杂了。建链接的SYN可能就被直接丢掉了（你可能会看到connection time out的错误）。**注：该参数在内核4.12版本之后已删除[commit](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=4396e46187ca5070219b81773c4e65088dac50cc)**

- - **关于tcp_max_tw_buckets**。这个是控制并发的TIME_WAIT的数量，默认值是180000，如果超限，那么，系统会把多的给destory掉，然后在日志里打一个警告（如：time wait bucket table overflow），官网文档说这个参数是用来对抗DDoS攻击的。也说的默认值180000并不小。这个还是需要根据实际情况考虑。

### 4.3 数据传输过程中的问题和解决方案

* 慢启动在一定大小文件上的传输问题。慢启动过程，tcp每次发送大概之前窗口的一倍长度数据。在理想环境下，每次连续发送数据的长度是以2的指数倍数增长的。当窗口剩余字节数为0的时候，tcp将不在发送数据，此时要等待客户端发ack确认数据收到，然后发送端才能根据确认数据的情况来增加窗口长度持续发送。正是由于这种慢启动加滑动窗口发送的机制，导致了一个问题，就是tcp在传输这种半大不小的数据时，在网络稍有延时增加的情况下，会体现出很大比例的延时增加。这个的优化可以通过更改cnwd的初始值来改善，前面说过，linux3.0将cnwd的初始值设置为10*MSS，但上面的特殊场景下，这个值可能就不适合了。具体的[细节](https://zorrozou.github.io/docs/tcp/tcp_slowstart.html)参考。
* **通告窗口长度**，每次传输的数据量受通告窗口长度影响，而通过窗口长度受系统的socket缓存大小影响。参考[该篇](https://zorrozou.github.io/docs/Socket%E7%BC%93%E5%AD%98%E7%A9%B6%E7%AB%9F%E5%A6%82%E4%BD%95%E5%BD%B1%E5%93%8DTCP%E7%9A%84%E6%80%A7%E8%83%BD.html)

***

## 总结

只能说，TCP真的很复杂呀。

**相关书籍和文档推荐**

- **[TCP/IP Illustrated](https://www.amazon.ca/TCP-IP-Illustrated-Protocols-2nd/dp/0321336313/ref=dp_ob_title_bk)**
- **[TCP/IP guide](http://www.tcpipguide.com/free/index.htm)**
- **[High Performance Browser Networking](https://hpbn.co/)**
- **[RFC 793 — Transmission Control Protocol](https://tools.ietf.org/html/rfc793)**

- [RFC 896 — Congestion Control in IP/TCP Internetworks](https://tools.ietf.org/html/rfc896)
- [RFC 1122 — Requirements for Internet Hosts – Communication Layers](https://tools.ietf.org/html/rfc1122)
- [RFC 1323 — TCP Extensions for High Performance](https://tools.ietf.org/html/rfc1323)
- [RFC 2081 — TCP Selective Acknowledgment Options](https://tools.ietf.org/html/rfc2081)
- [RFC 2581 — TCP Congestion Control](https://tools.ietf.org/html/rfc2581)
- [RFC 3168 — The Addition of Explicit Congestion Notification (ECN) to IP](https://tools.ietf.org/html/rfc3168)
- [RFC 3540 — Robust Explicit Congestion Notification (ECN) Signaling with Nonces](https://tools.ietf.org/html/rfc3540)
- [RFC 6633 — Deprecation of ICMP Source Quench Messages](https://tools.ietf.org/html/rfc6633)
- [RFC 6937 — Proportional Rate Reduction for TCP](https://tools.ietf.org/html/rfc6937)
- [Transmission Control Protocol Segment Structure](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#TCP_segment_structure)

***

**参考**

[https://coolshell.cn/articles/11564.html](https://coolshell.cn/articles/11564.html)

[RFC 793](https://tools.ietf.org/html/rfc793)

[TCP/IP guide](http://www.tcpipguide.com/free/index.htm)

[TCP/IP详解卷一: 协议]()

[https://sookocheff.com/post/networking/how-does-tcp-work/](https://sookocheff.com/post/networking/how-does-tcp-work/)

[https://zorrozou.github.io/](https://zorrozou.github.io/)


