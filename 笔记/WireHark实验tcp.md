# WireHark实验

(3)1.What is the IP address and TCP port number used by the client computer (source) that is transferring the file to gaia.cs.umass.edu?

ip:113.54.241.175     port:62582

2.What is the IP address of gaia.cs.umass.edu? On what port number is it sending and receiving TCP segments for this connection?

ip:128.119.245.12     port:80

4.What is the sequence number of the TCP SYN segment that is used to initiate the TCP connection between the client computer and gaia.cs.umass.edu? What is it in the segment that identifies the segment as a SYN segment?

   seq=0

![image-20210304144751675](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210304144751675.png)

5.What is the sequence number of the SYNACK segment sent by gaia.cs.umass.edu to the client computer in reply to the SYN? What is the value of the Acknowledgement field in the SYNACK segment? How did gaia.cs.umass.edu determine that value? What is it in the segment that identifies the segment as a SYNACK segment?

seq:0 

ack:1 

The seq in SYN it has received plus 1.

![image-20210304145110187](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210304145110187.png)

6.What is the sequence number of the TCP segment containing the HTTP POST command? Note that in order to find the POST command, you’ll need to dig into the packet content field at the bottom of the Wireshark window, looking for a segment with a “POST” within its DATA field.

seq:1 

7.Consider the TCP segment containing the HTTP POST as the first segment in the TCP connection. What are the sequence numbers of the first six segments in the  TCP connection (including the segment containing the HTTP POST)? At what time was each segment sent? When was the ACK for each segment received? Given the difference between when each TCP segment was sent, and when its acknowledgement was received, what is the RTT value for each of the six segments? What is the EstimatedRTT value (see Section 3.5.3, page 242 in text) after the receipt of each ACK? Assume that the value of the EstimatedRTT is equal to the measured RTT for the first segment, and then is computed using the EstimatedRTT equation on page 242 for all subsequent segments.

 Note: Wireshark has a nice feature that allows you to plot the RTT for each of the TCP segments sent. Select a TCP segment in the “listing of captured packets” window that is being sent from the client to the gaia.cs.umass.edu server. Then select: Statistics->TCP Stream Graph- >Round Trip Time Graph

1,733,2113,3493,4873,6253

![image-20210304152826817](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210304152826817.png)

![image-20210304153043710](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210304153043710.png)

8.What is the length of each of the first six TCP segments?4

732,1380,1380,1380,1380,1380

9.What is the minimum amount of available buffer space advertised at the received for the entire trace? Does the lack of receiver buffer space ever throttle the sender

131072-183296-192000-261632,yes

10.. Are there any retransmitted segments in the trace file? What did you check for (in the trace) in order to answer this question?

~~看wireshark标识~~   Seq相同的数据报在较长时间间隔重发两次

11.How much data does the receiver typically acknowledge in an ACK? Can you identify cases where the receiver is ACKing every other received segment (see Table 3.2 on page 250 in the text).

2760，

12.What is the throughput (bytes transferred per unit time) for the TCP connection? Explain how you calculated this value.

~~这真不会~~