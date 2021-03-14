## 概括

ref: [极客时间 - 趣谈网络协议 - 刘超](https://time.geekbang.org/column/article/7724)

只要是在网络上跑的包，都是完整的。可以有下层没上层，绝对不可能有上层没下层。例如对 tcp 协议来说，每一次有包发送出去，则必然有 ip 层和 mac 层，不然是不可能发出去的。

二层设备的定义：就是只把 mac 头摘下来，判断符不符合，应该是丢弃、转发还是交由上层；
三层设备：再把 ip 头摘下来，判断符不符合，应该是丢弃、转发还是交由上层。

### ip 地址
ipv4 被点分割为 4 个部分，每个部分 8 个比特；32 位的 ip 还被分为了 5 类，前一部分网络号后一部分主机号。

### CIDR 无类别域间路由
Classless Inter-Domain Routing，为了给用户分配 ip 地址以及有效地路由而提出的，对 ip 地址进行归类的方法。（基于分类的方法可扩充性不足）
10.100.122.2/24 就是一种表示形式，24 的意思是，在 32 位中，前 24 位是网络号，后 8 位是主机号。伴随其存在的还有广播地址： 10.100.122.255；与子网掩码 255.255.255.0。

### MAC 地址
Media Access Control Address，一个网络包要从一个地方传到另一个地方，除了要有确定的地址，要需要有定位功能。ip 才有定位属性，mac 更像是身份证，仅是一个唯一的标示。
设计目的：为了在组网的时候，不同的网卡在一个网络里时不会冲突；通信范围被限定在子网内。

### DHCP 动态主机配置协议
Dynamic Host Configuration Protocol，
工作图：https://docs.oracle.com/cd/E24847_01/html/819-7058/dhcp-overview-3.html

## 分层

### 第二层 - 数据链路层
数据链路层需要解决的，就是在解决往媒体上发数据的时候，谁先发谁后发、防止混乱等问题。即所做的就是媒体访问控制，这里会用到一个物理地址，也会叫做 mac 地址。

#### ARP 协议
已知 ip 地址，求 mac 地址段协议。
广而告之，发送一个广播包，谁是这个 ip 谁就来回答；为避免每次都是用 arp 请求，机器本地也会有缓存。

### icmp 协议
Internet Control Message Protocol，互联网控制报文协议。
分为主动请求与主动请求的应答，ping 就是一种主动请求并且获得应答。

## MAC 头 IP 头细节
```bash
		目标 mac 地址
		源 mac 地址
		协议类型
（mac 头）
	-------------------
（ip 头）
  版本 ｜ 首部长度 ｜ 服务类型TOS
  		总长度
  		 标识符
   标志位      片偏移
    TTL       协议
        首部校验和
        源 ip 地址
        目标 ip 地址
          选项
          数据
```

在 mac 头里：
- 协议类型：用来说明是 ip 协议；

在 ip 头里：
- 版本：目前还是 ipv4
- 首部长度：ip 首部的长度，一般为 20B；
- 服务类型（Type Of Service）：代表了当前包是高优先级还是低优先级；
- 总长度：数据包的总长度；
- 标识符：唯一表示主机发送的每一份数据报；
- 标志：分为 3 个字段，依次为保留位、不分片位和更多片位：
  - 保留位：一般被置为 0；
  - 不分片位：表示数据包是否被分片
  - 更多片位：除了最后一个分片，其他每个组成数据报的片都要将此设为 1；
- 片偏移：该分片相对于原始数据报开始处位置的偏移量；
- 生存时间：设置数据报可以经过的最多路由器数，它指定了数据报的生存时间。
- 协议：向 ip 层传输数据的协议类型，常见协议类型的值有
  - “0” ip
  - “1” icmp
  - “2” igmp
  - “6” tcp
  - “17” udp
- 首部校验和：值 ip 数据报包头的校验和，该数值用来检错，已确保包被正确无误的接收（仅报文头）；

在任意台机器上想要访问另一个 ip 的时候，首先会通过 cidr 和子网掩码来判断两机器是否在一个网段。
- 如果是：直接将源、目标地址放入 ip 头中，然后通过 arp 获取 mac 地址，同样放入 mac 头中。
- 如果不是：就需要发往默认网关 gateway，gateway 的地址一定是和源 ip 是一个网段的。
网关往往是一个路由器，是一个三层转发设备（三层设备就是，把 mac 头和 ip 头都取下来，根据里面的内容判断包需要往哪里发送。）

路由器是一台设备，它有五个网口或者网卡，分别连着五个局域网；每个网卡的 ip 地址都和局域网的 ip 地址段相同，也是那个局域网的网关。

### 路由转发，ip 头信息是否改变
mac 地址仅在局域网内有效，因此只要过网关 mac 地址就会改变。不改变 ip 地址的网关称为转发网关，改变 ip 地址的称为 NAT 网关。

整个转发网关的过程里，每到一个新的局域网 mac 都是要变的，但是 ip 地址都不变，里面不会保存任何网关的 ip 地址。所谓的下一跳，是需要将某个 ip 转为 mac 地址并放入 mac 头。

Nat Worknet Translation，是一种把内部私有网络地址翻译成合法网络 ip 地址的技术。

## wireshark 应用
https://www.cnblogs.com/linyfeng/p/9496126.html

## 传输层
Transmission Control Protocol；
传输层里两个重要的协议，tcp 和 udp。最常见的问题是：tcp 是面向连接的，udp 是面向无连接的。

**所谓的建立连接，是为了在客户端和服务端维护连接，而建立一定的数据结构来维护双方交互的状态，用这样的数据结构来保证所谓的面相连接的特性。**

例如：**tcp 提供可靠交付：**通过 tcp 连接传送的数据，无差错、不丢失、不重复、并且按顺序到达。**ip 包是没有任何可靠性保证的；**dup 继承了 ip 包的特性，无法保证不丢失、不保证按顺序到达。

例如：**tcp 是面向字节流的：**tcp 发送的时候是一个流，没头没尾。ip 是一个一个的包，字节流是 tcp 自己的状态维护做的事情。udp 是基于数据报的，一个一个的发一个一个的收。

例如：**tcp 是可以拥塞控制的：**tcp 会根据网络环境调整自己的发送速度。

例如：**tcp 是一个有状态服务：**里面清楚地记着发送和接收的成功或失败状态。

我们可以这样理解：mac 定义了本地局域网的传输行为，ip 层定义了真个网络端到端的传输行为。

### udp
ip 头里有一个 8 位协议，里面存放的数据将会指明是 tcp 还是 udp。包头展示：
```bash
  源端口号（16位）        目的端口号（16位）
  udp长度（16位）        udp校验和（16位）
                 数据
```

udp 特点：
1. 沟通简单：没有大量的数据结构、处理逻辑、包头字段；
2. 不会建立连接：虽然有端口号在监听，但是谁都可以给他传数据：
3. 不进行拥塞控制。

使用场景：
1. 网络情况较好的内网，或对于丢包不敏感的应用；
2. 不需要一对一沟通，而是可以广播的应用；udp 的不面向连接使得其可以承载广播协议，dhcp 就是一种；
3. 需要处理速度快，时延低、可以容忍少数丢包，即便是网络拥塞也毫不退让。

### tcp
tcp 包头：
```bash
  源端口号（16位）                                   目的端口号（16位）
                             序号（32位）
                            确认序号（32位）
首部长度  保留       URG、ACK、PSH、RST、SYN、FIN      窗口大小
 （4位） （6位）                                      （16位）
  校验和（16位）                                      紧急指针（16位）
                             选项
                             数据
```

- 序号：用以解决乱序的问题
- 确认序号：如果对方没有收到就应该重新发送，知道送达，用以解决不丢包的问题；ip 协议无法保证可靠，只能有 tcp 自己来保证可靠性；
- 状态位：例如 syn 是发起一个连接，ack 是回复，rst 是重新连接，fin 是结束连接；tcp 的面向连接，就是因为双方在维护连接的状态，这些带状态位的包的发送会引起双方的状态变更；
- 窗口大小：tcp 所做的流量控制，就是通信双方各声明一个窗口，标示自己当前的处理能力；

#### wait 等状态
 



## ipv6
[wiki](https://zh.wikipedia.org/wiki/IPv6)
NAT 与 CIDR 可以缓解 ipv4 匮乏的现象。
> ipv6 二进制下为 128 位长度，以 16 位为一组，每组以冒号 ':' 分开，可以分为 8 组，每组以 4 位十六进制方式表示。
例如：2001:0db8:86a3:08d3:1319:8a2e:0370:7344 就是一个合法的 ipv6 地址。

## 路由协议
[ref](https://community.fs.com/blog/ospf-vs-bgp-routing-protocol-choice.html#:~:text=The%20main%20difference%20between%20OSPF,routing%20operations%20performed%20between%20two)
> Both OSPF (Open Shortest Path First) and BGP (Border Gateway Protocol) are routing protocols that make routing decisions across the Internet. 

## JSON
[wiki](https://zh.wikipedia.org/wiki/JSON)
> JavaScript Object Notation，JavaScript 对象表示法。轻量级的资料交换语言，用来传输由属性值或序列性值（array）组成的数据对象。JSON 是独立于语言的文本格式。
1. 数值；
2. 字符串；
3. 布尔值；
4. array；
5. 对象 object；
6. null。

## protobuf
[https://developers.google.com/protocol-buffers](https://developers.google.com/protocol-buffers)
> Protocol buffers 是一种语言无关、平台无关的可扩展机制，用于序列化结构化数据——就像 xml 不过更小更快更简单。你仅定义一次数据被结构化的方法，就可以特殊的生成源码来从不同的数据流成读出或写入了。

## gprc
[wiki](https://zh.wikipedia.org/wiki/GRPC)
gRPC Remote Procedure Calls 是谷歌发起的一个开源远程过程调用系统，基于 HTTP/2 协议传输，使用 Protobuf 作为接口描述语言。
常见使用场景：
- 微服务框架下，多种语言服务之间的高效交互；

架构： client <-> gRPC Server <-> client

### Remote Procedure Call
[wiki](https://zh.wikipedia.org/wiki/%E9%81%A0%E7%A8%8B%E9%81%8E%E7%A8%8B%E8%AA%BF%E7%94%A8)
> 远程过程调用，是一个计算机的通信协议。该协议允许一台计算机上的程序调用另一个地址空间（commonly on another computer on a shared network）的子进程，而程序员就像调用本地程序一样，无需额外的为这个交互坐编程。也是一种 Client/Server 模式。

[ref- https://www.cnblogs.com/takumicx/p/10059448.html](https://www.cnblogs.com/takumicx/p/10059448.html)
> 对分布式系统而言，不同的服务可能分布在不同的节点之上，一个服务要完成自己的功能需要调用其他服务的接口，一种是采用 http 请求的方式，另一种是 rpc 的方式，让我们可以像调用本地接口一样使用远程服务。

## Transport Layer Security (TLS) 
[wiki](https://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E6%80%A7%E5%8D%94%E5%AE%9A)

[https://www.ruanyifeng.com/blog/2014/02/ssl_tls.html](https://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)

不使用 ssl/tls 的 http 通信都是非加密，直接使用明文进行通信的。
tls 希望达到：
1. 所有信息都是加密传播，第三方无法窃听；
2. 具有校验机制，一旦被篡改，通信双方都会发现；
3. 配备身份证书，防止身份被冒充。

1996 年 NetSpace 的 SSL 3.0 得到应用；99 年标准化组织发布 ssl 的升级 TLS 1.0 版本。

**基本思路：**TLS 采用公钥加密，也就是说客户端先要向服务器所要公钥，然后用公钥加密信息，服务器收到密文后用自己的私钥进行解密。
**要解决的两个问题：**
1. 如何保证公钥不被篡改？
  将公钥放到数字证书中，只要证书可信，公钥就可信。
  [wiki](https://zh.wikipedia.org/wiki/%E5%85%AC%E9%96%8B%E9%87%91%E9%91%B0%E8%AA%8D%E8%AD%89)Public key certificate，又称为 digital certificate.
2. 公钥加密（非对称）计算量大，如何减少耗时？
  每一次会话 session，客户端和服务端都会深成一个 session key，用它来加密信息，该对称加密运算非常快，而公钥只用来加密 session key 本身。







## tcp keepalive
[https://blog.csdn.net/lanyang123456/article/details/90578453](https://blog.csdn.net/lanyang123456/article/details/90578453)
tcp 是面向连接，当两端都没有数据的接收和发送时，如何判断连接是否正常呢？

```bash
~$ cat /proc/sys/net/ipv4/tcp_keepalive_time
7200
```

当两小时内此 socket 的任何一方都没有数据交换，tcp 就会自动发送存活检测探针，对方比相应，会出现下面三种情况：
1. 对方接收一切正常：以期望的 ack 相应，2 小时后将发送另一个存活探针；
2. 对方崩溃：以 RST 响应，socket 关闭；
3. 对方无任何响应：暂略。

## http 的幂等性
[https://www.cnblogs.com/weidagang2046/archive/2011/06/04/idempotence.html](https://www.cnblogs.com/weidagang2046/archive/2011/06/04/idempotence.html)

HTTP/1.1 规范中的幂等性定义：
> Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request.

幂等性是指一次或多次请求，应该具有相同的副作用。比如 GET 方法用于获取资源，不应有副作用，如：

```bash
GET http://examlpexxxxx.com/xxxx.html
```

GET 方法并不会改变资源的状态，不论调用一次还是调用 N 次都没有副作用。注意，这里提到的是调用一次和 N 次的副作用相同，而不是每次请求的结果相同。
GET 方法对一个资源的请求，可能会得到不同的结果，但本身没有产生副作用。

DELETE 方法用于删除资源，是有副作用的，但也应该满足幂等性。

```bash
DELETE http://www.examlpexxxxx.com/article/6666
```
如上调用一次或多次的副作用都是相同的，即删除掉 6666 的帖子，因此调用者可以执行多次而无需担心。

如下关注 POST 与 PUT 在幂等性上的区别：
> The POST method is used to request that the origin server accept the entity enclosed in the request as a new subordinate of the resource identified by the Request-URI in the Request-Line ...... If a resource has been created on the origin server, the response SHOULD be 201 (Created) and contain an entity which describes the status of the request and refers to the new resource, and a Location header.

POST 方法用于请求源服务器接收请求中包含的实体，作为请求行中标示的请求 URL 资源的新下属 。。。。


> The PUT method requests that the enclosed entity be stored under the supplied Request-URI. If the Request-URI refers to an already existing resource, the enclosed entity SHOULD be considered as a modified version of the one residing on the origin server. If the Request-URI does not point to an existing resource, and that URI is capable of being defined as a new resource by the requesting user agent, the origin server can create the resource with that URI.

### POST 与 GET
links:
1. https://www.cnblogs.com/hyddd/archive/2009/03/31/1426026.html
2. https://www.zhihu.com/question/28586791
3. https://stackoverflow.com/questions/630453/put-vs-post-in-rest
4. https://blog.csdn.net/mad1989/article/details/7918267

**对于 GET/POST 首先需要区分，是浏览器的使用场景，还是用 HTTP 作为接口传输协议的场景！**

**浏览器中的场景：**即从 HTML 和浏览器诞生开始就有的 http 协议中的 POST/GET，浏览器使用 GET 请求来获取一个 html 页面/图片/js 等资源；用 POST 来提交一个 <form> 表单，并得到一个结果网页。

GET 读取一个资源，反复的读取不会对访问的数据有副作用，这就是幂等性，因此也可在浏览器或 Server 端做缓存。

POST 会随着浏览器中 submit 元素的点击，而让服务器去做一件事，这件事往往是有副作用，不幂等的，因此无法做缓存。

GET 与 POST 携带数据的格式也有区别，浏览器中的 GET 只能由一个 url 所触发，所以 GET 之上的参数就只能依靠在 url 之上，但 http 协议本身并没有这个限制。

**在接口中：**也就是 Postman 发出来的 GET 和 POST 请求，此时的 G/P 请求不只能用在前后端的交互之中，还能用在各个子服务的调用之中，也就是当作一种 RPC 协议使用。
当 http 用作接口发送时就少了在浏览器当中的一些限制，只要符合 http 格式就可以发送：

```bash
<METHOD> <URL> HTTP/1.1\r\n
<HEADER1>: <HEADER1VALUE>\r\n
...
\r\n
<BODY DATA>
```

其中 method 可以是 GET 也可是 POST 或其他 http method，协议本身并没有限定 GET 一定不能没有 body，或 POST 不能将参数放置 url 中。虽然自由，但这就需要 client 与 server 端自行约定好。

于是一系列的接口规范/风格诞生了，其中名气最大的是 **REST**。
1. REST 约定 GET、POST、PUT、DELETE 分别用于获取、创建、替换和删除“资源”；
2. REST 最佳实践还推荐使用 json 作为请求体；

### REST
[wiki](https://zh.wikipedia.org/wiki/%E8%A1%A8%E7%8E%B0%E5%B1%82%E7%8A%B6%E6%80%81%E8%BD%AC%E6%8D%A2)

Representational State Transfer，表现层状态转换，是于 2000 年提出的万维网网络架构风格，**目的**是便于不同软件在网络中传递信息。
REST 是根据 http 之上而确定的一组约束和属性。符合 REST 的网络服务，允许客户端发出以 uri 访问或操作网络资源的请求。

REST 是**设计风格**，而不是标准：
1. 资源是由 uri 来指定；
2. 对资源的操作包括获取、创建、修改和删除，这正好对应 http 协议提供的 GET、POST、PUT 和 DELETE 方法；
3. 通过操作资源的表现形式来操作资源；
4. 资源的表现形式可以是 XML、HTML 或 JSON。

REST 架构风格最重要的 6 个**架构限制：**
1. Client/Server：将客户端和服务端的关注点分离；
2. 无状态 stateless：
  - 服务器不能保存客户端信息，每一次从客户端的请求要包含必须的状态信息，会话信息由客户端保存，服务端根据这些状态来处理请求；
  - 。。。
3. 缓存；
4. 接口统一：这是 REST 设计的基本出发点，减少了耦合性，可以让所有模块各自独立的进行改进；
5. 分层系统；
6. code-on-demand。

[https://www.ruanyifeng.com/blog/2011/09/restful.html](https://www.ruanyifeng.com/blog/2011/09/restful.html)
1. 资源 resource：就是网络上的一个实体，或者说网络上的一个具体信息；
2. 表现层 represention：我们把资源具体呈现出来的形式，叫做它的表现层（比如说文本可以用 txt、html、xml 或是 json 格式表现）；
3. 状态转化 state transfer：如果客户端想要操作服务器，必须通过某种手段，让服务器发生“状态转化“，而这种状态转换化是建立在表现层之上的，所以是表现层状态转化。

综上的解释：
1. 每一个 uri 代表一种资源；
2. 客户端和服务端之间传递这种资源的某种表现层；
3. 客户端通过四个 http 动词，对服务器进行操作，实现状态转化。




## http ua 简介
User-Agent 是 http header 的一部分，用于向所访问的网站提供你所使用的历览器类型及版本、操作系统类型及版本、浏览器内核等。通过该表示，网站将会像是不同的排版。


## http 1 与 2 的区别
todo



## tcp 为什么需要“三次握手”
[ref](https://draveness.me/whys-the-design-tcp-three-way-handshake/)
关注点是**三次握手**，而不是三次。
其中使用三次握手的首要原因是，为了阻止历史的重复连接初始化造成的混乱问题，防止使用了 tcp 协议的通信双方建立了错误的连接：

> The principle reason for the three-way handshake is to prevent old duplicate cononection initaitions from causing confusion.

想象如果通信双方的通信次数只有两次，那么发送发一旦发出建立连接的请求之后就无法撤回这次请求，如果在网络状况复杂或者较差的网络中，发送方连续发送多次建立连接的请求，当 tcp 建立连接只能通信两次的时候，那么发送方就只能选择接受、或是拒绝发送方的这次请求，而无法判断这个请求是否是一个过期请求。

所以 tcp 在使用三次握手来建立连接的时候引入了 res 控制信息，当接收方收到建立连接的请求时，会发送 seq+1 给对方，这时可由连接建立者来判断该连接是否已过期：

1. 如果是历史连接，即 seq 以过期或超时，那么建立连接方会直接发送 rst 来终止此次连接；
2. 如果不是历史连接，那么建立连接方会发送 ack 消息，通向双发连接建立成功。

**三次握手**和 rst 标志字段将是否建立连接的**最终控制权**交给了发送方，因为只有发送方才有足够的上下文来判断，当前连接是否已过期，这是 tcp 使用三次握手的最主要原因。

还有一个重要的原因是，**双方都需要获得一个发送信息的初始化序列号**，以作为一个可靠的传输层协议来保证数据包的缺失、乱序等问题。


## dns 解析过程
[https://time.geekbang.org/column/article/9895](https://time.geekbang.org/column/article/9895)
从根 dns 服务器到顶级域 dns 服务器，到权威 dns 服务器，形成树状的层次结构。
1. 根 dns 服务器：返回顶级域 dns 服务器的 ip 地址；
2. 顶级域 dns 服务器：返回权威 dns 服务器的 ip 地址；
3. 权威 dns 服务器：返回相应主机的 ip 地址。

为了提高 dns 性能，都会有多级缓存，解析流程如下[ref](https://github.com/skyline75489/what-happens-when-zh_CN#dns-%E6%9F%A5%E8%AF%A2)：
1. 电脑客户端请求例如 www.baidu.com 的 ip 地址，浏览器检查域名是否在缓存当中；
2. 如果浏览器的缓存没有命中，就去调用 gethostbyname 库函数进行查询；
3. gethostbyname 函数在试图进行 dns 解析之前，会先检查域名是否在 /etc/host 文件当中；
4. 如果 gethostbyname 没有缓存这个域名记录，也不再 host 文件中，他就会向本地 dns 服务器发送一条 dns 查询请求，通常到达网络服务商；
5. 如果该域名解析不在本地 dns 的缓存当中，那就会去询问根域名服务器，根域名服务器会返回顶级域 dns 服务器地址；
6. 顶级域 dns 服务器在返回 baidu.com 的权威 dns 服务器地址；
7. 权威 dns 服务器将对应主机的 ip 地址返回给本地 dns 服务器；
8. 本地 dns 再将结果放回给客户端。

从客户端的角度来看，是本地 dns 在全权效劳，客户端只需要等待结果。


## 当输入 url 回车后发生了什么
[https://github.com/skyline75489/what-happens-when-zh_CN#url](https://github.com/skyline75489/what-happens-when-zh_CN#url)

浏览器解析 url，获取协议以及所请求的资源；

DNS 查询：
- 如果 dns 服务器和我们的主机在同一个子网内，则会按照 arp 的过程对 dns 服务器进行 arp 查询；
- 如果 dns 服务器和我们的主机不在同一个子网内，则会按照 arp 的过程对默认网关进行查询；

ARP 过程：
想要发送 arp 地址解析协议，我们需要一个目标 ip 地址，同时还需要知道用于发送 arp 广播的接口 mac 地址；
- 首先查询缓存，如果命中便直接返回结果；
- 如果没命中：
  - 查看路由表，看看目标 ip 是否在本地路由表中的某个子网内。在的话，使用与其子网相连的接口，否则使用与默认网关相连的接口；
  - 查询选择的网络接口的 mac 地址；
  - 发送 arp 请求。

拿到 dns 服务器或者默认网关的 ip 地址，就可以继续 dns 请求：
- 使用 53 端口向 dns 服务器发送 udp 请求包；
- 如果本地 dns 服务器没有找到结果，它便会发送一个递归查询请求，直到找见结果并返回。

Socket：
当浏览器得到了目标服务器 ip 地址，以及 http 协议或 https 的默认端口号 80 或 443，便会使用系统调用 socket 来请求一个 tcp stream socket。
1. 该请求首先被交给传输层，在传输层请求被封装成 tcp segment。目标端口会被加入头部，源端口会在系统内核端口范围内动态获取；
2. tcp segment 被送往网络层，网络层会在其中在加入 ip 头部，其中包含了目标服务器的 ip 地址及本机的 ip 地址，并将其封装为一个 ip datagram（packet）；
3. 这个 tcp datagram 接下来会进入链路层，链路层会为其加入 frame（贞）头部，里面包含着本机网卡的 mac 地址以及网关（本地路由器）的 mac 地址。

至此，整个 tcp 包已准备就绪，可交给物理层进行传输。

最终，封包会到达管理本地子网的路由器。
一路上经过的这些路由器会从 ip 数据报头部，取出目标地址，并将封包正确地路由到下一个目的地。ip 数据报头部的 TTL time to live 的值会没经过一个路由器就减 1，当减至 0 时该包就会被丢弃。

客户端与服务端进行三次握手：
1. 客户端选择一个初始序列号，并且将设置了 SYN 标志位的包发送给服务端，以表明需要建立连接并且设置了初始序列号；
2. 服务端收到 SYN 包，如果它可以建立连接：
  - 服务器选择自己的初始序列号
  - 服务器设置 SYN 位，表明自己所选择的那个初始序列号
  - 服务器将客户端序列号 +1 复制到确认序号域，并且设置 ACK 位，表明自己已收到客户端的第一个封包已经其所期望的下一个字节
3. 客户端通过发送下面的一个封包来确认这次连接：
  - 自己的序列号 +1
  - 确认序号域设置为服务端序号 +1，并且设置 ACK 位

**数据传输方式：**
- 当一方发送了 N 各 bytes 的数据之后，将自己的序号也增加 N；
- 对方收到这个数据包之后，发送 ACK 包，并且设置自己的确认序号值为接受到的数据包序号最后一个字节 +1，以表明它的期待。

关闭连接时：
1. 要关闭一方发送一个 FIN 包；
2. 另一方确认这个 FIN 包；
3. 另一方发送自己的 FIN 包；
4. 要关闭的一方使用 ACK 确认接收到了 FIN；

**TLS 握手：**
1. 客户端发送一个 ClientHello 消息到服务器端，消息中同时包含了它的 Transport Layer Secure 版本，可用的加密算法和压缩算法；
2. 服务器端向客户端返回一个 ServerHello 消息，消息中包含了服务器端的 TLS 版本，服务器端所选择的加密和压缩算法，以及数字证书认证机构 CA 所签发的服务器公开证书，其中包含了公钥。客户端会使用这个公钥来加密接下来的握手过程，知道协商出一个新的对称密钥；
3. 客户端根据自己信任的 CA 列表，来验证服务端的证书是否可信。如果认为可信，客户端会生成一串伪随机数（这串随机数会用与生成新的对称密钥），使用服务器的公钥加密它；
4. 服务器使用自己的私钥来解密上面的随机数，然后使用这串随机数来生成自己的对称主密钥；
5. 客户端发送一个 Finished 消息给服务器端，使用对称密钥加密这次通信并生成一个散列值；
6. 服务器端生成自己的 hash 值，解密客户端的消息并判断两者是否对应，如果对应，就向客户端发送一个 finished 消息，也使用已协商好的对称密钥；
7. 接下来整个会话都使用对称密钥进行加密，传输应用层 http 内容。


















