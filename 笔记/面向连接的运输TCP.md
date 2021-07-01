# 面向连接的运输TCP

简述：TCP是因特网运输层的面向连接的可靠的运输协议。为了提供可靠数据传输，TCP依赖于差错检验，重传，累计确认，定时器以及序号和确认号的首部字段。

##### TCP连接

TCP被称为面向连接的，这是因为在一个应用进程向另一个应用进程发送数据前，这两个进程先相互握手，即互相发送某些预备报文段，以确保数据传输的参数。作为建立TCP连接的一部分，连接的双方都将初始化和TCP连接相关的许多TCP状态变量。

TCP连接是一条逻辑连接，其共同状态仅保留在两个通信端系统的TCP程序中。

TCP提供**全双工服务**（双向流动），也总是**点对点**（一次连接的主体是一对主机）。

**三次握手**：客户先发送一个特殊的TCP报文段，服务器用另一个特殊的TCP报文段来响应，最后，客户用第三条特殊报文段来作响应。前两个报文段不承载有效载荷（不包含应用层数据），而第三个报文段可以承载有效载荷。

建立好TCP连接后，两个应用进程便可以开始互相发送数据了。客户进程通过套接字传递数据流。TCP将这些数据引导到该连接的**发送缓存**。TCP可以从缓存中取出并放入报文段的数据数量受限于最大报文段长度（MSS）-MSS通常由发送主机的最大链路层帧长度决定（MTU）。

TCP为每块客户数据配上一个TCP首部，从而形成多个TCP报文段。这些报文段被下发给网络层，网络层将其封装在网络层IP数据报中。这些IP数据报被发送到网络中。当TCP在另一端接收到一个报文段后，该报文段的数据就被放入该TCP连接的接收缓存中。应用程序在缓存中读取数据流。TCP连接的每一端都有各自的发送缓存和接收缓存。

TCP连接的组成成分包括：一台主机的缓存，变量和进程连接的套接字，以及另一台主机上的缓存，变量和进程连接的套接字。在两台主机间的网络设备（路由器，交换机和中继器）中，没有为该连接分配任何缓存和变量。

##### TCP报文段结构

##### ![Exploring the anatomy of a data packet - TechRepublic](https://tr1.cbsistatic.com/hub/i/2015/06/03/596ecee7-0987-11e5-940f-14feb5cc3d2a/r00220010702mul01_02.gif)



TCP报文段由一个首部字段和一个数据字段组成。数据字段包括一块应用程序。当TCP发送一个大文件，该文件被划分为若干个长度为MSS的数据块（通常除最后一块）。交互式小程序长度一般小于MSS。

首部包括源端口号，目的端口号，用于多路分解/复用。

TCP和UDP一样，也包括检验和字段。

32比特的序号字段和32比特的确认号字段。用于实现可靠数据传输服务。

16比特的接收窗口字段。用于流量控制。

4比特的首部长度字段。指示TCP首部长度。由于TCP选项字段的原因，首部的长度是可变的。（TCP首部字段的典型长度是20字节）。

可选与变长的选项字段，该字段用于发送方和接收方协商MSS时，或在高速网络环境下用作窗口调节因子使用。

6比特的标志字段。ACK比特用于指示确认字段的值是有效的，即该报文段包括一个对已被成功接收报文段的确认。BST,SYN,FIN比特用于连接的建立和拆除。拥塞通告中使用了CWR和ECE比特。PSH比特上置时，接收方应立即将数据交给上层。URG比特用于指示报文段里存在着被发送端上层实体置为紧急的数据。紧急数据的最后一个字节由16比特的紧急数据指针字段指出。当紧急数据存在并给出指向紧急数据尾指针的时候，TCP必须通知接收端的上层实体。

##### 序号和确认号

TCP把数据看成一种无结构的，有序的字节流。一个报文段的序号是该报文段的首字节的字节流编号。如50000字节的文件，MSS为1000字节。TCP为该数据创建500个报文段，第一个报文段序号为0，第二个1000，第三个2000......

确认号

发送主机填充的确认号是发送主机期望从接收主机收到的下一个字节的序号。如A收到了来自B的0-535字节。它期望收到B的536即以后的字节。所以A会在它发往主机B的报文段确认号上填536.因为TCP只确认该流中第一个字节至第一个丢失字节为止的字节，所以TCP被称为累计确认。

TCP设计者在接收端接到失序的数据报时可选择：立刻丢弃这些失序的数据报或保留失序的数据报并等待缺少的字节弥补空隙。一般实际运用中后者更常见。

