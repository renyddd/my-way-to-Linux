todo 乱序、穿插

## CAP 定理
https://juejin.cn/post/6895761455358935053
[wiki](https://zh.wikipedia.org/wiki/CAP%E5%AE%9A%E7%90%86)
CAP 指出一个分布式计算机系统不可能同时满足一下三点：
- 一致性 Consistency：等同于所有节点访问同一份最新的数据副本；
- 可用性 Availability：每次请求都能得到非错的响应，但是不保证获取到的数据为最新数据（保证每个请求不管成功或是失败都会有响应[ref](https://baike.baidu.com/item/CAP%E5%8E%9F%E5%88%99)）；
- 分区容错性 Partition tolerance：以实际效果而言，分区相当于对通信的时间限制要求，系统如不能在时限内达成数据一致性，就意味着发生了分区，当前操作必须在 C 和 A 中做出选择（系统中任意信息的丢失或失败不会影响系统的继续运作）。

> 如果在某个分布式系统中数据无副本，那么系统必然满足强一致性，因为只有独一数据，不会出现数据不一致的情况，此时兼备 C 和 P；但是如果系统发生了网络分区状况或着宕机，必然导致某些数据无法访问，此时可用性条件就不能被满足，即此时为 CP 系统。


根据定理，分布式系统只能满足其中的两项，不可能全部满足。
理解：想象两个节点分处分区两侧：
- 允许至少一个节点更新状态会导致数据不一致，即丧志 C 性质（就是说某一时刻点的瞬间不同步）；
- 如果为了保证数据一致性，将分区设置为一侧节点不可用，又会丧失了 A 性质；
- 除非两个节点间可互相通信，才能同时保证 A 和 C，但又会丧失 P 性质。

## 字符编码
每一个 0、1 即为计算机中存储信息的最小单元，称为 bit；byte 字节是计算机信息的计量单位，是通信和存储上的概念，8 个比特组成一个字节。
- ASCII 全称 American Standard Code For Information Interchange，美国信息交换标准码，0～127 个字符，分为控制字符与打印字符（\n,\t,a,b,c 等）。
其中 ASCII 只用了一个字节的前 7 个比特，第一位规定为 0；
- 非 ASCII 编码：例如法国、俄国需要添加自己特有的字符时，就会独立利用 ASCII 码中剩余下来的第一位，所有这些不同的编码方式里只有 128-255 这一段是不同的。而例如汉字，无法用一个字节全部表示，就需使用额外一字节来表示 256\*256 个字符。
- Unicode，如上全球存在着多种编码方式，如不提前得知一段文本的编码方式则会导致读取乱码。Unicode 就是给每一个符号都分配一个独一无二的编码。
**Unicode 只是一个符号集，只规定了符号的二进制代码，却没有规定这个二进制代码的存储规范。**
UTF-8 是一种变长编码，可以使用 1-4 个字节表示一个符号。
**UTF-8 是 Unicode 的实现方式之一，**其两条编码规则如下：
1. 对于单字节的符号，字节第一位设置为 0，后 7 位为该符号 Unicode 码，因此对于英语字母的 UTF-8 编码是和 ASCII 相同的；
2. 对于大于 1 的 n 字节符号，最左侧第一个字节的前 n 位都设为 1，第 n+1 位设为 0；后面字节的前两位一律设为 10，剩下没提及到的二进制位，全部为这个符号的 Unicode。

由上的解读为：
1. 如果一个字节的第一位是 0，则这个字节单独就是一个字符；
2. 如果第一位是 1，则连续又几个 1 就表示当前字符占用多少个字节。

http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html

ANSI 全称 American National Standard Institute，美国国家标准协会。


### 内核创建进程时复制了那些数据结构？创建线程时又复制了哪些数据结构？
todo


## 线程的同步
todo：了解关键流程
https://www.guru99.com/process-synchronization.html#:~:text=Process%20Synchronization%20is%20the%20task,same%20shared%20data%20and%20resources.&text=So%20the%20change%20made%20by,accessed%20the%20same%20shared%20data.

1. 互斥量 mutex：是一把锁，在访问共享资源前对互斥量进行加锁，在访问完成后释放互斥量上的锁。
2. 信号量 semaphore：（上述的互斥锁只能用于一个资源的互斥访问）信号量可以实现多个同类资源的多线程互斥和同步：
- 当其值大于 0 时，表示当前可用资源量，
- 当其值小于 0 时，其绝对值表示等待使用该资源的进程数量。
- P 申请一个单位的资源；V 释放一个单位资源。
3. 读写锁 Readers-writer lock：也叫做多读者单写者锁，读操作可并发，写操作互斥。
- 当该资源被加了写锁时，其他进程的加读或写锁便会阻塞；
- 当该资源被加了读锁时，其他进程的加写锁会阻塞，加读锁会成功。


## 抢占进程调度
调度器简介 https://www.cnblogs.com/vamei/p/9364382.html
内核必须为所管理的全部进程决定，如何在进程间分配有限的计算资源，内核中安排进程执行的模块就是调度器 scheduler。
调度器可以切换进程状态，一个 Linux 进程下的进程有多种 R S D 等多种状态，但可归纳为三种基本类型：
1. 就绪 Ready：进程已获得了 cpu 以外所需的必要资源，等待到 cpu 即可执行；
2. 执行 Running：进程获得 cpu，执行进程；
3. 阻塞 Blocked：进程因为某个事件而无法执行，便会放弃 cpu 处于阻塞状态。

Linux 调度器就是 cpu 时间的管理员，负责做两件事情：
1. 选择某些就绪的进程来执行；
2. 打断某些执行中的进程，让他们变回就绪态；支持该双向转换的调度器被称为抢占式调度器。

> 抢占式内核可以保证系统的响应时间，最高优先级任务一旦就绪，便会得到 cpu 的使用权。

### 进程优先级
调度器分配 cpu 时间的基本依据是进程的优先级，根据优先级的特点可分为两类：
- 实时进程 Real-time process：优先级高，需要尽快被处理，仅能由 Linux 所创造；
- 普通进程 Normal process：优先级低、需要更长的执行时间，普通用户只能创建普通进程。

根据优先级，实时进程永远高于普通进程。进程的优先级是一个 0 到 139 的整数，数字越小优先级越高；其中实时进程占用 0-99，普通进程占用 100-139.

```bash
ps -l
```
一个普通进程的默认优先级是 120，但可用 nice 用于运行一个被修改了调度优先级的进程：
```bash
nice -n -20 vim &
```
nice 可调整值为 -20 到 19。

### O(n) 与 O(1) 调度器
以 Linux 2.6 内核作为使用分界点。
O(n) 调度器的名字来自该调度算法的时间复杂度，n 则代表系统中运行的进程个数。该调度器会在开始每个时间片时，去检查每个就绪进程的优先级，然后选择最高的去执行。
O(1) 调度器解决了前者的性能问题，每次选择要执行的进程的时间复杂度都是 1，即与进程数量无关。
  todo: 使用了什么数据结构？

### 完全公平调度器
CFS，completely Fair Scheduler 取代了 O(1) 调度器。
CFS 增加了一个**虚拟运行时 virtual time** 的概念，每次的某进程的执行，都会增加其虚拟运行时的记录。CFS 在每次选择时不按照优先级，而是**选择虚拟运行时最少的进程。**
CFS 会根据进程的优先级来计算一个时间片因子，针对同样一段真实时间片来计算次一次运行所需累加的 virtual time 数值。

### 容器化时代 cgroup 扩展
todo

## LRU
下文中。

## 死锁 deadlock
对于两个或以上的进程，双方都在等待对方停止运行，以获取系统资源，但是没有一方提前退出时就称为死锁。
例如 p1 占用了显示器，同时又必须使用打印机；而打印机正被 p2 占用，p2 又必须使用显示器。
发生条件（只有在四个条件同时满足时才会发生，预防死锁必须破坏其中一个）：
1. 禁止抢占 no preemption：系统资源不能被强制从一个进程中退出；
2. 持有和等待 hold and wait：一个进程可以在其等待时持有系统资源；
3. 互斥 mutual exclusion：资源只能同一时分配给一个进程，无法被共享；
4. 循环等待 circular waiting：一系列进程互相持有其他进程所需要的资源。

## 原子操作
https://www.cnblogs.com/fanzhidongyzby/p/3654855.html

todo
- 为何处理器是单条汇编指令执行？
- 被高级语言所翻译成的片段汇编语言，也会被不同进程交替执行吗？

## go 实现的线程池
https://learnku.com/docs/gobyexample/2020/work-pool-worker/6285

## 程序的编译过程
todo

## 虚拟内存
ref：深入理解计算机系统

虚拟内存是一种对主存的抽象概念，是硬件异常、硬件地址翻译、主存、磁盘文件和内核软件的完美交互。
vm 通过一种清晰的机制，提供了三个重要的能力：
1. 它将主存看成是一个存储在磁盘上的地址空间的高速缓存，在主存中只保存活动区域，并根据需要在磁盘和主存之间来回传送数据，通过这种方式高效的使用了主存；
2. 它为每个进程提供了一致的地址空间，从而简化了内存管理；
3. 它保护了每个进程的地址空间不被其他进程破坏。

### 物理和虚拟寻址
计算机系统的主存被组织成一个由 M 个连续的**字节大小**的单元组成，每个字节都有一个唯一的物理地址，第一个字节的地址为 0，依次增加。
cpu 访问内存最自然的方式就是使用物理地址，这称为物理寻址。

现代处理器使用虚拟寻址的形式，cpu 通过生成一个 virtual address 来访问主存，这个地址在被送往主存之前需先经过转换为适当的物理地址，这个任务叫 address translation。
cpu 芯片上的*内存管理单元*（Memory Management Unit）是转用硬件，利用存放在主存中的查询表，来动态翻译虚拟地址。

### 地址空间
*地址空间*是一个非负整数地址的有序集合，我们总认为其实连续的，也就是一个线性地址空间。一个地址空间的大小是由表示最大地址所需的位数来描述的（例：0x7ffff。）
- 虚拟地址空间：cpu 从一个 N = 2\*\*n 个地址的地址空间中生成虚拟地址；
- 物理地址空间：对应于系统中物理内存的 M 个字节。

现代操作系统通常支持 32 位或 64 位虚拟地址空间。因此在 32 位的机器上最大物理寻址为 2^32 字节，也就是 4 G
```python
2 ** 32 / 1024 / 1024 / 1024
```
而 64 位系统最大物理寻址则是 17179869184 G，理论上为即无限大。

### 虚拟内存作为缓存的工具
概念上，虚拟内存被组织为一个由存放在磁盘上的 N 个连续的字节大小的单元组成的数组。磁盘上数组的内容被缓存在主存中。
磁盘（较低层）上的数据被分割成块，这些块作为磁盘和主存（较高层）之间的传输单元。
虚拟内存被分割为虚拟页 Virtual Page，VP；物理内存被分割为物理页 Physical Page，PP，物理页也被称为页帧 page frame。
在任何时刻，虚拟页面 VP 的集合都分为三个不相交的子集：
- 未分配的：vm 系统还未分配或未创建的页，未分配的块没有任何数据和他们相关联，因此也就不占用磁盘空间；
- 缓存的：当前已缓存在物理内存中的已分配页；
- 未缓存的：未缓存在物理内存中的已分配页。

### 内存映射
todo
[wiki](https://zh.wikipedia.org/wiki/%E5%86%85%E5%AD%98%E6%98%A0%E5%B0%84%E6%96%87%E4%BB%B6)
> 最常见用途是绝大多数操作系统(包括Microsoft Windows与Unix-like系统)用于加载进程。
解惑：操作系统是如何加载程序代码的？如何保存至运行进程中的代码与数据段的？

Memory-mapped file 是一段虚拟内存逐字节对应于一个文件，使得应用程序处理映射部分如同访问主存。

主要用处有增加对大文件的 I/O 性能；但对于小文件的映射，会导致碎片空间浪费，因为内存映射总需要对齐 page 边界，这起码就是 4K，因而对于一个 5K 的文件起码会浪费 3KiB。
缺点：占用一块很大的连续虚拟地址空间。


### mmap
[ref](https://blog.csdn.net/mg0832058/article/details/5890688)


#### 匿名映射
```bash
man mmap

void *mmap(void *addr, size_t length, int prot, int flags,
                  int fd, off_t offset);

 MAP_ANONYMOUS
              The mapping is not backed by any file; its contents are initialized to zero.  The fd argument is ignored; however, some implementations require  fd  to
              be  -1  if  MAP_ANONYMOUS  (or  MAP_ANON)  is specified, and portable applications should ensure this.  The offset argument should be zero.  The use of
              MAP_ANONYMOUS in conjunction with MAP_SHARED is supported on Linux only since kernel 2.4.
```
[ref](https://stackoverflow.com/questions/34042915/what-is-the-purpose-of-map-anonymous-flag-in-mmap-system-call)
一个区域可以映射到一个匿名文件，匿名文件由内核创建，包含全二进制零。

[http://luodw.cc/2016/08/13/linux-cache/](http://luodw.cc/2016/08/13/linux-cache/)


### 静态库、动态库
ref: https://blog.csdn.net/kang___xi/article/details/80210717

### 进程 fork 细节
ref: https://blog.csdn.net/xy010902100449/article/details/44851453

fork() 会产生一个和父进程完全相同的子进程，但此后子进程多会执行 exec 系统调用；Linux 中有“写时复制”技术，也就是说，只有在子进程空间的内容发生变化时，系统才会将父进程的内容复制一份给子进程。
在 fork 之后 exec 之前，两个进程用的是相同的物理空间，子进程的代码、数据、堆和栈都指向父进程的物理空间；也就是两者的虚拟内存地址不一样，但其对应的都是同一个物理空间。
当父\子进程中有更改段行为发生时，内核才会为子进程相应的段空间分配物理空间（段，是对虚拟内存说的吗？）；
- 如果不是因为 exec，内核会给子进程的数据段、堆栈分配相应的物理空间（父子各自拥有互不影响），而代码段子进程持续共享，因为两者完全相同。
- 如果时因为 exec，由于两者将执行不同的代码，内核也会为子进程分配单独的物理空间。

结合内存实现来看：
> fork 使子进程获得于父进程相同的数据代码空间、堆和栈，所以变量的虚拟内存地址也是相同的。每个进程都有自己的虚拟内存地址空间，不同进程的相同的虚拟地址显然可以对应不同的物理地址；因此相同的虚拟地址可呈现不同的值。

由于写时复制的机制，调度器一般会先执行子进程，因为时常子进程将立马执行 exec，将会清空和父进程共享的空间，而加载新的代码段；这就避免了发生“复制”。而若父进程先执行，则会导致子进程发生无用的复制。

### 虚拟内存、常驻内存与共享内存
其分别对应 top 头部字段的 VIRT, RES（Resident）, SHR：
```bash
   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND


   21. RES  --  Resident Memory Size (KiB)
           A subset of the virtual address space (VIRT) representing the non-swapped physical memory a task is currently using.  It is also  the  sum
           of the RSan, RSfd and RSsh fields.

           It  can  include private anonymous pages, private pages mapped to files (including program images and shared libraries) plus shared anony‐
           mous pages.  All such memory is backed by the swap file represented separately under SWAP.

           Lastly, this field may also include shared file-backed pages which, when modified, act as a dedicated swap file and thus will never impact
           SWAP.

   29. SHR  --  Shared Memory Size (KiB)
           A  subset  of  resident  memory  (RES) that may be used by other processes.  It will include shared anonymous pages and shared file-backed
           pages.  It also includes private pages mapped to files representing program images and shared libraries.

   44. VIRT  --  Virtual Memory Size (KiB)
           The  total  amount  of virtual memory used by the task.  It includes all code, data and shared libraries plus pages that have been swapped
           out and pages that have been mapped but not used.  


```

虚拟内存就是对应着上文所描述的，操作系统的抽象概念虚拟内存；
常驻内存（或者被翻译**驻留内存**），是指那些被映射到进程虚拟内存空间的物理内存，因为并不是每一个进程虚拟内存的栈、堆空间都被用满，这其实就表示着进程占用的实际内存。
共享内存 share，表示着进程虚拟内存中动态库等的大小；而实际上这部分内容的物理内存地址都是被多个进程所共享的。

### free 命令
```bash
~$ free -h
              total        used        free      shared  buff/cache   available
Mem:            15G        1.1G        1.4G        158M         12G         13G
Swap:            0B          0B          0B          0B          0B          0B
```

- total：物理内存大小；
- used：已使用内存大小(calculated as total - free - buffers - cache)；
- free：未使用内存大小；
- shared：由 tmpfs 使用的内存；
- bffers：Memory used by kernel buffers（写入磁盘时）；
- cache：Memory used by the page cache and slabs（）；
- buff/cache： buffers 和 cache 总和；
- availabel：Estimation of how much memory is available for starting new applications, without swapping. 还可以被应用程序使用的物理内存大小；

[ref](https://stackoverflow.com/questions/41426656/what-is-available-and-free-memory-in-response-of-free-command-on-linux) 现代操作系统的设计就是尽可能少的出现 free 内存，因为空闲 free 内存更难使用（需要先转换为可用）。in use 的内存虽然不是 free 但是是可用的 available，很容易 switch。

[ref](https://askubuntu.com/questions/867068/what-is-available-memory-while-using-free-command#:~:text=Free%20memory%20is%20the%20amount,process%20or%20to%20existing%20processes.)
free 内存是指没有在任何一处被使用的内存，这个数字因该尽可能的小，因为这就是被浪费了；available 是指可分配给新进程或已运行进程的内存。

### swap
[ref](https://www.cnblogs.com/kerrycode/p/5246383.html)
Linux 将物理 RAM 划分成内存块，并且称之为 pages。swapping 就是将 pages 复制到磁盘预配置位置的过程，以释放内存的页。物理内存与 swap space 数量的总和，就是可用的虚拟内存量。

当物理内存（RAM）已满时将使用 Linux 中的 swap space。如果系统需要更多的内存资源而 RAM 已满时，内存中非活动的 pages 将会被移至 swap space。

### LRU
当有新文件要被置换入缓存时，必须根据一定的原则来取代掉适当的文件，置换方法有 LRU，Least Recently Used 最近最久未使用。

[ref](https://skyhigh233.com/blog/2016/10/07/lru-cache/)
> 在内存有限的情况下，扩展一部分外存作为虚拟内存，真正的内存只存储当前运行时所需信息。虚拟页式存储管理，是将进程所需空间划分为多个页面，内存中指存放当前所需的页，其余放入外存。虚拟页式存储虽然减少了进程所需的内存空间，但却带来了运行时间变长的缺点，会无可避免的把在外存中的信息和内存中已有的信息进行交换。因而需要好的算法来减少读取外存的次数。

简记链表实现：维护一个定长的链表，每次新插入数据项的时候都进行头插，而当数据命中的时候则把该数据项移动至链表头部，如果该数据不存在，则新建数据项进行头插，并且删掉尾节点（尾节点就是最近最久未访问。）



### huge pages
> 大多数操作系统采用了分段或分页的方式进行管理。分段（segment 大小 64K）是粗粒度的管理方式，而分页则是细粒度管理方式，分页方式可以避免内存空间的浪费。相应地，也就存在内存的物理地址与虚拟地址的概念。通过前面这两种方式，CPU必须把虚拟地址转换程物理内存地址才能真正访问内存。为了提高这个转换效率，CPU 会缓存最近的虚拟内存地址和物理内存地址的映射关系，并保存在一个由 CPU 维护的映射表中。为了尽量提高内存的访问速度，需要在映射表中保存尽量多的映射关系。Linux的内存管理采取的是分页存取机制，为了保证物理内存能得到充分的利用，内核会按照LRU算法在适当的时候将物理内存中不经常使用的内存页自动交换到虚拟内存中，而将经常使用的信息保留到物理内存。通常情况下，Linux默认情况下每页是 4K，这就意味着如果物理内存很大，则映射表的条目将会非常多，会影响CPU的检索效率。因为内存大小是固定的，为了减少映射表的条目，可采取的办法只有增加页的尺寸。因此Hugepage便因此而来。也就是打破传统的小页面的内存管理方式，使用大页面 2M,4M 等。如此一来映射条目则明显减少。TLB（Translation Lookaside Buffer 用于提升虚拟地址到物理地址的转译速度）缓存命中率将大大提高。

> In short, by enabling huge pages, the system has fewer page tables to deal with and hence less overhead to access/maintain them!


## k8s device plugin
todo
[设备插件wiki](https://kubernetes.io/zh/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/)

## 多 master 中各组件的选举机制
todo

## k8s Pod spec 更新冲突
todo










