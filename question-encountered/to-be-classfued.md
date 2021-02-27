# Questions

一些未被归类的问题

“总结：对容器和虚拟化有一定了解，系统基础较差，Linux 下编程经验偏少，写码能力不太行”

## 进程与线程的区别？

任何进程都默认有一个主线程，线程用于执行二进制指令，而进程还要负责处理内存、文件、权限等。

进程相当于一个项目，而线程就是为了完成项目需求而建立的一个个开发任务。

Linux 中无论是进程还是线程，在内核里都被统称为任务（Task），并由 task\_struct 进行管理。创建线程时使用 clone 系统调用，线程共享进程的数据结构；创建进程时使用 fork 系统调用，从此父子进程各用各的数据结构。

线程是调度的基本单位，而进程则是资源拥有的基本单位。所谓内核中的任务调度，实际上的调度对象是线程；而进程只是给线程提供了虚拟内存、全局变量等资源。所以可以这么理解：

* 当进程只有一个线程时，可以认为进程就等于线程；
* 当进程拥有多个线程时，这些线程会共享相同的虚拟内存和全局变量等资源，这些资源在上下文切换时无需修改。
* 线程也有自己的私有数据，如栈和寄存器等，这些在上下文切换时也需要保存。

[参考 - 极客时间趣谈Linux操作系统](https://time.geekbang.org/column/article/91289)

[参考 - Linux 性能优化实战](https://time.geekbang.org/column/article/69859)

某进程在运行时，无法直接访问硬件设备，必须通过操作系统。操作系统有两个基本的功能：
1. 防止硬件被失控的应用程序滥用；
2. 向应用程序提供一套简单一致的机制，来控制复杂而又大不相同的低级硬件设备。

操作系统通过进程、虚拟内存和文件这三个抽象概念来实现如上两个功能。文件是对 I/O 设备的抽象，虚拟内存是对主存和磁盘 IO 设备的抽象，进程则是对处理器、主存和 I/O 的抽象。（文件就是字节序列）

操作系统会对某时刻运行的进程提供一种假象，仅有当前进程在运行；看上去是在独占使用处理器、主存和 I/O 设备。这便是通过进程抽象概念来完成的。

进程是对一个正在运行的程序的抽象，系统所运行的进程数是会多过 CPU 个数，操作系统实现的在多进程间的切换称为**上下文切换**。也就是操作系统将会保持追踪进程运行所需的所有状态信息，如 PC、寄存器当前值以及主存的内容。

内核是操作系统代码常驻主存的部分。内核不是一个独立的进程。它是操作系统管理全部进程所用代码和数据结构的集合。

一个进程可由多个称为**线程**的执行单元组成，每个线程都运行在进程的上下文中，共享同样的代码和全局数据结构。


## 虚拟内存

虚拟内存也是一个抽象的概念，它为每个进程提供了一种假象，每个进程都在独占的使用主存。

每个进程看到的内存都是一致的，称为虚拟地址空间。

地址空间最上面的区域（高地址空间）是保留给内核的，这里对所有进程来说都是一样的（所有进程都有可能调用相同的系统内核代码。）

虚拟内存可以结合磁盘和物理内存的优势，为进程提供看起来速度足够快且容量足够大的存储。[ref](https://draveness.me/whys-the-design-os-virtual-memory/)

### 布局

操作系统会为进程分配虚拟内存地址，阻止其对物理地址的访问，并且所有进程的地址都从 0 开始编号。

用户态与内核态（涉及系统调用）的进程，都使用虚拟地址。

进程的大块连续空间一分为二，内核空间与用户空间；用户空间在下，在低地址；内核空间在上，在高地址。地址由 0 开始从下往上增长。

```
comment
从低位排起：**Text Segment、Data Segment 和 BSS Segment。**（ELF 格式）

* Text Segment：存放二进制可执行代码；
* Data Segment：存放静态常量；
* BSS Segment：存放未初始化的静态变量。
```

**程序代码和数据**

**堆 Heap** 往高地址增长，是用来动态分配内存的区域 malloc 作用于此。

**Memory Mapping Segment** 共享库；用于将 so 文件映射到内存中。

**栈 Stack** 局部变量和函数调用的上下文。

**内核虚拟内存** 必须通过系统调用来执行。

普通进程无法访问内核空间，需要通过系统调用进入内核，所有进程看到的都是同一个内核空间、同一个进程列表。

进程 task\_struct 中的 struct mm\_struct 结构来管理内存；其中成员变量 task\_size 划分用户态于内核态地址空间。

[参考 - 极客时间趣谈Linux操作系统](https://time.geekbang.org/column/article/94926)

todo：进程、虚拟内存全部根据 cs 书进行补充

### 虚拟内存中的堆与数据结构中的区别
[栈的是那种含义](http://www.ruanyifeng.com/blog/2013/11/stack.html)
1. 数据结构，也就是数据的一种存放方式 LIFO；
2. 代码运行方式，即调用栈，函数层层存放实现层层调用；
3. 内存区域，存放数据的一种内存区域，解释如下：

> Stacks in computing architectures are regions of memory where data is added or removed in a last-in-first-out (LIFO) manner.

内存区域的 stack 是有结构的，明确每个区块的大小；而 heap 没有结构，数据可以任意存放，因此 stack 的寻址速度要快于 heap。

> Heap is used for dynamic memory allocation and unlike stack, the program needs to look up the data in heap using pointers (Think of it as a big multi-level library).

每个线程都会分配一个 stack，而每个进程才会分配一个 heap，也就是说 heap 是线程共享的。


todo：例如 c 中，int 一个变量与 malloc 一个变量有何区别？在 go 当中呢？
[ref - part1](https://deepu.tech/memory-management-in-programming/)
[ref - part2](https://deepu.tech/memory-management-in-golang/)

## goroutines
[effective go](https://golang.org/doc/effective_go#goroutines)
一个 goroutine 是一个轻量级的模型，它是在相同地址空间上与其它 goroutines 并发执行的函数。它的消耗仅比分配栈空间要多一点，初始的栈很少量并且可增长，有需要便会分配堆。

goroutines 是被多路复用到少量的 os 线程上的，而不是一比一映射（被 go runtime 所管理。）

go 中函数的 literal 就是闭包 closure，这确保了闭包中引用到的变量将持久存活：
> In Go, function literals are closures: the implementation makes sure the variables referred to by the function survive as long as they are active.


## Numa 非统一内存访问架构
Non-Uniform Memory Access，是为多处理器的电脑设计的非统一内存访问架构（描述了多处理器系统中主存模块相对于处理器的位置），内存的访问时间时依赖于处理器和内存之间的相对位置。这里的设计存在和处理器相对较近的内存，被称为本地内存；和相对较远的非本地内存。

一个 Numa Node 是由一个物理 cpu 和其本地内存组成（表示一组资源），如下命令：
```bash
~$ numastat
                           node0
numa_hit              5648809546
numa_miss                      0
numa_foreign                   0
interleave_hit             25434
local_node            5648809546
other_node                     0
```
ref: 
- [https://www.cnblogs.com/smlie/p/11546478.html](https://www.cnblogs.com/smlie/p/11546478.html)
- [http://www.sukihiro.cn/2020/03/25/numa-os-part1/](http://www.sukihiro.cn/2020/03/25/numa-os-part1/)

内存架构演进图[ref](https://lwn.net/Articles/250967/)
![nsbridge](https://static.lwn.net/images/cpumemory/cpumemory.4.png)

所有 cpu 通过公共总线连接到北桥，北桥还包含内存控制器，北桥必须与南桥通信，南桥亦称为 I/O 桥。需要注意如下：
- 多个 cpu 间的数据必须必须经过北桥的同一总线；
- 与 RAM 的通信必须通过北桥；
- cpu 与连接到南桥的设备的通信必须有北桥路由。

![Integrated Memory controlelr](https://static.lwn.net/images/cpumemory/cpumemory.6.png)

如上，是将内存控制器集成到 cpu 中，并将内存连接到每个 cpu。这种架构也是有缺点的；首先机器还是必须对所有系统内存都是可访问的，因此内存开始不再统一（因此该架构才有了 NUMA 这名字————非统一内存架构。）


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

常用信号：(todo 完善)

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





## 集线器、交换机和路由器的区别？

https://www.cnblogs.com/hnrainll/archive/2011/09/21/2183743.html

## 进程在什么情况下会进入到内核态？

用户态的进程所能访问的资源受限，必须通过系统调用陷入到内核中，才能访问这些核心资源。

从用户态到内核态的转变需要通过系统调用来完成，系统调用时会发生 CPU 上下文的切换。CPU 寄存器里原来用户态的指令位置，需要先保存起来；接着为了执行内核态代码 CPU 寄存器需要更新为内核态指令的新位置，最后才是跳转到内核态运行内核任务。

（注意：当进程执行系统调用陷入内核态之后，这些内核代码所使用的栈并不是原先再用户空间当中的，而是一个内核空间的栈，称作进程的**内核栈**）
进程从用户空间进入内核空间时，特权级发生变化，需要切换堆栈空间，内核空间中所使用的就称为内核栈。

而当系统调用结束后，CPU 寄存器需要恢复原来保存的用户态，然后再切换到用户空间继续用行。所以一次系统调用的过程，其实发生了两次 CPU 上下文切换。

不过在系统调用中所发生的与进程上下文切换不同，进程上下文切换是指从一个进程切换到另一个进程；而系统调用过程中一直是同一个进程在运行，也通常称为特权模式切换。

todo: 特权模式切换 [ref](https://www.cnblogs.com/gtarcoder/articles/5278074.html)

## 僵尸进程与孤儿进程
### 僵尸进程
那些已死但没有被回收的进程。
正常情况下将由父进程通过 wait 系统调用进行回收，若长时间保持大量僵尸进程将导致 resource leak（系统的 pid 数量有限制）。
应用：父进程想占用相同的进程号
子进程死亡后系统并不会立即从内存中删除其进程描述符，系统会发送 SIGCHLD 信号给父进程通知死亡，调用 wait 后才会从内存中完全删除。
处理：可使用 SIGCHLD 信号
```bash
kill -s SIGCHLD PARENT_PID
```
若父进程因为编码错误而忽略了 SIGCHLD 信号，在需要杀死僵尸进程的父进程，使 systemd 成为这些僵尸进程的新父进程，sytemd 会定期执行 wait() 调用。
### 孤儿进程
Orphan Process，是指在其父进程（创建该进程的进程）执行完成或被终止后仍继续运行的一类进程。
系统为避免孤儿进程退出时无法释放所占用的资源，systemd 进程会立马接管为其子进程。
应用：如在 shell 上用户想是该进程与会话脱离，并转至后台运行，nohup 命令；或需要长时间启动的进程，即守护进程 daemon。

## http 协议状态返回码

- 1xx：100-101，信息提示 
- 2xx：200-206，成功 
- 3xx：300-305，重定向 
- 4xx：400-415，错误类信息，客户端错误 
- 5xx：500-505，错误类信息，服务器端错误

常用的状态码： 
- 200：成功，请求的所有数据通过响应报文的 entity-body 部分发送；OK 
- 301：请求的 URL 指向的资源已经被删除；但在响应报文中通过首部 Location 指明了资源现在所处的新位置；Moved Permanently 
- 302：与 301 相似，但在响应报文中通过 Location 指明资源现在所处临时新位置；Found 
- 304：客户端发出了条件式请求，但服务器上的资源未曾发生改变，则通过响应此响应状态码通知客户端；Not Modified
- 401：需要输入账号和密码认证方能访问资源；Unauthorized 
- 403：请求被禁止；Forbidden 404：服务器无法找到客户端请求的资源；Not Found 
- 500：服务器内部错误；Internal Server Error
- 502：代理服务器从后端服务器收到一条伪响应；Bad Gateway

## awk 简单使用？
todo

## cpu 负载的具体数值是如何计算得出的？

uptime 命令中的过去 1 分钟、5 分钟、15 分钟的平均负载（Load Average），

```bash
man uptime
```

平均负载是指单位时间内系统中处于运行（runnable）状态和不可中断（uninterruptible）状态的平均进程数。

如下命令结合 htop 和容仪观测到，所有 cpu 将渐渐被打满：
```bash
while true; do dd if=/dev/zero of=/dev/null & sleep 10; done
```

不可中断状态的进程是正处于内核态关键流程中的进程，这些流程是不可打断的，即为 ps 命令中的 D 状态。（一般表示进程正在跟硬件交互，并且交互过程不允许被其他进程中断。）

lscpu 命令可查得 CPU 个数，最理想的情况是每颗上都运行着一个进程，当平均负载比 CPU 个数还大时，系统就已经出现过载了。

常见的进程状态：
- R：Running 或 Runnable，表示进程在 cpu 的就绪队列中正在运行或正在等待运行；
- D：Disk sleep 的缩写，也就是不可中断态 Uninterruptible sleep，表示进程正在跟硬件交互并且该过程不能被打断（不可被内核系统调用的信号所打断）；
- Z：Zombie 僵尸进程，表示该进程实际上是结束了，但父进程还没有为其回收资源；
- S：Interruptible sleep，也就是可中断睡眠态，表示进程因为等待某个事件而被系统挂起，当进程等待到事件的发生时就会被唤醒进入 R 状态；
- I：Idle 空闲状态，发生在不可中断睡眠的内核线程上；
- T：Stopped 或 Traced，表示进程处于暂停或跟踪状态；
- X：Dead 表示进程已经消亡，无法在 top 或 ps 中看到。

```bash
# man ps
PROCESS STATE CODES
       Here are the different values that the s, stat and state output specifiers (header "STAT" or "S") will
       display to describe the state of a process:

               D    uninterruptible sleep (usually IO)
               R    running or runnable (on run queue)
               S    interruptible sleep (waiting for an event to complete)
               T    stopped by job control signal
               t    stopped by debugger during the tracing
               W    paging (not valid since the 2.6.xx kernel)
               X    dead (should never be seen)
               Z    defunct ("zombie") process, terminated but not reaped by its parent

       For BSD formats and when the stat keyword is used, additional characters may be displayed:

               <    high-priority (not nice to other users)
               N    low-priority (nice to other users)
               L    has pages locked into memory (for real-time and custom IO)
               s    is a session leader
               l    is multi-threaded (using CLONE_THREAD, like NPTL pthreads do)
               +    is in the foreground process group
```

[参考 - Linux 性能优化实战](https://time.geekbang.org/column/article/69859)

[参考第二个回答](https://stackoverflow.com/questions/223644/what-is-an-uninterruptible-process)

当一个进程处在用户态的时候（user mode），他可以在任意时间被 interrupte（切换内核台 kernel mode）。当从内核态返回用户态的时候，将会进行是否有任何待处理信号（例如 SIGTERM, SIGKILL）的检查，这就意味着一个进程只能在返回用户态时被杀掉。
```bash
:~$ (sleep 200; cat .gitconfig ) | grep git
# another
:~$ ps axu | grep grep
renyido+ 3034639  0.0  0.0  12784   892 pts/0    S+   11:42   0:00 grep git
renyido+ 3034648  0.0  0.0  12784   968 pts/1    S+   11:42   0:00 grep grep
```

[how-can-i-put-process-into-uninterruptible-sleep](https://stackoverflow.com/questions/12304183/how-can-i-put-process-into-uninterruptible-sleep)

**进程无法在内核态下被杀死的原因是，这可能将会破坏同一台计算机上所有进程所使用的内核结构（对于线程 thread 来说也是如此。）**

当内核将做一些可能耗时较长的操作时（例如等待另一个向管道中写数据的进程，或者等待硬件），该程序自身将被标注为睡眠并且通知调度器切换至其他进程。

如果一个信号将要发送至一个 sleeping 的进程，则必须先唤醒它，然后才能返回用户态，从而再处理待处理的信号。下面是两种睡眠态的区别：
1. TASK_INTERRUPTIBLE: 此标记休眠的任务可通过信号唤醒，你无法避免处于此状态的进程，从硬盘中读写数据总是需要花些时间的。
2. TASK_UNINTERRUPTIBLE: 不可被唤醒。

## cpu 调度算法 - run queue
todo


## Linux 下有几种文件打开方式？

open

mmap

## 查看进程的资源使用情况？dstat 命令？

## perf 命令

## KVM 如何进行资源的限制？与 docker 有什么不同？
todo

## Cgroup v1, v2?
todo
[https://chrisdown.name/talks/cgroupv2/cgroupv2-fosdem.pdf](https://chrisdown.name/talks/cgroupv2/cgroupv2-fosdem.pdf)

关注：
1. How did this work in cgroupv1?

https://blog.csdn.net/juS3Ve/article/details/78566714
https://www.lijiaocn.com/%E6%8A%80%E5%B7%A7/2019/01/28/linux-tool-cgroup-detail.html

## 存储驱动 overlay2

[how-the-overlay2-driver-works](https://docs.docker.com/storage/storagedriver/overlayfs-driver/#how-the-overlay2-driver-works)

overlay 的不同层 layers 在 Linux 主机上表现为不同的目录，整个的统一过程可称为 union mount。底层目录称为 lowerdir，对应的 upperdir 和联合层 merged。


## containerd withrunc
[https://www.inovex.de/blog/containers-docker-containerd-nabla-kata-firecracker/](https://www.inovex.de/blog/containers-docker-containerd-nabla-kata-firecracker/)

[https://stackoverflow.com/questions/41645665/how-containerd-compares-to-runc](https://stackoverflow.com/questions/41645665/how-containerd-compares-to-runc)

1. containerd 是容器运行时，可管理整个容器的生命周期，从镜像的拉取到容器的执行、监控等；
2. container-shim 处理 headless 容器，这意味着一但 runc 完成了对容器的初始化便会退出，转由 container-shim 控制，充当一种中间人的角色；
3. runc 是一种统一的轻量级容器运行时，有 OCI（
Open Container Initiative）维护规范，containerd 就使用 runc 来产生与运行容器；
4. grpc 用于 containerd 和 docker-engine 间的通信；

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

每个进程的 namespace 都在它对应的 /proc/PID/ns 目录下，连接到真实的 namespace 文件上。一个进程也可以选择连接至某个已有进程的 ns 中，这就是 docker exec 的原理。
```bash
:~$ sudo ls /proc/242/ns/
cgroup  ipc  mnt  net  pid  pid_for_children  user  uts
```

## Linux Control Group

主要作用为限制一个进程组能够使用的资源上限，包括 CPU、内存、磁盘、网络带宽等等。

Cgroups 给用户暴露出来的接口是文件系统，即它以文件和目录的方式组织在操作系统的 /sys/fs/cgroup 路径下。可以通过 mount 命令进行展示：

```bash
:~$ sudo ls /sys/fs/cgroup/
blkio  cpuacct      cpuset   freezer  net_cls   net_prio    pids
cpu    cpu,cpuacct  devices  memory   net_cls,net_prio  perf_event  systemd

:~$ mount -t cgroup
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls,net_prio)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
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

## 容器 vs VM
VM 中的用户应用对宿主机操作系统的调用都会被虚拟化程序所拦截处理，这就多了一层消耗。

## 理解 Pod
Pod 扮演的是传统部署环境里的“虚拟机”的角色（进程是以组的形势运行的）。再把容器看作是运行在这个“机器”里的用户程序，凡是调度、网络、存储的事情进本都是 Pod 级别的。因此凡是跟容器的 Namespace 相关的属性，一定也是 Pod 级别的。

若使用例如 deployment 控制器的话，其中 template 字段的内容与一个标准的 Pod 对应的 API 定义丝毫不差。

[极客时间 - 深入剖析 Kubernetes](https://time.geekbang.org/column/article/40583)

## k8s pod 的生命周期
[kubernetes.io](https://kubernetes.io/zh/docs/concepts/workloads/pods/pod-lifecycle/)

[DZone - Birth of a Pod](https://dzone.com/articles/kubernetes-lifecycle-of-a-pod)

![pic](https://cdn-images-1.medium.com/max/1500/1*WDJmiyarVfcsDp6X1-lLFQ.png)

导致 Pod 诞生的一系列事件：

1. kubectl 或者其他任意的 API 客户端向 API server 提交 Pod spec；
2. API server 将该 Pod object 写入 etcd；一旦写入成功，etcd 将会向 API server 发回认可；
3. API server 反应 etcd 中的状态变化；
4. 所有 k8s 组件都通过 watch 持续监视 API server 相关的变更；
5. kube-scheduler 通过他的 watcher 发觉还有一个新的 Pod 还没有绑定任何节点；
6. kube-scheduler 为该 Pod 分配一个 Node 并且更新 API server；
7. 这个更改将会传给 etcd，API server 还会将这个节点的分配，反映到 Pod 对象中；
8. 每个 node 上都有 watcher 在监视 API server，因此目标节点监视到有一个新 Pod 被分配过来了；
9. Kubelet 运行容器并且向 API server 更新其状态；
10. API server 将该 Pod 持久化存储至 etcd；
11. 一旦 etcd 反馈写入成功，API server 便将该认可发送给 kubelet，表明这个事件已被接受。

[raft - Understandable Distributed Consensus](http://thesecretlivesofdata.com/)


## kubelet 作用整理
[ref - kubenetes.io](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kubelet/)

Kubelet 是每个节点上运行的主要 node agent，并负责向 API Server 注册该节点。
Kubelet 通过一段描述 Pod 的 Json 或 Yaml 的 PodSpec 来工作，kubelet 接受一组 API Server 下发的 PodSpec 并且确保这些 spec 中描述的容器健康的运行。

in-action 整理：
1. 向 API server 创建 Node 资源并注册该节点；
2. 持续监控 API server 是否该节点分配给 Pod 然后启动 Pod 容器；
3. 监控容器运行，持续报告状态、事件；
4. 运行容器存活探针的组件，并且将负责重启容器。

## ping 命令使用到的协议详解？
todo

ICMP: Internet Control Message Protocol

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

## Top k
[https://www.geeksforgeeks.org/k-largestor-smallest-elements-in-an-array/](https://www.geeksforgeeks.org/k-largestor-smallest-elements-in-an-array/)

# C 中的问题

## 各种 sizeof

有如下程序代码：

```c
#include <stdio.h>
#include <string.h>

int main() {
    char* a = "aaa";

    printf("%ld\n", strlen(a));
    printf("%ld\n", sizeof(a));
    printf("%ld\n", sizeof(char*));
    /*
3
8
8
    */
    
    printf("\n======\n");

    char str[] = "hello!";
    printf("%ld\n", strlen(str));
    printf("%ld\n", sizeof(str));

    const char* ch = str;
    printf("%ld\n", sizeof(ch));
    /*
6
7
8
    */

    printf("\n======\n");

    char src[3];
    strcpy(src, "abcde");
    printf("%ld\n", strlen(src));
    printf("%ld\n", sizeof(src));
    printf("%c\n", src[4]);
    /*
5
3
e
    */
    
	/*
test.c:24:5: warning: ‘__builtin_memcpy’ writing 6 bytes into
 a region of size 3 overflows the destination [-Wstringop-ove
rflow=]
   24 |     strcpy(src, "abcde");
      |     ^~~~~~~~~~~~~~~~~~~~
    */

    return 0;
}
```

解析：

```c
#include <string.h>
size_t strlen(const char *s); 
```

函数 strlen 计算 s 所指向的字符串的长度，**不包括**其结束标记`\0`；返回值为 s 指向的字符串的字节数。

`sizeof`是**运算符**，

C 语言对内存的限制为什么？

## volatile 关键字
禁止编译器优化：
https://stackoverflow.com/questions/246127/why-is-volatile-needed-in-c


## 在 C 中的单例情况下使用多线程，会发生什么问题？又该如何上锁呢？



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



## rsync, ethtool 命令



## RAID 介绍及分类？以及容量计算？

Redundant Arrays of Inexpensive(Independent) Disks；

- 磁盘的并行读写：提高 IO 能力；

- 磁盘的冗余：提高耐用性；

RAID-0：条带卷，strip（夸一个或多个组件安排数据存储的卷）

- 数据分散存储，读写能力提升；
- 可用空间：N*min(S1, S2, ...)
- 无容错能力；
- 最少磁盘：2, 2+

RAID-1：镜像卷，mirror

- 读性能提升、写性能下降：资源要写两份；
- 可用空间：1*min(S1, S2, ...)
- 有冗余能力；
- 最少磁盘数：2, 2+

RAID-3：一块盘作为校验盘；剩余 N-1 块相当于 RAID-0 的同时读写。单一校验盘可恢复丢失数据，但往往也会成为系统瓶颈。

RAID-5：一块盘的大小做数据校验，N-1 块盘的大小做为数据盘，其中校验码分布在各个盘中：

- 读写性能提升；
- 可用空间：(N-1)*min(S1, S2, ...)
- 有容错能力的：一块磁盘
- 最少磁盘数：3

**混合类型：**多个 RAID 的等级组合起来，实现优势互补，广泛应用的只有 RAID01 和 RAID10。（首数字为高级，优先处理方式）

- RAID-01：先做条带化再做镜像（本质是对物理磁盘实现镜像，更好，一组镜像中有完整的数据）；
- RAID-10：先做镜像化再做条带（一组镜像中没有完整的数据）。

```c
              RAID-01
               RAID-1
    RAID-0               RAID-0
A1        A2           A1       A2
A3        A4           A3       A4

              RAID-10
               RAID-0
    RAID-1               RAID-1
A1        A1           A2       A2
A3        A3           A4       A4

```



## lsof 的常见使用情景？

man lsof



## 如何查看某命令使用（或是加载）到的动态库？



## keepalived 的协议是？

