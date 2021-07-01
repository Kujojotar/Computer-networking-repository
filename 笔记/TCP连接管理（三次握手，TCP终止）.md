## TCP连接管理（三次握手，TCP终止）

![Applied Sciences | Free Full-Text | TRAP: A Three-Way Handshake Server for  TCP Connection Establishment | HTML](https://www.mdpi.com/applsci/applsci-06-00358/article_deploy/html/images/applsci-06-00358-g001.png)

#### 三次握手过程简述：

第一步：客户端TCP首先向服务器端TCP发送一个特殊的TCP报文段。该报文段不包含应用层数据。但在报文段中的一个标志为SYN被置为1.因此这个报文段被称为SYN报文段。客户随机选择一个初始序号client_isn，并将此编号置于该起始的TCP_SYN报文段的序号字段中。该报文将被封装到一个IP数据包中并发送给服务器。

第二步：服务器从到达的数据报中提取报文段，为TCP分配缓存和变量。，并向该客户发送允许连接的报文段。允许报文段也不包含应用层数据。SYN被置为1，TCP数据包的确认号字段被指为client_isn+1。最后，服务器选择自己的初始号server_isn置于序号字段。该允许连接的报文被称为SYNACK报文段。

第三步：在接到SYNACK报文段后，客户也为该连接分配缓存和和变量。客户主机向服务器发送另外一个报文段，最后一个报文段对允许连接的报文段进行了确认（确认号为server_isn+1）。因为连接已经被建立好了，所以SYN字段被置为0.这个报文段已经可以携带用户到服务器的数据了。

#### TCP关闭

参与TCP连接的任意一方可以终止该连接。连接结束后，资源都将被释放。

![TCP Connection Termination - GeeksforGeeks](https://media.geeksforgeeks.org/wp-content/uploads/CN.png)

简述：客户TCP向服务器TCP发送一个特殊的TCP报文段。这个特殊的报文段标志为FIN被置为1.当服务器接收到该报文段后，就向发送方回传一个确认报文段。然后，服务器发送自己的终止报文段（FIN被置为1），客户对服务器的终止报文段进行确认。此时，两台主机上用于该连接的所有资源都被释放了。

##### TCP状态（TCP state）

![TCP Connection Termination - GeeksforGeeks](https://media.geeksforgeeks.org/wp-content/uploads/CN-1.png)

![IBM Knowledge Center](https://www.ibm.com/support/knowledgecenter/SSLTBW_2.1.0/com.ibm.zos.v2r1.halu101/dwgl0004.gif)

主机不接受连接的情况

主机将向源发送一个特殊的重置报文段。该TCP报文段将RST置为1，告诉源"没有报文段的那个套接字，不要再发送连接请求了"。UDP连接将发送一个特殊的ICMP报文段。



##### TCP拥塞控制

TCP拥塞控制原则：

1.一个丢失的报文段意味着拥塞，因此当丢失报文段时应当降低TCP发送方的速率。

2.一个确认的报文段指示该网络正在向接收方交付发送方的报文段，当先前未确认报文段的确认到达时，可以提高发送方的速率。

3.带宽检测。TCP调节的策略是增加其速率以响应到达的ACK，除非出现丢包事件时才减小传输速率。



##### TCP拥塞控制算法

包括内容：cwnd，慢启动，拥塞避免，快速恢复。慢启动，拥塞避免为强制部分，快速恢复为推荐部分。

cwnd的一点小补充：LastByteRecv-LastByteRead<=min{rwnd(流量控制),cwnd(拥塞控制)}

1.慢启动

![Home Page](https://www.isi.edu/nsnam/DIRECTED_RESEARCH/DR_HYUNAH/D-Research/e-slow.gif)

一条TCP连接开始的时候，cwnd的值常被设为一个MSS的较小值。每当一个传输的报文段被确认就增加一个MSS。当出现丢包事件时，ssthresh被设置为cwnd/2。cwnd被置为1并重新开始慢启动过程。当cwnd增加到ssthresh值时更加谨慎地增加cwnd。另一种结束慢启动的方式是接收到三个冗余ACK，这时TCP进行快速重传。

2.拥塞避免

当cwnd的值达到ssthresh时，经过一个RTT便将cwnd的值翻一倍显得鲁莽。TCP采用更为保守的方法即一个RTT只增加一个MSS。除非出现丢包事件或三个冗余CK结束这种状态。

3.快速恢复

在快速恢复中，对于收到的每个冗余ACK增加一个MSS。在丢失报文段的一个ACK传来后，TCP在降低cwnd的值后进入拥塞避免状态。如果出现超时事件，同步操作并迁移到慢启动状态。

![TCP Slow-Start and Congestion Avoidance phase. | Download Scientific Diagram](https://www.researchgate.net/profile/Drghassan-Abed/publication/256868797/figure/fig1/AS:297662294315010@1447979629987/TCP-Slow-Start-and-Congestion-Avoidance-phase.png)