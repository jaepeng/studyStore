# TCP、IP、UDP

[参考：一篇文章看明白 TCP/IP，TCP，UDP，IP，Socket](https://blog.csdn.net/freekiteyu/article/details/72236734)

参考：图解TCP/IP

## TCP/IP

通常所说的TCP/IP指的是利用IP进行通信时所必须用到的协议群的统称。

<img src="https://gitee.com/pengjae/pic/raw/master/img/20201126195606.png" alt="image-20201126195605993" style="zoom: 50%;" />

## TCP/IP与OSI参考模型



![image-20201126215137936](https://gitee.com/pengjae/pic/raw/master/img/20201126215138.png)

### 互联网层（网络层）

1. **使用IP协议**——将分组数据包发送到目的主机
2. IP协议基于IP地址转发分包数据
3. TCP/IP分层中的**互联网层**与**传输层**的功能通常由操作系统提供。尤其是路由器，它必须得实现通过互联网层转发分组数据包的功能
4. 连接互联网的主机跟路由器必须都实现IP的功能

![image-20201126215915704](https://gitee.com/pengjae/pic/raw/master/img/20201126215915.png)

### 传输层

TCP/IP协议的传输层有两个具有代表性的协议。该层功能与OSI参考模型中的传输层类似。

传输层的最主要功能能就是能够让应用程序之间实现通信。

#### TCP

![image-20201127121959834](https://gitee.com/pengjae/pic/raw/master/img/20201127122006.png)

#### UDP

![image-20201127122018347](https://gitee.com/pengjae/pic/raw/master/img/20201127122027.png)

### 应用层（会话层以上的分层）

TCP/IP的分层中，将OSI参考模型中的会话层，表示层和应用层的功能都集中到了应用程序中实现。

- 服务端：提供服务的程序
- 客户端：接受服务的程序

浏览器与服务端之间通信所用的协议是HTTP，所传输数据的主要格式是HTML

Http是属于OSI应用层的协议，HTML属于表示层的协议。

### 数据包首部

![image-20201127122725248](https://gitee.com/pengjae/pic/raw/master/img/20201127122725.png)

#### 包首部：

![image-20201127123147165](https://gitee.com/pengjae/pic/raw/master/img/20201127123147.png)

**一些定义：**

+ 包:全能性述语
+ 帧：数据链路层中包的单位。
+ 数据报：IP和UDP等网络层以上的分层中包的单位。
+ 段：TCP数据流中的信息
+ 消息：应用协议中数据的单位

例子：

![image-20201127124018727](https://gitee.com/pengjae/pic/raw/master/img/20201127124018.png)

## TCP与UDP

IP首部中有一个协议字段用来标识网络层（IP）的上一层所采用的是哪一种传输层协议。根据这个字段的协议号，就可以识别IP传输的数据部分究竟是TCP，还是UDP。

TCPUDP为了识别自己所发送的数据部分究竟发给谁，也设定这么一个编号。为了指出这个具体的应用程序，就使用端口号这么一种识别码。根据端口号就可以识别在传输层上一层的应用层中进行处理的具体程序。

​	

### UDP

#### UDP特点

1. 不提供复杂的控制机制，利用IP提供面向无连接的通信服务
2. UDP协议实现了端口，从而让数据包可以在送到IP地址的基础上，进一步可以送到某个端口。
3. 当应用程序**对传输的可靠性要求不高**，但是对**传输速度**和**延迟要求较高**时，可以用UDP协议来替代TCP协议在传输层控制数据的转发。
4. 本身不负责重发。甚至当出现包的到达顺序乱掉时也没有纠正功能，需要在应用程序中去纠正。

![image-20201127141909665](https://gitee.com/pengjae/pic/raw/master/img/20201127141909.png)

### 适用方面

![image-20201127135626957](https://gitee.com/pengjae/pic/raw/master/img/20201127135626.png)

##### **如果使用UDP丢包该怎么办？**

- 为什么会丢包？

1. 缓存太小（默认8k）不能及时接受数据

   1. 连续多个UDP包超过UDP包接受缓冲区大小：

      i. UDP包过大

      ii.UDP发包速率过快，突发大户据流量超过了缓冲区上线

2. recvfrom()接受数据之后处理太慢

   1. 如果数据接收和处理是连续进行的，那么可能由数据处理过慢，两次recvfrom调用的时间间隔里发过来的包丢失。

- 解决方案：

  1. UDP包过大

     解决办法：增加系统发送接受缓冲区大小

     ```java
     int nBuf=32*1024;//设置为32K  
     setsockopt(s,SOL_SOCKET,SO_RCVBUF,(const char*)&nBuf,sizeof(int));
     setsockopt(s,SOL_SOCKET,SO_SNDBUF,(const char*)&nBuf,sizeof(int));
     ```

  2. 发包速率过快

     ​	解决办法：增加应答机制，处理完一个包后，在继续发包

  3. recvfrom()接收到数据之后处理速度太慢

     ​	解决办法：服务器程序启动之初，开辟两个线程，一个线程专门用于接收数据包，并存放在应用层的缓存区；另外一个线程用于专门处理和响应数据包请求，避免因为处理数据造成数据丢包。其本质上还是增大了缓冲区大小，只是将系统缓冲区转移到了自己的缓冲区。

  4. 最复杂的解决方式：

     在应用层实现丢包**重发机制**和**超时机制**，确保数据包不丢失。

##### UDP实现可靠传输

[如何用UDP实现可靠传输](https://blog.csdn.net/niumengting/article/details/90262100?utm_medium=distribute.pc_relevant_t0.none-task-blog-searchFromBaidu-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-searchFromBaidu-1.control)

​	客户用UDP给服务器传送数据的数据丢失，使得服务器端的recvfrom函数始终处于阻塞模式

从应用层考虑，关键是以下两点：（模仿TCP）

1. 提供超时重传，避免数据报丢失
2. 提供确认序列号，可以对数据报进行确认和排序。

Windows操作系统下建议用MFC的CAsyncSocket类，它是个异步类，不会发生阻塞。

**已经实现的可靠UDP:**

1. RUDP 可靠数据报传输协议；
2. RTP 实时传输协议
   1. 为数据提供了具有实时特征的端对端传送服务；
   2.   Eg：组播或单播网络服务下的交互式视频、音频或模拟数据

3. UDT
   +  基于UDP的数据传输协议，是一种互联网传输协议；
   +  主要目的是支持高速广域网上的海量数据传输，引入了新的拥塞控制和数据可靠性控制机制（互联网上的标准数据传输协议TCP在高带宽长距离的网络上性能很差）；
   +    UDT是面向连接的双向的应用层协议，同时支持可靠的数据流传输和部分可靠的数据报服务；
   +  应用：高速数据传输，点到点技术(P2P)，防火墙穿透，多媒体数据传输；

#### 有TCP了为什么还要让UDP实现可靠传输？

1. TCP效率过低，传统TCP的AIMD算法很难充分利用带宽。
2. 公平性问题: 由于AIMD算法与RTT是相关的, 因此不可避免的存在不公平性, 即RTT小的链接会抢占更多的带宽.
   1.  tcp的性能随着丢包率和RTT的增大, 急剧的下降.
   2. 改进方法
      1. 修改AIMD算法,  使用大一点的增长参数, 小一点的减少参数. 例如: HighSpeed, Scalable, BiC, FAST, H-TCP, L-TCP. 但是这个难以部署, 开源的linux, 也要重新编译才能部署
      2.  Parallel TCP, Rate-based reliable UDP, 主要用于私有网络, 其作用是最大可能的利用带宽, 实际使用时, 参数需要根据实际的网络进行调整, 适用场景较少.
   3. UDT等基于UDP的可靠传输优点：
      1. 应用程序级别, 方便使用, 基于udp实现可靠传输，能充分利用带宽
      2. udt使用新的拥塞控制算法, 更好实现公平性.





### TCP

#### TCP特点

1. 充分地实现了数据传输时各种控制功能，可以进行丢包时的重发控制，还可以对次序乱掉的分包进行顺序控制。
2. 面向有连接的协议，只有在确认通信对端存在时才会发送数据，从而控制通信流量的浪费。
3. ![image-20201127141723376](https://gitee.com/pengjae/pic/raw/master/img/20201127141723.png)
4. 以段为单位发送数据包

![image-20201127142014028](https://gitee.com/pengjae/pic/raw/master/img/20201127142014.png)

####  TCP如何保证可靠性

> TCP通过检验和，序列号（丢弃重复数据包8位字节编号），确认应答（确认收到数据包），重发控制，连接管理以及窗口控制等机制实现可靠性传输。

如何确认重发超时时间：

1. 高速的LAN中时间短一些，在长距离的通信当中，应该比LAN长一些。
2. 根据不同时段的网络拥堵程度时间的长短也会发生变化。

TCP是传输可靠，在传输层的数据传输可靠，但是**在应用层还需要校验**，不然可能也有问题。

tcp协议数据要**封装成IP数据包传输**，tcp是可靠传输，而IP是尽力而为传输。





### 两者的不同

1、TCP 是**面向连接**的传输控制协议，而UDP 提供了**无连接**的数据报服务；

2、TCP 具有**高可靠性**，确保传输数据的正确性，不出现丢失或乱序；UDP 在传输数据前**不建立连接**，不对数据报进行检查与修改，无须等待对方的应答，所以会出现分组丢失、重复、乱序，应用程序需要负责传输可靠性方面的所有工作；

3、UDP 具有较好的**实时性**，工作效率较 TCP 协议高；

4、UDP 段结构比 TCP 的段结构简单，因此网络**开销也小**。









## Socket 套接字

![image-20201127130153102](https://gitee.com/pengjae/pic/raw/master/img/20201127130153.png)

应用程序利用套接字，可以设置对端的 IP 地址，端口号，并实现数据的发送与接收。



