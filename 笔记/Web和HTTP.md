# Web和HTTP

### HTTP概况

Web（World Wide Web万维网)的应用层协议是HTTP（HyperText Transfer Protocol，超文本传输协议),HTTP由两个程序实现：一个客户程序和一个服务器程序。客户程序和服务器程序运行在不同的端系统中，通过交换HTTP报文进行会话。HTTP定义了这些报文的结构以及客户和服务器进行报文交换的方式。

Web页面由对象组成。一个对象只是一个文件，诸如一个HTML文件，一个	JEPG图形，一个Java小程序或一个视频片段这样的文件，且它们可通过一个URL地址寻址。多数Web页面含有一个HTML基本文件以及几个引用对象。

每个URL由两部分组成：存放对象的服务器主机名和对象的路径名，如

http：//www.someSchool.edu/someDepart-ment/picture.gif

其中www.someSchhol.edu是主机名，/someDepartment/picture.gif是路径名。

流行的Web服务器有Apache和Microsoft Internet Information Server微软互联网信息服务器

HTTP定义了Web客户向Web服务器请求Web页面的方式，以及服务器向客户传送Web页面的方式。

HTTP使用TCP作为它的支撑运输协议。一旦连接建立，浏览器和服务器就可以通过套接字接口访问TCP（详细见上一节）

HTTP是一个无状态协议：服务器向客户发送被请求的文件，而不储存任何关于客户的状态信息。加入某个特定的客户在几秒内两次请求同一对象，服务器并不会因之前为客户提供了该对象就不在做出任何反应，而是重新发送该对象，就像服务器已经完全忘记不久前所做过的事一样。



#### 非持续连接和持续连接

多数情况下，客户和服务器会在相当长的时间范围内通信，客户发出一系列请求且服务器对这一系列请求做出响应。根据应用情况，这一系列请求可以以规则周期性或者间断地一个一个发出。此时便要求应用程序研制者决定这一系列请求响应对有多个TCP连接完成，还是有相通的TCP连接完成。前者称为非连续连接，后者成为持续连接。HTTP默认使用持续连接，HTTP和服务器也能配置成使用非持续连接。



#### 采用非持续连接的HTTP

![image-20210102120004980](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210102120004980.png)

事实上，根据现代浏览器的调度，可以选择使用串行的或并行的TCP连接，大部分浏览器打开5-10个并行的连接，每个连接处理一个请求响应事务。当然，如果用户愿意，也可以使用10个串行连接，不过使用并行连接可以缩短响应时间。

 往返时间RTT：一个短分组从客户到服务器再返回客户所花费的时间。

![image-20210102120708942](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210102120708942.png)

一般来说，请求并接收一个HTML文件所需的估算时间为2个RTT加上服务器传输HTML文件的时间。

缺点：每次请求都需重新建立一个TCP连接，每次额外为客户和服务器分配TCP缓冲区和保持TCP变量，为Web服务器带来严重负担。每个请求对象经历大致两个RTT时延。





#### 采用持续连接

​    服务器在发送响应后保持该TCP连接打开。一个完整的Web页面（一个HTML基本文件加上10个图片）可以采用单个持续的TCP连接进行传送，有时甚至可以在一个TCP连接完成多个Web页面传输。一般来说，如果一个连接经过一定时间间隔仍未被使用，HTTP服务器便关闭此连接。HTTP的默认模式是使用带流水线的持续连接。

   



#### HTTP报文格式

##### HTTP请求报文

![image-20210102121830159](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210102121830159.png)

每行以一个回车和换行符结束，最后一行后再附加一个回车和一个换行符。

HTTP请求报文第一行叫做请求行，其后继的行叫做首部行。



请求行：

方法字段 URL字段 HTTP版本字段

方法：GET,POST,HEAD,PUR,DELETE,大部分使用GET方法。

URL标识：请求对象/somedir/page.html。

