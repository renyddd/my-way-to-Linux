# Questions

一些未被归类的问题

“总结：对容器和虚拟化有一定了解，系统基础较差，Linux 下编程经验偏少，写码能力不太行”

## 进程与线程的区别？

任何进程都默认有一个主线程，线程用于执行二进制指令，而进程还要负责处理内存、文件系统、权限等。

进程相当于一个项目，而线程就是为了完成项目需求而建立的一个个开发任务。

Linux 中无论是进程还是线程，在内核里都被统称为任务（Task），并由 task\_struct 进行管理。创建进程时使用 fork 系统调用，从此父子进程各用各的数据结构；创建线程时使用 clone 系统调用，线程共享进程的数据结构。

线程是调度的基本单位，而进程则是资源拥有的基本单位。所谓内核中的任务调度，实际上的调度对象是线程；而进程只是给线程提供了虚拟内存、全局变量等资源。所以可以这么理解：

* 当进程只有一个线程时，可以认为进程就等于线程；
* 当进程拥有多个线程时，这些线程会共享相同的虚拟内存和全局变量等资源，这些资源在上下文切换时无需修改。
* 线程也有自己的私有数据，如栈和寄存器等，这些在上下文切换时也需要保存。

[参考 - 极客时间趣谈Linux操作系统](https://time.geekbang.org/column/article/91289)

[参考 - Linux 性能优化实战](https://time.geekbang.org/column/article/69859)

## 虚拟内存？

### 布局

操作系统会为进程分配虚拟内存地址，阻止其对物理地址的访问，并且所有进程的地址都从 0 开始编号。

用户态与内核态（涉及系统调用）的进程，都使用虚拟地址。

进程的大块连续空间一分为二，内核空间与用户空间；用户空间在下，在低地址；内核空间在上，在高地址。

从低位排起：**Text Segment、Data Segment 和 BSS Segment。**（ELF 格式）

* Text Segment：存放二进制可执行代码；
* Data Segment：存放静态常量；
* BSS Segment：存放未初始化的静态变量。

**堆 Heap，**往高地址增长，是用来动态分配内存的区域 malloc 作用于此。

**Memory Mapping Segment** 用于将 so 文件映射到内存中。

**栈 Stack，**局部变量和函数调用的上下文。

普通进程无法访问内核空间，需要通过系统调用进入内核，所有进程看到的都是同一个内核空间、同一个进程列表。

进程 task\_struct 中的 struct mm\_struct 结构来管理内存；其中成员变量 task\_size 划分用户态于内核态地址空间。

[参考 - 极客时间趣谈Linux操作系统](https://time.geekbang.org/column/article/94926)

## 进程间通信方式？

### 管道

bash 中的竖线就是**匿名管道**，是一种通过缓存完成的数据单向传输机制，并且要求两进程有亲缘关系。

命名管道可通过 mkfifo 命令显示创建，成功创建后表现为一类型 p 的文件；进行输入后如不读取则相应进程会进入阻塞状态。

### 消息队列

将数据分成一个一个独立的数据单元，也就是消息体，有着固定大小的存储块。包括有 msgget、msgsnd、msgrcv 函数的使用。

查看创建的消息队列对象：ipcs -q

### 共享内存

将一块虚拟地址空降映射到相同的物理内存中。shmget 函数创建一个共享内存，shmat（attach）和 shmdt 解除绑定。

ipcs ­­-m

### 信号量 Semaphore

使同一个共享资源，同时只能被一个进程访问的保护机制。信号量为一计数器用于实现进程间的互斥与同步，而非进程间通信数据。

将信号量初始化为一个数值用以代表某种资源的总量，对信号量的两种原子操作 P 操作于 V 操作；申请资源操作于归还资源操作。

### 信号

常用信号：

* 1 号 SIGHUP：无需关闭进程而让其重读配置文件
* 2 号 SIGINT：中止正在运行的进程，相当于 Ctrl + C
* 9 号 SIGKILL：强制杀死运行中的进程
* 15 号 SIGTERM：终止正在运行中的进程
* 18 号 SIGCONT：继续运行
* 19 号 SIGSTOP：后台休眠，Ctrl + z

### socket

应用层和内核互通的机制就是通过 Socket 系统调用，而 Socket 不属于哪一层而是操作系统的概念。二到四层的处理代码在内核里面，七层的处理代码需要应用自己实现，两者需要跨内核态用户态通信。

socket 函数用于创建一个唯一标识的文件描述符。

[参考 - 极客时间趣谈Linux操作系统](https://time.geekbang.org/column/article/101719)

## 当系统系统资源消耗严重运行缓慢时你会怎么做？

查看 CPU 使用率；

平均负载，即为系统的平均活跃进程数；

进程上下文切换，虽本身为 Linux 一核心功能，但过多的切换会缩短进程真正的运行时间；

CPU 缓存命中率，CPU 的处理速度比内存的访问速度要快得多，为避免等待内存的相应 CPU 用于协调两者间巨大的性能差距。

### ebpf 的使用及系统缓存优化

利用系统缓存优化程序的运行效率：

Buffer 和 Cache 利用内存充当慢速磁盘与快速 CPU 之前的桥梁，可以加速 I/O 的访问速度；分别缓存的是对磁盘和文件系统的读写数据。

缓存的命中率，是指直接通过缓存获取数据的请求次数，占所有数据请求次数的百分比。命中率越高，表示使用缓存带来的收益越高程序的性能也越好。

cachestat 提供了整个操作系统缓存的读写命中情况；cachetop 提供了每个进程的缓存命中情况。两者都属于 bcc 软件包的一部分，都基于内核的 eBPF（extended Berkeley Packet Filters）机制，来跟踪内核中管理的缓存，并输出缓存的使用和命中情况。注：bcc-tools 需要内核版本 4.1 或更高。

还有涉及到 redis、memcached 部分后续再来补充！

[reference](https://time.geekbang.org/column/article/75242)

### Linux 网络性能

C10K 和 C1000K，单机同时处理一万、一百万个请求的问题；epoll 解决了 C10K 的问题，通过 dpdk 直接跳过网络协议栈在用户空间通过轮询的方式处理。

[reference](https://time.geekbang.org/column/article/81497)

## DNS 解析过程？

域名由一串点分隔开的字符串组成，被用作互联网中的某一台或某一组计算机的名称。每个被点分割开的字符串，就构成了域名中的一个层级，位置越靠后层级越高。

```bash
ping www.baidu.com
ping www.baidu.com.
```

com 是顶级域名，baidu 是二级域名，www 是三级域名。点（.）是所有域名的根，所有域名都以点作为后缀。

> DNS 协议在 TCP/IP 栈中属于应用层，不过实际传输还是基于 UDP 或者 TCP 并监听在 53 端口上。

域名解析使用递归的方式进行，从顶级开始，发送给每个层级的域名服务器直到得到解析结果。每级 DNS 服务器都会有解析记录缓存。DNS 服务器通过资源记录的方式来管理数据：

* A 记录，用来把域名转换成 IP 地址；
* CNAME 记录，用来创建别名；
* NS 记录，表示该域名对应的域名服务器地址。

dig 的 trace 功能展示了递归查询的整个过程，

```bash
dig +trace www.baidu.com
```

可通过 /etc/hosts 文件完成局域网内部的主机名解析。

[reference](https://time.geekbang.org/column/article/81850)

## https 以及 CA 认证？

## 进程在什么情况下会进入到内核态？

用户态的进程所能访问的资源受限，必须通过系统调用陷入到内核中，才能访问这些核心资源。

从用户态到内核态的转变需要通过系统调用来完成，系统调用时会发生 CPU 上下文的切换。CPU 寄存器里原来用户态的指令位置，需要先保存起来；接着为了执行内核态代码 CPU 寄存器需要更新为内核态指令的新位置，最后才是跳转到内核态运行内核任务。而当系统调用结束后，CPU 寄存器需要恢复原来保存的用户态，然后再切换到用户空间继续用行。所以一次系统调用的过程，其实发生了两次 CPU 上下文切换。

不过在系统调用中所发生的与进程上下文切换不同，进程上下文切换是指从一个进程切换到另一个进程；而系统调用过程中一直是同一个进程在运行，也通常称为特权模式切换。

## 僵死进程如何产生？又该如何处理？

## 对 nginx 返回的错误码完成统计排序？

1xx：100-101，信息提示 2xx：200-206，成功 3xx：300-305，重定向 4xx：400-415，错误类信息，客户端错误 5xx：500-505，错误类信息，服务器端错误

常用的状态码： 200：成功，请求的所有数据通过响应报文的entity-body 部分发送；OK 301：请求的 URL 指向的资源已经被删除；但在响应报文中通过首部 Location 指明了资源现在所处的新位置；Moved Permanently 302：与 301 相似，但在响应报文中通过 Location 指明资源现在所处临时新位置；Found 304：客户端发出了条件式请求，但服务器上的资源未曾发生改变，则通过响应此响应状态码通知客户端；Not Modified 401：需要输入账号和密码认证方能访问资源；Unauthorized 403：请求被禁止；Forbidden 404：服务器无法找到客户端请求的资源；Not Found 500：服务器内部错误；Internal Server Error 502：代理服务器从后端服务器收到一条伪响应；Bad Gateway

## awk 简单使用？

## cpu 负载的具体数值是如何计算得出的？

uptime 命令中的过去 1 分钟、5 分钟、15 分钟的平均负载（Load Average），

```bash
man uptime
```

平均负载是指单位时间内系统中处于运行（runnable）状态和不可中断（uninterruptable）状态的平均进程数。

不可中断状态的进程是正处于内核态关键流程中的进程，这些流程是不可打断的，即为 ps 命令中的 D 状态。

lscpu 命令可查得 CPU 个数，最理想的情况是每颗上都运行着一个进程，当平均负载比 CPU 个数还大时，系统就已经出现过载了。

[参考 - Linux 性能优化实战](https://time.geekbang.org/column/article/69859)

## Linux 下有几种文件打开方式？

open

mmap

## 查看进程的资源使用情况？dstat 命令？

## KVM 如何进行资源的限制？与 docker 有什么不同？

## IaaS 与 PaaS 的区别？云计算中用到的虚拟化与容器？

## namespace 的种类？及延伸？

容器技术的核心功能，就是通过约束和修改进程的动态表现，从而为其创造出一个“边界”。对于 Docker 来说，Cgroups 技术用来制造约束的主要手段；Namespace 技术则是用来修改进程视图的主要方法。

```bash
yum install man-pages # 安装函数帮助
man 2 clone
```

clone\(\) 函数在为我们创建一个新进程的时候，其参数 flags 可接受指定 CLONE\_NEWIPC、CLONE\_NEWNET 等参数。

* Mount：被隔离进程只能看到当前 Namespace 信息，与其他不同的是它对容器视图的改变一定是伴随着挂载操作（mount）才能生效；
* UTS
* IPC
* Network
* User

由此看来，用户运行在容器里的应用进程都是在由宿主机操作系统统一管理（同一内核，隔离得不彻底；类似时间就无法被 Namespace 化），只不过这些进程多了额外设置的 Namespace 参数；Docker 扮演的更多是旁路式的辅助和管理工作。

每个进程的 namespace 都在它对应的 /proc/PID/ns 目录下，连接到真实的 namespace 文件上。一个进程也可以选择某个已有进程的 ns 中，这就是 docker exec 的原理。

### Linux Control Group

主要作用为限制一个进程组能够使用的资源上限，包括 CPU、内存、磁盘、网络带宽等等。

Cgroups 给用户暴露出来的接口是文件系统，即它以文件和目录的方式组织在操作系统的 /sys/fs/cgroup 路径下。可以通过 mount 命令进行展示：

```bash
mount -t cgroup
```

以查看已运行的 docker 容器为例：

```bash
find /sys/fs/cgroup/ -iname docker -ls
# 以选定第一个容器为例
ls /sys/fs/cgroup/memory/docker/284319b
```

该目录下的 tasks 文件为该正在运行容器的相关进程号，

```bash
cat /sys/fs/cgroup/memory/docker/284319b/tasks
# 或者查看 cpu period 的限制
cat /sys/fs/cgroup/cpu,cpuacct/docker/284319b/cpu.cfs_period_us
```

Docker 项目的核心原理就是为待创建的用户进程：

1. 启用 Linux Namespace 配置；
2. 设置指定的 Cgroups 参数；
3. 切换进程的根目录（change root）；

其中 rootfs 只是一个操作系统所包含的文件、配置和目录，但并不包括内核。容器的 rootfs 由以下三部分组成：

1. 可读写层（rw）
2. Init 层（ro+wh）
3. 只读层（readonly + whiteout “白障”）

容器镜像的操作是增量式的，这些层都存储在：

```bash
/var/lib/docker/<storage driver>
```

还有 CoW 写时复制机制，在读时使用镜像中的现有文件，仅在需要做出修改时才将其复制至可读写层。

[参考 - 极客时间深入剖析 Kubernetes](https://time.geekbang.org/column/article/14642)

## ping 命令使用到的协议详解？

[https://cloud.tencent.com/developer/article/1656071](https://cloud.tencent.com/developer/article/1656071)

## C++ 多态？

多态性定义：一个操作返回不同。

静态联编：在编译阶段系统就要确定主调和被调的调用关系，所有非虚函数的调用均为静态联编；动态联编：在程序运行时才确定方法调用。

虚函数为首先是类的成员函数，定义关键字 virtual，注：只有在继承中才使用虚函数。

多态性条件：

1. 基类和派生类有同名同参同返回同访问权限的成员方法；
2. 基类的同名方法被虚化，派生类的同名方法将自动被虚化；
3. 基类指针指向派生类对象（通过基类指针只可访问派生类从基类继承而来的成员）。

总结：通过基类指针调用同名方法时，系统采用动态联编，调用派生类的同名方法。

```cpp
class Base {
    public:
        virtual void fun() {
            cout << "Base fun()" << endl;
        }
};

class Derived1: public Base {
    public:
        void fun() {
            cout << "Derived1 fun()" << endl;
        }
};

class Derived2: public Base {
  public:
        void fun() {
            cout << "Derived2 fun()" << endl;
        }
};
// client

Base *pb;
pb = new Base();
pb->fun(); // Base::fun()

pb = new Derived1();
pb->fun(); // Derived::fun()
```

空虚函数，有为空的函数体，但可以调用：

```cpp
virtual void fun() {}
```

纯虚函数，不可调用。

```cpp
virtual void fun() = 0;
```

抽象类：含有纯虚函数的类，且抽象类一定为基类，抽象类不可创建对象。

## 红黑树

极端情况下二叉树会退化为链表，时间复杂度会退化到 O\(n\)，因此你需要一种平衡二叉查找树。

平衡二叉树的严格定义：二叉树中任意一个节点的左右子树的高度相差不能大于 1。发明其的初衷是解决普通二叉查找树在频繁的插入、删除等动态更新的情况下，出现时间复杂度退化的问题。

> 平衡二叉查找树中”平衡“ 的意思，其实就是让整棵树左右看起来比较”对称“、比较”平衡“，不要出现左子树很高、右子树很矮的情况。这样就能让整棵树的高度相对来说低一些，相应的插入、删除、查找等操作的效率高一些。

红黑树（Red-Black Tree，R-B Tree）并非一种严格的平衡查找二叉树，满足以下定义：

* 根节点是黑色的；
* 每个叶子节点都是黑色的空节点（NIL），也就是说，叶子节点不存储数据；
* 任何相邻的节点都不能同时为红色，也就是说，红色节点是被黑色节点隔开的；
* 每个节点，从该节点到达其可达叶子节点的所有路径，都包含相同数目的黑色节点。

[参考 - 极客时间数据结构与算法之美](https://time.geekbang.org/column/article/68638)

## 启动流程

按下启动电源键时主板上电，主板上的 ROM（Read Only Memory）上固化有初始化程序 BIOS（Basic Input and Output System）；

在 BIOS 界面上有启动盘选项，MBR（Master Boot Record）在第一个扇区共 512 字节：

* 前 446 字节：bootloader；
* 中 64 字节：分区表；
* 后 2 字节：55AA 表示结束。

这些代码是由 Grub2（Grand Unified Bootloader Version 2）程序生成于此。

未完待续。

[参考 - 极客时间趣谈Linux操作系统](https://time.geekbang.org/column/article/89739)

### 内核启动

内核的启动从 init/main.c 文件中的 start\_kernel\(\) 开始，这相当于是内核的 main 函数。

系统开始创建第一个即 0 号进程，进程列表的第一个；

trap\_init\(\) 初始化中断门（Interrupt Gate）以处理各种中断，包括系统调用的中断。

mm\_init\(\) 初始化内存管理模块；

sched\_init\(\) 初始化调度模块；

vfs\_caches\_init\(\) 初始化基于内存的文件系统 rootfs。

创建第二个进程 1 号进程，运行为用户进程。其中 x86 有 Ring0~Ring3 机制，越往里权限越高。

> 能够访问关键资源的代码放在 Ring0 称为内核态（Kernel Mode）；将普通的程序代码放在 Ring3 称为用户态（User Mode）。

处于保护模式的系统禁止处于用户态的代码执行更高权限的指令，系统调用是其访问核心资源的统一入口；

> 当一个用户态的程序运行到一半，要访问一个核心资源，例如访问网卡发一个网络包，就需要暂停当前的运行，调用系统调用，接下来就轮到内核中的代码运行了。

在暂停时需要用内存来保存程序运行时的中间结果，保存寄存器 - 内核态执行系统调用 - 恢复寄存器 - 返回用户态。

ramdisk 部分待续；

创建统一管理内核态进程的 2 号进程，

[参考 - 极客时间趣谈Linux操作系统](https://time.geekbang.org/column/article/90109)

## C 中的问题

```c
char *a  "aaa"

strlen(a)

sizeof(a)
```

volatile 关键字

在 C 中的单例情况下使用多线程，会发生什么问题？又该如何上锁呢？

## bash 编程

已知经 md5sum 后的字符串，推算其源字符串

## 统计 tcp 连接状态

```bash
ss -s
```

## bash 编程 MySQL 删除用户权限？删除以库中以某字符开头的表？

## 对 500G 的文件使用 scp 或 rsync 同步时卡在 25G，明确磁盘够用网络正常，你会怎么做？

## 使用 echo 命令中 "/" 有替换作用吗？

```bash
str=aaa111bbb222ccc
echo ${str/a/b}
# baa111bbb222ccc 结果只替换一次
echo ${str//a/b}
# bbb111bbb222ccc 替换全部
```

[refer](https://blog.csdn.net/yi412/article/details/77061869)

## cut，sed 与 awk  的简单使用复习

```bash
]$  cp /etc/passwd /tmp
]$  tail /tmp/passwd -n1             
redis:x:992:988:Redis Database Server:/var/lib/redis:/sbin/nologin
```

### cut

```bash
  -b, --bytes=LIST        select only these bytes
  -c, --characters=LIST   select only these characters
  -f, --fields=LIST       select only these fields;

  Use one, and only one of -b, -c or -f.  Each LIST is made up of one
range, or many ranges separated by commas.
  N     N'th byte, character or field, counted from 1
  N-    from N'th byte, character or field, to end of line
  N-M   from N'th to M'th (included) byte, character or field
  -M    from first to M'th (included) byte, character or field

  -d, --delimiter=DELIM   use DELIM instead of TAB for field delimiter
```

示例：

```bash
]$  tail /etc/passwd -n1 | cut -d ':' -f1
redis
]$  tail /etc/passwd -n1 | cut -b -5    
redis
]$  tail /etc/passwd -n1 | cut -b 7-
x:992:988:Redis Database Server:/var/lib/redis:/sbin/nologin
```

### sed

sed - stream editor for filtering and transforming text，流编辑器。其中 input stream \(a file or input from a pipeline\)。sed works by making only one pass over the input\(s\)。

-n 参数只会将经过处理的行输出至屏幕，-i 直接编辑原文件。

```bash
]$  sed -n 's/redis/rrrrrs/p' passwd
rrrrrs:x:992:988:Redis Database Server:/var/lib/redis:/sbin/nologin
]$  sed -n 's/redis/rrrrrs/gp' passwd # 一行中全部替换
rrrrrs:x:992:988:Redis Database Server:/var/lib/rrrrrs:/sbin/nologin

]$  sed '$s/redis/rrrrrs/' passwd # 输出处理过的全文内容
]$  sed '$s/redis/rrrrrs/p' passwd # 若加 p 则会多出被处理的行
```

地址定界：

* 空地址：对全文进行处理
* **：指定行，/pattern/：被次模式所匹配到的每一行；**

```bash
]$  sed '1c\redis' passwd # 对第一行进行替换命令
]$  sed '1i\redis' passwd # 在第一行前进行插入命令
]$  sed '1d' passwd # 删除第一行

# 以下都为显示最后一行
]$  sed -n '$p' passwd # 只输出最后一行
]$  sed '$!d' passwd # 只不对最后一行进行删除操作
```

### awk

pattern scanning and processing language

## RAID 介绍及分类？

Redundant Arrays of Inexpensive Disks；Independent
