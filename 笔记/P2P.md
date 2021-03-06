# P2P

**P2P**概述：对总是打开的基础设施服务器有**最小的（或者没有）依赖**。与之相反，成对间歇连接的主机（或称为**对等方**）彼此**直接通信**。这些对等方并不为服务提供商所拥有，而是受用户控制的桌面计算机和膝上计算机。





实例：P2P文件分发

  在文件分发情境中，如果使用服务器-客户端结构，为每个对等方发送一个文件的拷贝将为服务器带来极大的负担并消耗大量的服务器带宽。而在P2P结构中，每个对等方能够重新分发它所有的该文件的任何部分，从而协助服务器。

   

  1.P2P结构的自扩展性



![image-20210105141628456](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210105141628456.png)

us表示服务器接入链路的上载速率

ui表示第i个对等方接入链路的上载速率

di表示第i个对等方接入链路的下载速率

F分发的文件长度

N获得该副本的对等方的数量



C/S结构时间分析

最少时间：NF/us-由于服务器上载速率瓶颈要求的最小上载时间

dmin=min{d1，d2 。 。 。 dn}

最小分发时间：F/dmin

Time=max{F/dmin，NF/us}



P2P结构时间分析

开始时，仅有服务器拥有文件

最小分发时间：F/us

相同的是：拥有最小下载速率的对等方接受文件的时间

dmin=min{d1，d2 。 。 。 dn}

F/dmin

最后：总上载能力等于服务器上载速率和对等方上载速率

ut=sigma（ui）

Dp2p>=max{F/us，F/dmin，NF/us+sigma（ui）}

![image-20210105142613491](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210105142613491.png)





**BitTorrent**

BitTorrent是一种**用于文件分发的流行P2P协议**。用该协议术语，参与一个特定文件分发的所有对等方的集合被称为一个**洪流**。在一个洪流中的对等方彼此下载等长度的文件快，典型的块长度是256KB.当一个对等方首次加入一个洪流时，它没有块。随着时间流逝，它积累了越来越多的块。当他下载块时，也为其他对等方上载块。同时，任何对等方可能在任何时候仅具有块的子集就离开该洪流，并在以后重新加入该块洪流中。

每个洪流具有一个基础设施节点，称为**追踪器**。当任何一个对等方加入某洪流时，它向追踪器注册自己，并周期性地通知注册器自己仍在该洪流中。以这种方式，追踪器跟踪正参与在洪流中的对等方。一个给定的洪流可能在任何时刻具有数以百计或以千计的对等方。

当一个新的对等方Alice加入该洪流时，追踪器随机地从参与对等方的集合中选择对等方的一个子集，并将子集中的对等方的IP地址发送给Alice。Alice与子集上每个对等方创建并行的TCP连接。我们把所有与Alice建立TCP连接的对等方称为**临近对等方**。

![image-20210105144519889](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210105144519889.png)

在给定时间内，每个对等方就有来自该文件的块子集，并且不同的对等方具有不同的子集。Alice周期性地询问每个临近对等方所具有的块列表，并通过该列表对目前还没有的块发出请求。（过程通过TCP）。

在任何时刻，Alice具有块的子集并知道它地邻居具有哪些块。利用这些信息，她可以使用最稀缺优先技术首先请求邻居副本中数量最少的块，最稀缺的更为迅速的得到重新分发。

为了决定响应哪些请求，BitTorrent根据目前能够以最高速率向她提供数据的邻居，给出优先权。

Alice对于每个邻居持续测量接收到比特的速率，并确定最高速率流入的4个邻居，每十秒重新计算一次。没过30秒随机选择另一个邻居向其发送块。这4个对等方称为**疏通**。

![image-20210105145627429](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210105145627429.png)





分布式散列表

  本节内容介绍在P2P网络中如何实现一个简单的数据库。该数据库只包含键值对，就是熟悉的散列表。



在数以百计的对等方上存储键值对，在P2P系统中，每个对等方将保持键值对仅占总体的一个小子集。我们将允许任何对等方用一个特别的键来查询**分布式数据库**。分布式数据库也将允许在数据库中插入新键值对。这样一种分布式数据库被称为**分布式散列表**。（Distributed Hash Table）



建立**DHT**：为每个对等方分配一个标识符。规定最邻近的对等方定义为键的**最邻近后继**。如果这些键恰好等于这些对等方标识符之一，我们在完全匹配的对等放中存储键值对；如果键大于所有的对等方标识符，使用模2^n 就是散列表那玩意 在具有最小标识符的对等放中存储键值对。

确定对等方



环形DHT

![image-20210105151348065](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210105151348065.png)

简单来说a图就是环形链表查找，b图加入了快捷路径。

![image-20210105151453262](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210105151453262.png)

![image-20210105151603426](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210105151603426.png)





对等方扰动

  P2P系统中，对等方随时的到来和离去会破环平衡，一下给出解决方案。

为了处理对等方扰动，我们要求每个对等方联系其第一个和第二个后继，并周期性确认它的两个后继是存活的。

a图中，假定对等方5突然离开。4，3得知5已经离开，需更新它们的后继信息状态。

4用它的第二个后继代替第一个后继。4用新的后继询问它的直接后继10的标识符与IP地址。然后对等方4将10标记为它的第二个后继。



加入情景

  若对等方13要加入，它须向1发送报文，获取它加入位置后的前任与后继，然后13加入DHT并通知12更改后继信息。

![image-20210105152524316](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210105152524316.png)