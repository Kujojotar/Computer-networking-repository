## 计算机网络-运输层编程LAB

#### abp比特交换协议版本

由于一些奇怪的原因，这个任务写废了，这边记录以下大致的思路流程，顺便下午或晚上趁热打铁把GBN给写了。

先看以下大致的结构。

```
struct msg
{
    char data[20];
};
```

这边定义了运输的数据载荷

```
struct pkt
{
    int seqnum;
    int acknum;
    int checksum;
    char payload[20];
};
```

运输的包

```
void starttimer(int AorB, float increment);
void stoptimer(int AorB);
void tolayer3(int AorB, struct pkt packet);
void tolayer5(int AorB, char datasent[20]);
```

这部分是已经写好的，startttimer开始计时，stoptimer停止即使，tolayer3模拟数据包向下层发送，tolayer5模拟数据包往上层接收。

```
enum SenderState
{
    WAIT_LAYER5,
    WAIT_ACK
};
```

定义了发送方可能处于的两个状态，等待上层接收或者是等待Ack。

```
struct Sender
{
    enum SenderState state;
    int seq;
    float estimated_rtt;
    struct pkt last_packet;
} A;

struct Receiver
{
    int seq;
} B;
```

发送方和接收方的定义

```
int get_checksum(struct pkt *packet)
{
    int checksum = 0;
    checksum += packet->seqnum;
    checksum += packet->acknum;
    for (int i = 0; i < 20; ++i)
        checksum += packet->payload[i];
    return checksum;
}
```

获取checksum

```
/* called from layer 5, passed the data to be sent to other side */
void A_output(struct msg message)
{
    if (A.state != WAIT_LAYER5)
    {
        printf("  A_output: not yet acked. drop the message: %s\n", message.data);
        return;
    }
    printf("  A_output: send packet: %s\n", message.data);
    struct pkt packet;
    packet.seqnum = A.seq;
    memmove(packet.payload, message.data, 20);
    packet.checksum = get_checksum(&packet);
    A.last_packet = packet;
    A.state = WAIT_ACK;
    tolayer3(0, packet);
    starttimer(0, A.estimated_rtt);
}

/* need be completed only for extra credit */
void B_output(struct msg message)
{
    printf("  B_output: uni-directional. ignore.\n");
}

/* called from layer 3, when a packet arrives for layer 4 */
void A_input(struct pkt packet)
{
    if (A.state != WAIT_ACK)
    {
        printf("  A_input: A->B only. drop.\n");
        return;
    }
    if (packet.checksum != get_checksum(&packet))
    {
        printf("  A_input: packet corrupted. drop.\n");
        return;
    }

    if (packet.acknum != A.seq)
    {
        printf("  A_input: not the expected ACK. drop.\n");
        return;
    }
    printf("  A_input: acked.\n");
    stoptimer(0);
    A.seq = 1 - A.seq;
    A.state = WAIT_LAYER5;
}

/* called when A's timer goes off */
void A_timerinterrupt(void)
{
    if (A.state != WAIT_ACK)
    {
        printf("  A_timerinterrupt: not waiting ACK. ignore event.\n");
        return;
    }
    printf("  A_timerinterrupt: resend last packet: %s.\n", A.last_packet.payload);
    tolayer3(0, A.last_packet);
    starttimer(0, A.estimated_rtt);
}

/* the following routine will be called once (only) before any other */
/* entity A routines are called. You can use it to do any initialization */
void A_init(void)
{
    A.state = WAIT_LAYER5;
    A.seq = 0;
    A.estimated_rtt = 15;
}

/* Note that with simplex transfer from a-to-B, there is no B_output() */

void send_ack(int AorB, int ack)
{
    struct pkt packet;
    packet.acknum = ack;
    packet.checksum = get_checksum(&packet);
    tolayer3(AorB, packet);
}

/* called from layer 3, when a packet arrives for layer 4 at B*/
void B_input(struct pkt packet)
{
    if (packet.checksum != get_checksum(&packet))
    {
        printf("  B_input: packet corrupted. send NAK.\n");
        send_ack(1, 1 - B.seq);
        return;
    }
    if (packet.seqnum != B.seq)
    {
        printf("  B_input: not the expected seq. send NAK.\n");
        send_ack(1, 1 - B.seq);
        return;
    }
    printf("  B_input: recv message: %s\n", packet.payload);
    printf("  B_input: send ACK.\n");
    send_ack(1, B.seq);
    tolayer5(1, packet.payload);
    B.seq = 1 - B.seq;
}

/* called when B's timer goes off */
void B_timerinterrupt(void)
{
    printf("  B_timerinterrupt: B doesn't have a timer. ignore.\n");
}

/* the following rouytine will be called once (only) before any other */
/* entity B routines are called. You can use it to do any initialization */
void B_init(void)
{
    B.seq = 0;
}
```

