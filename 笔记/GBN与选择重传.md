## GBN与选择重传

协议优势:解决rdt3.0发送方管道利用率过小的缺陷，提高性能。

此技术也被称为流水线技术，对可靠传输协议的影响

1.每个输送中的分组必须有一个唯一的序号，而且也有许多个在传输中未确认的报文。

2.协议的发送方和接收端两端缓存多个分组。



### GBN协议

##### GBN特征

特点:允许发送方发送多个分组不需等待确认，受限于未确认的分组数不能超过某个最大值N。

![image-20210303123820370](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210303123820370.png)

如果将基序号定义为最早的未确认分组的序号，下一个序号定义为最小的未使用序号，可将序号范围分为4段。

[0,base-1]-已经发送并被确认的分组

[base,nextseqnum-1]-已经发送但未被确认的分组

[nextseqnum,N+base-1]将要立即被发送的分组

大于等于N+bas-目前不能使用

N被称为窗口长度，GBN协议常常被称为滑动窗口协议



实践中，一个分组的序号承载在分组首部的一个固定长度的字段中，如果分组字段序号的比特数是k，那么序号的范围在[0,2^k-1]。



##### GBN运行简介

![image-20210303124740760](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210303124740760.png)

![image-20210303124752716](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210303124752716.png)



发送方响应事件：

1.上层的调用。上层调用rdt_send()时，发送方先检查窗口是否已满。窗口未满则产生一个分组并发送，相应的更新变量。如果窗口已满，发送方将数据返回给上层，隐式地指示上层该窗口已满，上层过一会再试试。实际实现中，更可能缓存这个分组或者使用同步机制（一个信号量或标志）允许上层在窗口不满时才调用rdt_send()。

2.收到ACK。

3.超时事件。如果出现超时，发送方重传所有已被发送但未被确认的分组。如果收到一个ACK，但仍有

已被发送但未被确认的分组，定时器将会重新启动。如果没有，记事起将会终止。

接收方

如果一个序号为n的分组被正确的接收到，并且按序，接收方为分组n发送一个ACK，并将分组中的数据交付上层。其他情况下，接收方丢弃该分组，并为最近按序接收到的分组重新发送ACK。



协议优点：接收方不需缓存任何失序的分组。而发送方需要维护窗口的上下边界和nextseqnum在窗口中的位置。

![image-20210303130209050](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210303130209050.png)



### 选择重传

GBN性能问题：当窗口长度和带宽时延都很大时，单个分组的出错可能导致GBN重传大量分组，很多分组都没必要进行重传。随着信道差错率的增加，流水线可能被这些不必要的分组所充斥。

SR协议通过让发送方仅重传那些它怀疑在接收方出错的分组而避免了不必要的重传。这些个别的分组由发送方逐个重新发送并确认正确接收。

![image-20210303132744625](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210303132744625.png)

![image-20210303132815042](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210303132815042.png)

SR接收方将确认一个正确接收的分组而不管其是否按序。失序的分组将会被缓存直到所有丢失的分组皆被接受到为止。

![image-20210303133113518](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210303133113518.png)

![image-20210303133126438](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210303133126438.png)

注意到2很重要。在SR协议中，可能接收端发送的ACK因各种原因没有被发送端接收。因此若接收端不对重复接收到的分组进行确认的话，发送端的窗口将不会向下移动。在SR协议中，发送端和接收端的窗口不总是一致的。

![image-20210303133751213](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210303133751213.png)

![image-20210303133909858](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210303133909858.png)