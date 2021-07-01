## 网际协议：IPv4，寻址，IPv6

#### Ipv4数据报格式

![What is Internet protocol? – IP definition - IONOS](https://www.ionos.co.uk/digitalguide/fileadmin/DigitalGuide/Screenshots/EN-Internet-Protocol-1.PNG)

关键字段：

​    版本号：规定了数据报的IP协议版本。

​    首部长度：确定IP数据报中的载荷实际开始的地方。大多数IP数据报不包含选项，一般具有20字节的首部。

​    服务类型：将不同类型的IP数据报区别开。

   数据报长度：IP数据报的总长度，以字节记，理论最大长度为65535字节。

   标识，标志，片偏移：与IP分片有关。

   寿命：确保数据报不会永远在网络中循环，每经过一台路由器该字段减1，减为0时将被抛弃。

   协议：指示IP数据报文应交给哪个特定的运输层协议，通常在到达最终目的地时才有用。

   首部校验和：和udp类似。

  源和目的地IP地址。

  选项：允许IP首部被扩展。因为使问题复杂等原因在Ipv6中被移除。

  数据：有效载荷，大多数情况下包含交给UDP或TCP的运输层报文段，也可以承载其他数据。



IP报文段首部一般20字节，如果承载TCP报文段，将有40字节的首部以及应用层报文。



IPv4数据报分片

一个链路层帧能承载的最大数据量叫做最大传送单元。有时出链路的MTU小于IP数据报长度，需要将IP数据报分为更小的片。IPv4的设计者决定将片的组装工作放到端系统中。当生成一个数据报时，发送主机在为该数据报设置源和目的地址和标识号。以便于目的主机确认收到的片是那些较大的数据报的组成部分。由于IP的不可靠，最后一个片的标志比特将被设为0，其他的均为1.让目的主机确认是否收到初始数据报的最后一个片。



  ![查看源图像](https://tse3-mm.cn.bing.net/th/id/OIP.AqmpwNq0jMWPuXO6oNBvWAHaFS?pid=ImgDet&rs=1)





### DHCP

![A DHCP Client cannot obtain an IP address [All About Switches]](https://forum.huawei.com/enterprise/en/data/attachment/forum/202003/12/113904og7bm2c50i593yqy.png?26.png)

新到达的主机获取IP流程



DHCP服务器发现：使用DHCP发送报文完成，客户在UDP分组中向端口67发送该发现报文。该UDP分组被封装在一个IP数据报中。IP数据报中使用广播地址255.255.255.255和源地址0.0.0.0 。链路层将该帧广播到所有该子网连接的节点。

DHCP服务器提供：DHCP服务器收到一个DHCP发现报文时，用DHCP提供报文像客户做出响应。该报文仍采用广播地址，并提供发现报文的事务ID,向客户推荐的IP地址，网络掩码和IP地址租用期。

DHCP请求：新到达的用户从服务器中选择一个用DHCP请求报文进行响应，回显配置参数。

DHCP ACK：服务器用DHCP ACK报文进行响应，证实所请求参数。