暂且略去各个方法的详解，接下来的时间看看能不能模仿这个样子把GBN给弄出来。

### GBN

这边只贴一下自己写的代码部分与个人理解，其他写好的代码样式与ABP基本一致。

```C
#define WINDOW_LENGTH 8
struct sender{
    int basenum;
    int nextnum;
    float rdt;
    int windowSize;
    struct pkt packets[WINDOW_LENGTH];
} A;

struct receiver{
    int basenum;
    int windowSize;
} B;

```

这块代码是对GBN传输方与接收方的定义，传输方需要保留basenum，nextnum，一个设定好的rdt，窗口长度以及窗口。接收方保留basenum与窗口长度。

```C
int get_checksum(struct pkt* packet){
    int checksum=0;
    checksum+=packet->seqnum;
    checksum+=packet->acknum;
    for(int i=0;i<20;i++){
        checksum+=packet->payload[i];
    }
    return checksum;
}

void send_all_packets(){
    printf("Time is up,sender resend all packets!!\n");
    for(int i=A.basenum;i<A.nextnum;i++){
        printf("Sender resend the packet seq=%d ack=%d msg=%s \n",A.packets[i].seqnum,A.packets[i].acknum,A.packets[i].payload);
        tolayer3(0,A.packets[i%WINDOW_LENGTH]);
    }
}

```

两个辅助函数，上面的不细说了，下面的就是将窗口中的报文全部重传一次。

```C
/* called from layer 5, passed the data to be sent to other side */
void A_output(struct msg message){
    if(A.nextnum-A.basenum>=WINDOW_LENGTH){
        printf("Hi,Layer5.Sorry,my window is full.\n");
        return ;
    }
    A.nextnum++;
    struct pkt packet;
    packet.seqnum=A.nextnum-1;
    packet.acknum=A.nextnum-1;
    memmove(packet.payload,message.data,20);
    packet.checksum=get_checksum(&packet);
    A.packets[(A.nextnum-1)%WINDOW_LENGTH]=packet;
    tolayer3(0,packet);
    if(A.basenum==A.nextnum-1){
        starttimer(0,A.rdt);
    }
}

```

A的上层调用代码，注意到当窗口已满时不能再接受上层的请求，故在函数开始进行一次判断。剩余的完成将message封装为packet再调用下层进行传输的操作。

```C
/* need be completed only for extra credit */
void B_output(struct msg message){
   printf("It is no user for me to get the extra credit,hahaha!!\n");
}

```

~~获得额外学分才要的，所以直接略过，感情这玩意儿是个作业？~~

```C
/* called from layer 3, when a packet arrives for layer 4 */
void A_input(struct pkt packet){
   if(packet.checksum!=get_checksum(&packet)){
       printf("Holy shit,it is corrupt packet!");
       stoptimer(0);
       tolayer3(0,A.packets[A.basenum%WINDOW_LENGTH]);
       starttimer(0,A.rdt);
       return ;
   }
   if(packet.seqnum>A.basenum&&packet.seqnum<A.nextnum){
       printf("Oh,it seems packets i sent have been disorder now.\n");
       tolayer3(0,A.packets[A.basenum%WINDOW_LENGTH]);
       return ;
   }
   if(packet.seqnum==A.basenum){
       stoptimer(0);
       A.basenum++;
       if(A.basenum==A.nextnum){
           return ;
       }else{
           tolayer3(0,A.packets[A.basenum%WINDOW_LENGTH]);
           starttimer(0,A.rdt);
       }
   }
}

```

