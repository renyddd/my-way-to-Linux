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
- 首部校验和：值 ip 数据报包头的校验和，该数值用来检错，已确保包被正确无误的接收；

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

#### 三次握手
**请求 》应答 》应答之应答**





#### wait 等状态



