版本字段：浏览器实现的是HTTP/1.1版本



首部行：

Host：www.someschool.edu 指明对象所在的主机 此信息为Web高速代理缓存所要求的。

Connection：close  浏览器告诉服务器不希望麻烦的使用持续连接，它要求服务器在发送完请求资源对象后就关闭这条连接。

User-agent：指明用户代理，即向服务器发送请求的浏览器的类型。服务器可以有效地为不同类型的用户代理实际发送相同对象的不同版本。（每个版本都有相同的YRL寻址）

Accept-language：表示用户想得到该对象的语言版本，此处为法语。



HTTP请求报文通用格式

![image-20210102122920683](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210102122920683.png)



实体体：使用GET方法时实体体为空，使用POST方法时才会提供该实体体。

表单情景，就是问卷调查那种：使用POST方法请求Web页面，Web页面的特定内容依赖于用户在表单字段输入的内容。实体体包含的便是用户在表单字段中的输入值。

![image-20210102123315176](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210102123315176.png)





##### HTTP响应报文

HTTP/1.1 200 OK

Connection：close

Date：Tue，09 Aug 2011 15：44：04 GMT

Server：Apache/2.2.3 （Centos）

Last-Modified：Tue，09 Aug 2011 15：11：03 GMT

Content-Length：6812

Content-Type：text/html



（data data data data data data...)



响应体三部分

协议版本字段，状态码，相应状态信息



状态行：



首部行：

Connection：close 发送报文后将关闭TCP连接

Date： 首部行指示服务器产生并发送该响应报文的日期和时间

Server： 首部行指示该报文是由...服务器产生的，类似于User-agent

LastModefied：首部行指示了对象创建或修改的最后日期和时间

Content-Length：首部行指示被发送对象中的字节数。

Content-Type：指示实体体的对象是什么格式

实体体：





![image-20210102151810326](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210102151810326.png)





![image-20210102151847687](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210102151847687.png)

![image-20210102153648700](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210102153648700.png)

返回报文



浏览器产生的首部行与很多因素有关，包括浏览器的类型和协议版本，浏览器的用户配置

Web服务器在产品，版本，配置上都有差异，这些都会影响响应报文的首部行



#### Cookie

HTTP是一个无状态协议，然而一个Web站点通常希望能够识别用户。为此HTTP中使用了cookie

cookie四组件：

1.HTTP响应报文中有一个cookie首部行，见上

2.HTTP请求报文中有一个cookie首部行

3.用户端系统中保留有一个cookie文件，并由用户的浏览器进行管管理

4.位于Web站点的一个后端数据库

实例情景：

![image-20210102155416287](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210102155416287.png)

![image-20210102155431472](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210102155431472.png)

![image-20210102155633871](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210102155633871.png)

![image-20210102155701015](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210102155701015.png)



### Web缓存

Web缓存器也叫代理服务器，它是能够代表Web服务器满足HTTP请求的网络实体。Web缓存器有自己的磁盘存储空间，并在存储空间保存最近被请求过的对象的副本

![image-20210102160151050](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210102160151050.png)

Web缓存器常由ISP购买并安装。

Web缓存器作用：

1.大大减少对客户请求的响应时间，特别是客户与Web缓存器之间的瓶颈带宽远高于客户与初始服务器的瓶颈带宽。

实例见P75-	P76

2.减少一个机构的接入链路到因特网的通信量，从而降低成本并增加带宽。



##### 条件GET方法

从上方引入：若Web缓存其中存储的副本为旧副本，则需要条件GET方法。

条件GET：

1.请求报文使用GET方法

2.请求报文中包含“If-Modified-Since”首部行。当请求对象在指定日期后产生过修改才服务器才发送该对象，否则发送一个实体体为空的响应报文。

![image-20210102162121505](C:\Users\Jonny\AppData\Roaming\Typora\typora-user-images\image-20210102162121505.png)