A接受B返回ack的处理，注意到信道传输中报文有可能受损，这里需要对报文进行一次受损判断，如果是受损的报文则重发packets[basenum]。如果接受到的ack位于basenum和nextnum之间，由于接收方并不会对报文进行缓存处理，所以A知道报文失序到达后会重发packets[basenum]的报文。当A接收到正确且未受损的报文将自己的basenum++，然后继续发送packets[basenum].

```C
/* called when A's timer goes off */
void A_timerinterrupt(void){
    send_all_packets();
    starttimer(0,A.rdt);
}
```

定时器超时后，A将重新发送所有报文对重新计时。

```C
/* the following routine will be called once (only) before any other */
/* entity A routines are called. You can use it to do any initialization */
void A_init(void){
    A.basenum=0;
    A.nextnum=0;
    A.windowSize=WINDOW_LENGTH;
    A.rdt=15.0;
}

```

A的初始化代码，这个没啥好讲的。

```C
/* Note that with simplex transfer from a-to-B, there is no B_output() */

void receiver_respond(int acknum,int checksum){
    struct pkt packet;
    packet.seqnum=acknum;
    packet.acknum=acknum;
    packet.checksum=checksum;
    tolayer3(1,packet);
}

```

将B返回的报文写成函数，更加方便罢了。

```c
/* called from layer 3, when a packet arrives for layer 4 at B*/
void B_input(struct pkt packet){
    printf("I get a packet,seqnum=%d acknum=%d payload=%s\n",packet.seqnum,packet.acknum,packet.payload);
    if(packet.checksum!=get_checksum(&packet)){
       printf("Holy shit,it is corrupt packet!");
       receiver_respond(packet.acknum,get_checksum(&packet));
       return ;
    }
    if(packet.seqnum>B.basenum&&packet.seqnum<B.basenum+B.windowSize){
       printf("Disorder packet!");
       receiver_respond(packet.acknum,get_checksum(&packet));
       return ;
    }
    if(packet.seqnum==B.basenum){
        printf("Well,the packet is what I want!!\n");
        B.basenum++;
        struct pkt packet1;
        packet1.seqnum=packet.seqnum;
        packet1.acknum=packet.acknum;
        packet1.checksum=packet.checksum;
        memmove(packet1.payload,packet.payload,20);
        tolayer3(1,packet1);
        tolayer5(1,packet.payload);
        return ;
    }
    if(packet.seqnum<B.basenum&&packet.seqnum>=B.basenum-WINDOW_LENGTH){
        printf("Oh,it is a dumplicate packet");
        struct pkt packet1;
        packet1.seqnum=packet.seqnum;
        packet1.acknum=packet.acknum;
        packet1.checksum=packet.checksum;
        memmove(packet1.payload,packet.payload,20);
        tolayer3(1,packet1);
        return ;
    }
    printf("Sorry,this packet I receive is an illegal packet");
}

```

对B接受报文，也要求判断是否受损以及对序号进行相应的检查。B接受失序的报文虽然会直接丢弃，但仍会发送一个Ack以保证A知道报文到达的失序，不过返回的报文是一个烂报文，不用担心一些诡异的情况。B会将basenum-WINDOW-LENGTH的报文重新发送，因为B无法确定A是否完好的接收到了其返回的ack。

```C
/* called when B's timer goes off */
void B_timerinterrupt(void){
    printf("  B_timerinterrupt: B doesn't have a timer. ignore.\n");
}
/* the following rouytine will be called once (only) before any other */
/* entity B routines are called. You can use it to do any initialization */
void B_init(void){
    B.basenum=0;
    B.windowSize=WINDOW_LENGTH;
}

```

一个初始化代码和一个timeout代码，很简单的实现，这样结束了。