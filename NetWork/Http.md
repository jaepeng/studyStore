# Http

[参考链接]([你每天都在使用的HTTP协议，到底是什么鬼？ (qq.com)](https://mp.weixin.qq.com/s/AK1Pb9rx0q5Hf8dq6HNOhw))

拓展点：

1. [当网页里输入网址，回车后会发生什么](#writeURL)
2. [为什么要三次握手](#way3Hand)
3. [为什么要四次挥手](#why4Bye)
4. [加密方式](#encode)



## 基本介绍

1. Http协议用于客户端和服务器端之间的通信(还有TCP/IP协议等众多协议)
2. Client客户端请求Server服务端，Server服务端响应给Client客户端,浏览器或其他任何客户端都可以用HTTP协议的，**通过URL地址向HTTP的服务器即Web服务器发送所有请求**，Web服务器端在接收到请求后会做出反应，响应给对方，就是向客户端回传响应的信息。
3. 一端必是客户端,另一端必是服务器端,单由一条通信线路来讲,是确定的
4. .由HTTP协议就可以区分谁是客户端，和谁是服务器端了。
5. 通过发送请求和回应请求达成通信

## <span id="httpPoint">特点</span>

1. 支持客户端、服务器端模式，简单快速，客户端向服务器端请求服务时，只需传送请求方法和路径，灵活，HTTP允许传输任意类型的数据对象.
   1. **无连接 [每次服务器在处理完客户端的请求后，并收到客户的应答后，就断开了通信，当客户端再次发送请求时就是一个新的连接，采用这种方式可以节省传输时间。]**，限制每次连接只处理一个请求
   2. **无状态 [每次HTTP请求都是独立的，任何两个请求之间没有必然的联系]**,HTTP协议是无状态协议，**指明协议对于事务处理没有记忆能力**,。
2. HTTP都是由**客户端发起请求**的，并且**由服务器端回应响应消息**的。

<img src="https://gitee.com/pengjae/pic/raw/master/img/20201126100609.jpeg" alt="这里写图片描述" style="zoom:80%;float:left" />

### 输入网址回车：

<span id="writeURL">**客户端输入URL回车**</span>，DNS解析域名得到服务器的IP地址，服务器在80端口监听客户端请求，端口通过TCP/IP协议（可以通过Socket实现）通过三次握手建立TCP连接，向服务器发送HTTP Request请求，并得到服务器的Response响应；最后浏览器根据响应结果渲染输出页面。HTTP属于TCP/IP模型中的应用层协议，所以通信的过程其实是对应数据的入栈和出栈。

<img src="https://gitee.com/pengjae/pic/raw/master/img/20201126101142.jpeg" alt="这里写图片描述" style="zoom:80%;float:left" />

**报文从运用层传送到运输层，运输层通过TCP三次握手和服务器建立连接，四次挥手释放连接。**

[TCP的三次握手与四次挥手理解及面试题（很全面） - 李卓航 - 博客园 (cnblogs.com)](https://www.cnblogs.com/bj-mr-li/p/11106390.html)

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126103257.png)

<span id="way3Hand">**为什么需要三次握手呢？**</span>为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误。

例如：client发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间的滞留了，以致延误到连接释放以后的某个时间才到达server。本来这是一个早已失效的报文段，但是server收到此失效的连接请求报文段后，就误认为是client再次发出的一个新的连接请求，于是就向client发出确认报文段，同意建立连接。假设不采用“三次握手”，那么只要server发出确认，新的连接就建立了，由于client并没有发出建立连接的请求，因此不会理睬server的确认，也不会向server发送数据，但server却以为新的运输连接已经建立，并一直等待client发来数据。所以没有采用“三次握手”，这种情况下server的很多资源就白白浪费掉了。

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126103321.png)

<span id="why4Bye">**为什么需要四次挥手呢？**</span>TCP是**全双工模式**，当client发出FIN报文段时，只是表示client已经没有数据要发送了，client告诉server，它的数据已经全部发送完毕了；但是，这个时候client还是可以接受来server的数据；当server返回ACK报文段时，表示它已经知道client没有数据发送了，但是server还是可以发送数据到client的；当server也发送了FIN报文段时，这个时候就表示server也没有数据要发送了，就会告诉client，我也没有数据要发送了，如果收到client确认报文段，之后彼此就会愉快的中断这次TCP连接。



### Http/1.0缺点

1. 每个TCP连接只能发送一个请求，发送数据完毕后，连接就关闭了，如果还要请求就必须要新建一个请求连接。（无法复用连接）
2. HTTP是一种不保存状态，无状态协议，协议对于发送过来的请求或是响应都不做持久化处理。
3. 其次就是队头阻塞（`head of line blocking`）。由于`HTTP1.0`规定**下一个请求必须在前一个请求响应到达之前才能发送**。假设前一个请求响应一直不到达，那么下一个请求就不发送，同样的后面的请求也给阻塞了。

### Http1.1优势

1. HTTP1.1虽然是无状态协议，但是为了实现期望的保持状态功能，于是**引入了Cookie技术**，有了Cookie，和HTTP协议通信，就可以**管理状态**了。

   ![img](https://gitee.com/pengjae/pic/raw/master/img/20201126082934.webp)

   ![img](https://gitee.com/pengjae/pic/raw/master/img/20201126083032.webp)

   

   2. TCP连接的新建成本很高，因为需要客户端和服务器端三次握手。而**Http1.1可以让多个请求复用**

      [个人总结](https://github.com/jaepeng/studyStore/blob/master/NetWork/TCP%E3%80%81IP%E3%80%81UDP.md)

      [TCP/UDP描述]([面试官问我：一个 TCP 连接可以发多少个 HTTP 请求？我竟然回答不上来..._零度源码-CSDN博客](https://blog.csdn.net/mK0vouYv4BwgX190fSd/article/details/99905715?utm_medium=distribute.pc_relevant_bbs_down.none-task-blog-baidujs-1.nonecase&depth_1-utm_source=distribute.pc_relevant_bbs_down.none-task-blog-baidujs-1.nonecase))

      [TCP/HTTP关联,Http1.0/1.1/2.0区别]([TCP、HTTP的概念、关联，HTTP1.0/1.1/2.0的区别_xmd415606062的博客-CSDN博客](https://blog.csdn.net/xmd415606062/article/details/104553128/))
   
      1. 流程概述:客户端发起连接，客户端发起请求，服务器端响应请求，服务器端关闭连接。
      2. htpp1.0是一个服务器发送完一个Http响应后就断TCP连接了.每次请求都会重新建立和断开TCP连接,代价极大
   3. 某些服务器对 Connection: keep-alive 的 Header 进行了支持。意思是说，完成这个 HTTP 请求之后，**不要断开 HTTP 请求使用的 TCP 连接**。这样的好处是连接可以被重新使用，之后发送 HTTP 请求的时候不需要重新建立 TCP 连接，以及如果维持连接，那么 SSL 的开销也可以避免
      4. **Http1.1**可以被多个请求复用，只有在一段时间内，没有请求，就可以自动关闭。

   3. 既然维持 TCP 连接好处这么多，**HTTP/1.1** 就把 Connection 头写进标准，并且**默认开启持久连接**，除非请求中写明 Connection: close，那么浏览器和服务器之间是会维持一段时间的 TCP 连接，不会一个请求结束就断掉。[详细说明](https://tools.ietf.org/html/rfc2616#section-8.1)

   4. `HTTP1.1`还加入了**缓存处理（强缓存和协商缓存[[传送门](http://www.yangzicong.com/article/12)]）**新的字段如`cache-control`，支持**断点传输**，以及增加了**Host字段**（使得一个服务器能够用来创建多个Web站点）。

   5. Http1.1版还引入了 **<span id="pipeDes">管道机制（pipelining）</span>**，即在同一个TCP连接里面，客户端可以同时发送多个请求。这样就进一步改进了HTTP协议的效率,**原理**：**服务器必须按照客户端请求的先后顺序依次回送相应的结果，以保证客户端能够区分出每次请求的响应内容。也就是说，`HTTP`管道化可以让我们把先进先出队列从客户端（请求队列）迁移到服务端（响应队列）。**。**（但是默认不开启,原因如下）**
   
      1. 一些代理服务器不能正确的处理 HTTP Pipelining。
   2. 正确的流水线实现是复杂的。
      3. Head-of-line Blocking 连接头阻塞：在建立起一个 TCP 连接之后，假设客户端在这个连接连续向服务器发送了几个请求。按照标准，服务器应该按照收到请求的顺序返回结果，假设服务器在处理首个请求时花费了大量时间，那么后面所有的请求都需要等着首个请求结束才能响应。
   
   6. Http1.1下，现在的浏览器允许我们建立多个TCP连接，具体数量看浏览器定义，Chrome支持6个。对同域下并行加载6~8个资源

## HTTP2.0

[参考连接]([HTTP1.0 HTTP1.1 HTTP2.0 主要特性对比_weixin_34235371的博客-CSDN博客](https://blog.csdn.net/weixin_34235371/article/details/88910200?utm_medium=distribute.pc_relevant_t0.none-task-blog-searchFromBaidu-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-searchFromBaidu-1.control))

1. `HTTP2.0`通过在应用层和传输层之间增加一个二进制分帧层，突破了`HTTP1.1`的性能限制、改进传输性能。

   ![img](https://gitee.com/pengjae/pic/raw/master/img/20201126092534.jpeg)

2. 简单来说，`HTTP2.0`只是把原来`HTTP1.x`的`header`和`body`部分用`frame`重新封装了一层而已。

3. **多路复用（连接共享）**

   1. 下面是几个概念：
      - 流（`stream`）：都有一个唯一标识符和可选的优先级信息，用于承载双向信息。
      - 消息：与逻辑消息对应的完整的一系列数据帧。
      - 帧（`frame`）：`HTTP2.0`通信的最小单位，每个帧包含帧头部，至少也会标识出当前帧所属的流（`stream id`）。消息重组的依据就靠帧的标识作用

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126092731.jpeg)

      2.  从图中可见，所有的`HTTP2.0`通信都在**一个`TCP`连接上完成**，这个连接可以承载任意数量的**双向数据流**。
      1. 每个数据流以消息的形式发送，而消息由一或多个帧组成。这些帧可以乱序发送，然后再根据每个帧头部的流标识符（`stream id`）重新组装。
      2. 多路复用（连接共享）可能会导致关键请求被阻塞。`HTTP2.0`里每个数据流都可以设置优先级和依赖，优先级高的数据流会被服务器优先处理和返回给客户端，数据流还可以依赖其他的子数据流。
      3. HTTP2.0`实现了真正的并行传输，它能够在一个`TCP`上进行任意数量`HTTP`请求。而这个强大的功能则是基于“二进制分帧”的特性。

4. **头部压缩**
   1. 在`HTTP1.x`中，头部元数据都是以纯文本的形式发送的，通常会给每个请求增加500~800字节的负荷。比如说`cookie`，默认情况下，浏览器会在每次请求的时候，把`cookie`附在`header`上面发送给服务器。（由于`cookie`比较大且每次都重复发送，一般不存储信息，只是用来做状态记录和身份认证）
   2. `HTTP2.0`使用`encoder`来减少需要传输的`header`大小，通讯双方各自`cache`一份`header fields`表，既避免了重复`header`的传输，又减小了需要传输的大小。高效的压缩算法可以很大的压缩`header`，减少发送包的数量从而降低延迟。
5. **服务器推送**
   1. 服务器除了对最初请求的响应外，服务器还可以额外的向客户端推送资源，而无需客户端明确的请求。
   2. 意思是说客户端请求server后，server会把一些相关的客户需要的数据也一并传输过去，免得客户端再次建立连接请求server。比如一些静态文件，这种方式就非常适合。
   3. 例如客户端请求 page.html 页面，服务端就把 script.js 和 style.css 等与之相关的资源一起发给客户端。

##  HTTP的消息结构

### 请求消息结构:

1. 一个请求消息是由**请求行，请求头字段，一个空行和消息主体**构成。

2. **消息主体**是响应消息的承载数据。

**客户端：**发送请求

​	客户端发送给某个HTTP服务器端的请求报文中的内容

```
GET/HTTP/1.1
Host: hackr.jp
```

**服务器：**发送响应

```
HTTP/1.1 200 OK
Date: Tue, 10 Jul ...
Content.Length: 362
Content.Type: text/html
<html>
...
```

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126084905.webp)



> GET，Request Method，请求方法，Request URL，为请求的url的地址，Status Code为状态码，Remote Address为地址。

3. HTTP是基于TCP/IP协议的应用层协议，不涉及数据包传输，规定了客户端和服务器端之间的通信方式，默认使用80端口，就如同他俩交流的语言。
4. HTTP1.0的发布，任何格式的内容都可以发送了，不仅可以发送文件，图片，视频，二进制文件等。

**HTTP发送请求的例子：**

<img src="https://gitee.com/pengjae/pic/raw/master/img/20201126085035.webp" alt="img" style="float:left;" />

**服务器回应消息格式：**

<img src="https://gitee.com/pengjae/pic/raw/master/img/20201126085120.png" alt="img" style="float:left;" />

**响应头：**

Server为服务器的名称，Location为通知客户端新的资源位置，Content-Type响应数据的类型，Content-Encoding为响应数据的编码格式。

<img src="https://gitee.com/pengjae/pic/raw/master/img/20201126085240.webp" alt="img" style="float:left;" />

**Content-Type的字段值：**

```
text/plain
text/html
text/css
image/jpeg
// 上面的图片返回的是
image/png
image/svg+xml
audio/mp4
video/mp4
application/javascript
application/pdf
application/zip
application/atom+xml
```

**Accept字段声明自己可以接受哪些数据格式：**

```
Accept: */*
```

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126085453.webp)

请求行为请求消息的第一行，它说明了请求方法，**资源标示符**，HTTP版本

1. 请求URI定位资源：HTTP协议使用URI定位互联网上的资源。

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126085754.jpeg)

2. URI,URL,URN是用来识别，定位和命名互联网上的资源。

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126085821.webp)

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126085830.png)

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126085837.webp)

```
URI：
Uniform Resource Identifier，统一资源标识符
URL：
Uniform Resource Locator，统一资源定位符
URN：
Uniform Resource Name，统一资源名称
```

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126085947.webp)

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126085954.png)

**请求报文**

请求报文是由**请求方法，请求URL，协议版本，可选的请求首部字段和内容实体**构成的。

 1. 消息报头

    ![img](https://gitee.com/pengjae/pic/raw/master/img/20201126090145.webp)

    2. 响应报文由协议版本，状态码，响应的首部字段，以及实体主体构成。

       ![img](https://gitee.com/pengjae/pic/raw/master/img/20201126090242.webp)

       

## Http1.1的请求方法

> 主要还是Get和Post

### Get和Post的区别

**GET 用于获取资源**

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126090421.webp)

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126090429.webp)



**POST：传输实体主体**  

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126090619.png)

**两者参数比较：**

1. GET 和 POST 的请求都能使用额外的参数，但是 GET 的参数是**以查询字符串出现在 URL** 中，而 POST 的参数存储在实体主体中。不能因为 POST 参数存储在实体主体中就认为它的安全性更高，因为照样可以通过一些抓包工具（Fiddler）查看。

2. 因为 URL 只支持 ASCII 码，因此 GET 的参数中如果存在中文等字符就需要先进行编码。例如 `中文` 会转换为 `%E4%B8%AD%E6%96%87`，而空格会转换为 `%20`。POST 参数支持标准字符集。

3. ```
   GET /test/demo_form.asp?name1=value1&name2=value2 HTTP/1.1
   
   POST /test/demo_form.asp HTTP/1.1
   Host: w3schools.com
   name1=value1&name2=value2
   ```

4. **GET 方法是安全的**，**而 POST 却不是**，因为 POST 的目的是传送实体主体内容，这个内容可能是用户上传的表单数据，上传成功之后，服务器可能把这个数据存储到数据库中，因此状态也就发生了改变。

**PUT：传输文件** 

PUT 方法用来传输文件。

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126090643.png)

```
 HEAD：获得报文首部 
DELETE 方法用来删除文件，是与 PUT 相反的方法。
OPTIONS：询问支持的方法 
OPTIONS 方法用来查询针对请求 URI 指定的资源支持的方法。 
```

**HTTP超文本传输协议是一个基于请求与响应模式的，无状态的，应用层的协议，常基于TCP的连接方式。**HTTP表示通过HTTP协议定位网络资源，host表示合法的Internet主机域名或者ip地址，port指定为端口号。

```
第一行：方法，URL，协议版本
第二行：请求首部字段
第三行：内容实体
```

**服务器端响应消息：**

状态行，消息报头，空行，响应正文，这是一个HTTP响应的响应消息。

## HTTP状态码

![img](https://gitee.com/pengjae/pic/raw/master/img/20201126091025.webp)

## Http1.0和Http1.1的区别

1. 长连接：HTTP1.0需要采用keep-alive参数来告知server建立一个长连接，而HTTP1.1默认是长连接。HTTP是基于TCP的应用连接，如果每次通讯都需要连接的话，那性能消耗太大了，因此最好建立一个长连接。
2. 节约带宽：HTTP1.1**支持只发送header信息（不需要带任何请求体）**，如果server认为client可以请求数据，则返回100，否则返回401。客户端如果收到100才把请求体发送给server。这样当服务器返回401时，client就可以不用发送请求体了，节约了带宽。另外，HTTP还支持传送内容的一部分。这样当client已经有了一部分的资源后，只需要跟server请求剩余的资源了。这是断点续传的基础。
3. host域：HTTP1.1的场景下，web server如tomcat是支持多个虚拟站点可以共享同一个ip和端口的。
4. **http1.1新特性**
   1. 默认是长连接
   2. 支持流水线
   3. 支持同时打开多个 TCP 连接
   4. 支持虚拟主机
   5. 新增状态码 100
   6. 支持分块传输编码
   7. 新增缓存处理指令 max-age

## **[HTTP](#httpPoint)和HTTPS的区别**

[HTTP和HTTPS协议，看一篇就够了_不一样的博客-CSDN博客_https](https://blog.csdn.net/xiaoming100001/article/details/81109617)

 HTTPS（全称：Hypertext Transfer Protocol over Secure Socket Layer），是以安全为目标的HTTP通道，简单讲是HTTP的安全版。即**HTTP下加入SSL层**，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。 它是一个URI scheme（抽象标识符体系），句法类同http:体系。用于安全的HTTP数据传输。https:URL表明它使用了HTTP，但HTTPS存在不同于HTTP的默认端口及一个加密/身份验证层**（在HTTP与TCP之间）**。这个系统的最初研发由网景公司进行，提供了身份验证与加密通讯方法，现在它被广泛用于万维网上安全敏感的通讯，例如交易支付方面。

### Https特点

### Http的安全问题：

- 使用明文进行通信，内容可能会被窃听；
- 不验证通信方的身份，通信方的身份有可能遭遇伪装；
- 无法证明报文的完整性，报文有可能遭篡改。

### <span id ="encode">加密方式：</span>

> 加密是指利用某个值（密钥）对明文的数据通过一定的算法变换成加密（密文）数据的过程。他的逆向过程叫做解密。

#### 非对称加密

**特点:**

1. 非对称密钥加密，又称公开密钥加密（Public-Key Encryption），加密和解密使用不同的密钥（公钥和私钥）。
2. 公开密钥所有人都可以获得，通信发送方获得接收方的公开密钥之后，就可以使用公开密钥进行加密，接收方收到通信内容后使用私有密钥解密。
3. 非对称密钥除了用来加密，还可以用来进行签名。因为私有密钥无法被其他人获取，因此通信发送方使用其私有密钥进行签名，通信接收方使用发送方的公开密钥对签名进行解密，就能判断这个签名是否正确。
4. 最大的挑战：如何传递安全的密钥

- 优点：可以更安全地将公开密钥传输给通信发送方；
- 缺点：运算速度慢。

![非对称](https://gitee.com/pengjae/pic/raw/master/img/20201126142909.png)

#### 对称加密（Symmetric-Key Encryption）：

 > 加密和解密使用同一密钥。

- 优点：运算速度快；
- 缺点：
  - 无法安全地将密钥传输给通信方。
  - 加密和解密上需要花费时间较长

![对称加密](https://gitee.com/pengjae/pic/raw/master/img/20201126142632.png)

#### HTTPS 采用的加密方式

上面提到对称密钥加密方式的传输效率更高，但是无法安全地将密钥 Secret Key 传输给通信方。而非对称密钥加密方式可以保证传输的安全性，因此我们可以**利用非对称密钥加密方式将 Secret Key 传输给通信方**。HTTPS 采用混合的加密机制，正是利用了上面提到的方案：

- 使用非对称密钥加密方式，传输对称密钥加密方式所需要的 Secret Key，从而保证安全性;
- 获取到 Secret Key 后，再使用对称密钥加密方式进行通信，从而保证效率。（下图中的 Session Key 就是 Secret Key）

### 基本特征

1. 基于HTTP协议，通过SSL或TLS提供加密处理数据、验证对方身份以及数据完整性保护
   1. HTTPS 并不是新协议，而是让 HTTP 先和 SSL（Secure Sockets Layer）通信，再由 SSL 和 TCP 通信，也就是说 HTTPS 使用了隧道进行通信。
   2. 通过使用 SSL，HTTPS 具有了加密（防窃听）、认证（防伪装）和完整性保护（防篡改）。
2. 内容加密：采用混合加密技术，中间者无法直接查看明文内容
3. 验证身份：通过证书认证客户端访问的是自己的服务器
4. 保护数据完整性：防止传输的内容被中间人冒充或者篡改

``` 
混合加密：结合非对称加密和对称加密技术。客户端使用对称加密生成密钥对传输数据进行加密，然后使用非对称加密的公钥再对秘钥进行加密，所以网络上传输的数据是被秘钥加密的密文和用公钥加密后的秘密秘钥，因此即使被黑客截取，由于没有私钥，无法获取到加密明文的秘钥，便无法获取到明文数据。


数字摘要：通过单向hash函数对原文进行哈希，将需加密的明文“摘要”成一串固定长度(如128bit)的密文，不同的明文摘要成的密文其结果总是不相同，同样的明文其摘要必定一致，并且即使知道了摘要也不能反推出明文。


数字签名技术：数字签名建立在公钥加密体制基础上，是公钥加密技术的另一类应用。它把公钥加密技术和数字摘要结合起来，形成了实用的数字签名技术。
```

- 收方能够证实发送方的真实身份；
- 发送方事后不能否认所发送过的报文；
- 收方或非法者不能伪造、篡改报文。

**SSL建立连接过程**

![在这里插入图片描述](https://gitee.com/pengjae/pic/raw/master/img/20201126102647.png)

1. client向server发送请求https://baidu.com，然后连接到server的443端口，发送的信息主要是随机值1和客户端支持的加密算法。
2. server接收到信息之后给予client响应握手信息，包括随机值2和匹配好的协商加密算法，这个加密算法一定是client发送给server加密算法的子集。
3. 随即server给client发送第二个响应报文是数字证书。服务端必须要有一套数字证书，可以自己制作，也可以向组织申请。区别就是自己颁发的证书需要客户端验证通过，才可以继续访问，而使用受信任的公司申请的证书则不会弹出提示页面，这套证书其实就是一对公钥和私钥。传送证书，这个证书其实就是公钥，只是包含了很多信息，如证书的颁发机构，过期时间、服务端的公钥，第三方证书认证机构(CA)的签名，服务端的域名信息等内容。
4. 客户端解析证书，这部分工作是由客户端的TLS来完成的，首先会验证公钥是否有效，比如颁发机构，过期时间等等，如果发现异常，则会弹出一个警告框，提示证书存在问题。如果证书没有问题，那么就生成一个随即值（预主秘钥）。
5. 客户端认证证书通过之后，接下来是通过随机值1、随机值2和预主秘钥组装会话秘钥。然后通过证书的公钥加密会话秘钥。
6. 传送加密信息，这部分传送的是用证书加密后的会话秘钥，目的就是让服务端使用秘钥解密得到随机值1、随机值2和预主秘钥。
7. 服务端解密得到随机值1、随机值2和预主秘钥，然后组装会话秘钥，跟客户端会话秘钥相同。
8. 客户端通过会话秘钥加密一条消息发送给服务端，主要验证服务端是否正常接受客户端加密的消息。
9. 同样服务端也会通过会话秘钥加密一条消息回传给客户端，如果客户端能够正常接受的话表明SSL层连接建立完成了。

**数字证书内容**

包括了加密后服务器的公钥、权威机构的信息、服务器域名，还有经过CA私钥签名之后的证书内容（经过先通过Hash函数计算得到证书数字摘要，然后用权威机构私钥加密数字摘要得到数字签名)，签名计算方法以及证书对应的域名。

**验证证书安全性过程**

1. 当客户端收到这个证书之后，使用本地配置的权威机构的公钥对证书进行解密得到服务端的公钥和证书的数字签名，数字签名经过CA公钥解密得到证书信息摘要。
2. 然后证书签名的方法计算一下当前证书的信息摘要，与收到的信息摘要作对比，如果一样，表示证书一定是服务器下发的，没有被中间人篡改过。因为中间人虽然有权威机构的公钥，能够解析证书内容并篡改，但是篡改完成之后中间人需要将证书重新加密，但是中间人没有权威机构的私钥，无法加密，强行加密只会导致客户端无法解密，如果中间人强行乱修改证书，就会导致证书内容和证书签名不匹配。

**那第三方攻击者能否让自己的证书显示出来的信息也是服务端呢？**

伪装服务端一样的配置）显然这个是不行的，因为当第三方攻击者去CA那边寻求认证的时候CA会要求其提供例如域名的whois信息、域名管理邮箱等证明你是服务端域名的拥有者，而第三方攻击者是无法提供这些信息所以他就是无法骗CA他拥有属于服务端的域名。

### Https问题

**1.怎么保证保证服务器给客户端下发的公钥是真正的公钥，而不是中间人伪造的公钥呢？**

![这里写图片描述](https://gitee.com/pengjae/pic/raw/master/img/20201126102815.png)

2. **证书如何安全传输，被掉包了怎么办？**

   ![](https://gitee.com/pengjae/pic/raw/master/img/20201126102832.png)

   

## 其他一些提问

1. 现代浏览器在与服务器建立了一个 TCP 连接后是否会在一个 HTTP 请求完成后断开？什么情况下会断开？
   1. 一个服务器在发送完一个 HTTP 响应后，会断开 TCP 链接。
   2. 所以虽然标准中没有设定，某些服务器对 Connection: keep-alive 的 Header 进行了支持。可以通过这个是的TCP保持长连接
   3. 完成这个 HTTP 请求之后，不要断开 HTTP 请求使用的 TCP 连接。这样的好处是连接可以被重新使用，之后发送 HTTP 请求的时候不需要重新建立 TCP 连接，以及如果维持连接，那么 SSL 的开销也可以避免
2. 一个 TCP 连接可以对应几个 HTTP 请求？
   1. **如果维持连接**，一个 TCP 连接是可以发送多个 HTTP 请求的。
3. 一个 TCP 连接中 HTTP 请求发送可以一起发送么（比如一起发三个请求，再三个响应一起接收）？
   1. 单个 TCP 连接在同一时刻只能处理一个请求，意思是说：两个请求的生命周期不能重叠，任意两个 HTTP 请求从开始到结束的时间在同一个 TCP 连接里不能重叠。
   2. 虽然Http1.1引入了Pipelining 来试图解决这个问题，但是这个功能在浏览器中默认是关闭的。详细解释可以翻看[上面](#pipeDes)
4. HTTP/1.1 时代，浏览器是如何提高页面加载效率的呢？
   1. 维持和服务器已经建立的 TCP 连接，在同一连接上顺序处理多个请求。
   2. 和服务器建立多个 TCP 连接。
5. 浏览器对同一 Host 建立 TCP 连接到数量有没有限制？
   1. 有。Chrome 最多允许对同一个 Host 建立六个 TCP 连接。不同的浏览器有一些区别。