![Transmission Control Protocol](https://www.net.t-labs.tu-berlin.de/teaching/computer_networking/03.05-Dateien/telnet2.jpe)

##### 往返时间的估计和超时

报文段的样本RTT就是从报文段被发出到对该报文段的确认被收到之间的时间量。大多数TCP的实现仅在某个时刻做一次SampleRTT的测量，而不是为每个报文段测量一个SampleRTT。在任意一个时刻，仅为一个已发送的但目前尚未被确认的报文段估计SampleRTT。在任意时刻，仅为一个已发送的但目前尚未被确认的报文段估计SampleRTT，从而产生一个接近每个RTT的新SampleRTT值。TCP仅为传输一次的报文段测量SampleRTT，它不为重传的报文段计算SampleRTT。

TCP以下列公式计算EstimatedRTT。

EstimatedRTT=(1-α)乘以EstimatedRTT+α乘以SampleRTT （α由编程语言给出，建议值为0.125）

Rtt偏差计算

DevRTT=（1-β）乘以DevRTT+β乘以|SampleRTT-EstimatedRTT|(β推荐值为0.25)

最终超时

TimeoutInterva=EstimatedRTT+4*DevRTT

##### 可靠数据传输

![Transport Layer3-1 TCP sender (simplified) NextSeqNum = InitialSeqNum  SendBase = InitialSeqNum loop (forever) { switch(event) event: data  received from. - ppt download](https://slideplayer.com/7378792/24/images/slide_1.jpg)

TCP发送方高度简化的概括：

事件一：TCP从应用程序中接收数据，将数据封装到一个报文段中，并把该报文段交给IP。每一个报文段都包含一个序号，这个序号就是数据流中的第一个数据字节的字节流编号。当报文段被传给IP，TCP就启动该定时器。该定时器的过期间隔是TimeoutInterval。

事件二：超时，TCP通过重传引起超时的报文段来响应超时事件。然后TCP重启定时器。

事件三：ACK到达。TCP将ACK的值y与SendBase进行比较。SendBase是最早的未被确认的字节的序号。y>SendBase，发送方要更新SendBase变量。

![image-20210303194948134](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210303194948134.png)

![image-20210303195001837](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210303195001837.png)



每当超时，定时器新的过期时间将提高到原来的两倍。



快速重传：

![image-20210303195223870](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210303195223870.png)

一旦接受3个冗余ACK，TCP就会执行快速重传。重传丢失的报文段。

![image-20210303195430219](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210303195430219.png)

TCP与GBN,SR

TCP与GBN显著区别：大多数TCP实现会将正确接收但失序的报文段缓存起来。GBN会重传所有未确认分组n及之后的分组，而TCP仅重传至多一个报文段。在允许TCP接收方有选择地确认失序报文段，即修改TCP**选择确认**后，TCP更像是GBN和SR的结合体。

##### 流量控制

TCP为它的应用程序提供了**流量控制服务**以消除发送方使接收方缓存溢出的可能性。流量控制力图让发送方的发送速度和接收方的读取速度相匹配。TCP发送方也可能因为IP网络的拥塞而被遏制。这种形式的发送方被称为**拥塞控制**。

TCP通过让发送方维护一个称为接收窗口的变量来提供流量控制。接收窗口用于给发送方一个指示-该接收方还有多少可用的缓存空间。TCP是全双工通信的，在连接两端的发送方都各自维护一个接收窗口。假设主机A通过一条TCP连接向主机B发送一个大文件。主机为该连接连续分配了一个接收缓存，并用RcvBuffer表示大小。主机B的应用程序时不时地从缓存读取数据，有以下变量

LastByteRead：主机B上的应用进程从缓存读取的数据流的最后一个字节编号。

LastByteRcvd:从网络中到达的并且已经放入主机B接收缓存的数据流最后一个字节编号。

必有

LastByteRcvd-LastByteRead<=BufferSize

接收窗口用rwnd表示

rwnd=RcvBuffer-[LastByteRcvd-LastByteRead]

![SCTP Congestion Control | Peter's blog](http://indigoo.com/petersblog/wp-content/uploads/2013/05/cwnd-rwnd.png)

主机B通过把当前的rwnd值放入主机A的报文段的接收窗口字段，通知主机A它在该连接的缓存中还有多少可用空间。开始时rwnd=RcvBuffer。

主机A轮流跟踪两个变量，在整个生命周期中保证

LastByteRcvd-LastByteRead<=rwnd

TCP规定，主机B的接收窗口为0，主机A发送只有一个字节的数据报文段。这个报文会被接收方确认，最终缓存将被清空，确认报文将包含一个非0的rwnd值。