## 0.目录

- bin：所有linux系统的命令
- root：root用户主目录
- home：所有普通用户主目录
- usr：资源共享目录，软件一般安装在usr/local
- etc：配置文件目录

## 1.常用命令

https://mp.weixin.qq.com/s?__biz=MzU3NTgyODQ1Nw==&mid=2247492681&idx=2&sn=0896c71a73488fce258218b039f5f2f3&chksm=fd1f9ccfca6815d943b88de522103f5535fa724010a0d013bd62fd33cd89985d862cb40f043c&scene=126&sessionid=1594687571&key=e0611399b6399aa91cf64932fd290874e20a40dface318173ee65a8a23d4a5bc8307310c315d0a90457cfa8f3a0b416aab171527dbef29c2ab770e4bda90eb54fd57ffc3d24fa8dc271e237c36b9ccf4&ascene=1&uin=MTY3NzczNjk0MA%3D%3D&devicetype=Windows+10+x64&version=6209007b&lang=zh_CN&exportkey=AVZ23p2IGgWkbseyDQm93lE%3D&pass_ticket=GTBadQW%2FdFvK5dn6GKpvaLb5xR4g33CK8c%2FyMHXeFtFM%2FD5nII0mIRfeA%2BGlgqrr

## 2.性能分析
### 2.1 概述

```bash
yum install sysstat

uptime
dmesg | tail
vmstat 1
mpstat -P ALL 1
pidstat 1
iostat -xz 1
free -m
sar -n DEV 1
sar -n TCP,ETCP 1
top
```

### 2.2 uptime

````bash
[root@localhost ~]# uptime
 17:06:30 up  8:22,  1 user,  load average: 0.00, 0.01, 0.05
````

这是一个快速展示系统平均负载的方法，这也指出了等待运行进程的数量。在 Linux 系统中，这些数字包括等待 CPU 运行的进程数，也包括了被不可中断 I/O（通常是磁盘  I/O）阻塞的进程。这给出了资源负载的很直接的展示，可以在没有其它工具的帮助下更好的理解这些数据。它是唯一快捷的查看系统负载的方式。

这三个数字是以递减的方式统计了过去 1 分钟，5 分钟和 15  分钟常数的平均数。这三个数字给我们直观展示了随着时间的变化系统负载如何变化。例如，如果你被叫去查看一个有问题的服务器，并且 1  分钟的所代表的值比 15 分钟的值低很多，那么你可能由于太迟登陆机器而错过了问题发生的时间点。

在上面的例子中，平均负载显示是在不断增加的，1 分钟的值是 30，相比 15 分钟的值 19 来说是增加了。这个数字这么大就意味着有事情发生了：可能是 CPU 需求；

### 2.3 dmesg | tail

这里展示的是最近 10 条系统消息日志，如果系统消息没有就不会展示。主要是看由于性能问题导致的错误。上面这个例子中包含了杀死 OOM 问题的进程，丢弃 TCP 请求的问题。

### 2.4 vmstat

````bash
[root@localhost ~]# vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 259464   2108 1292392    0    0     6    27   52  119  0  0 100  0  0
 0  0      0 259472   2108 1292424    0    0     0     0  115  266  0  1 100  0  0
 0  0      0 259472   2108 1292424    0    0     0     0  104  255  0  0 100  0  0
 0  0      0 259472   2108 1292424    0    0     0     0  114  267  0  0 100  0  0
 0  0      0 259472   2108 1292424    0    0     0     0  100  244  0  1 100  0  0
````

vmstat 使用参数 1 来运行的时候，是每 1 秒打印一条统计信息。在这个版本的 vmstat 中，输出的第一行展示的是自从启动后的平均值，而不是前一秒的统计。所以现在，可以跳过第一行，除非你要看一下抬头的字段含义。

**每列含义说明：**

1. r: CPU 上的等待运行的可运行进程数。这个指标提供了判断 CPU 饱和度的数据，因为它不包含 I/O 等待的进程。可解释为：“r” 的值比 CPU 数大的时候就是饱和的。
2. free：空闲内存，单位是 k。如果这个数比较大，就说明你还有充足的空闲内存。“free -m” 和下面第 7 个命令，可以更详细的分析空闲内存的状态。
3. si，so：交换进来和交换出去的数据量，如果这两个值为非 0 值，那么就说明没有内存了。
4. us，sy，id，wa，st：这些是 CPU 时间的分解，是所有 CPU 的平均值。它们是用户时间，系统时间（内核），空闲，等待 I/O 时间，和被偷的时间（这里主要指其它的客户，或者使用 Xen，这些客户有自己独立的操作域）。

CPU 时间的分解可以帮助确定 CPU 是不是非常忙（通过用户时间和系统时间累加判断）。持续的 I/O 等待则表明磁盘是瓶颈。这种情况下 CPU  是比较空闲的，因为任务都由于等待磁盘 I/O 而被阻塞。你可以把等待 I/O 看作是另外一种形式的 CPU  空闲，而这个命令给了为什么它们空闲的线索。

系统时间对于 I/O 处理来说是必须的。比较高的平均系统时间消耗，比如超过了 20%，就有必要进一步探索分析了：也有可能是内核处理 I/O 效率不够高导致。

在上面的例子中，CPU 时间几乎都是用户级别的，说明这是一个应用级别的使用情况。如果 CPU 的使用率平均都超过了 90%。这不一定问题；可以使用 “r” 列来检查使用饱和度。

### 2.5 mpstat -P ALL

````bash
[root@localhost ~]# mpstat -P ALL 1
Linux 3.10.0-1062.el7.x86_64 (localhost.localdomain) 	06/11/2020 	_x86_64_	(2 CPU)

05:21:31 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
05:21:32 PM  all    0.00    0.00    0.50    0.00    0.00    0.00    0.00    0.00    0.00   99.50
05:21:32 PM    0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
05:21:32 PM    1    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00

05:21:32 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
05:21:33 PM  all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
05:21:33 PM    0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
05:21:33 PM    1    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00

05:21:33 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
05:21:34 PM  all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
05:21:34 PM    0    0.00    0.00    1.00    0.00    0.00    0.00    0.00    0.00    0.00   99.00
05:21:34 PM    1    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
````

这个命令分打印各个 CPU 的时间统计，可以看出整体 CPU 的使用是不是均衡的。有一个使用率明显较高的 CPU 就可以明显看出来这是一个单线程应用。

### 2.6 pidstat

````bash
[root@localhost ~]# pidstat 1
Linux 3.10.0-1062.el7.x86_64 (localhost.localdomain) 	06/11/2020 	_x86_64_	(2 CPU)

05:22:55 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
05:22:56 PM     0      1117    0.00    0.98    0.00    0.98     0  containerd
05:22:56 PM     0     14937    0.00    0.98    0.00    0.98     1  pidstat

05:22:56 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
05:22:57 PM     0     14937    1.00    2.00    0.00    3.00     1  pidstat

05:22:57 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
05:22:58 PM     0      1117    0.00    1.00    0.00    1.00     0  containerd
05:22:58 PM     0     14937    0.00    1.00    0.00    1.00     1  pidstat
````

pidstat 命令有点像 top 命令中的为每个 CPU 统计信息功能，但是它是以不断滚动更新的方式打印信息，而不是每次清屏打印。这个对于观察随时间变化的模式很有用，同时把你看到的信息（复制粘贴）记到你的调查记录中。

上面的例子可以看出是 2 个 java 进程在消耗 CPU。`%CPU` 列是所有 CPU 的使用率；1591% 是说明这个 java 进程消耗了几乎 16 个 CPU 核。

### 2.7 iostat -zx

```bash
Linux 3.10.0-1062.el7.x86_64 (localhost.localdomain) 	06/11/2020 	_x86_64_	(2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.10    0.00    0.22    0.01    0.00   99.67

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
scd0              0.00     0.00    0.00    0.00     0.03     0.00   114.22     0.00    1.22    1.22    0.00   0.89   0.00
sda               0.00     0.08    0.32    0.79    11.94    52.21   115.27     0.00    1.73    0.57    2.20   0.30   0.03
dm-0              0.00     0.00    0.25    0.87    10.92    52.15   112.08     0.00    3.47    0.70    4.28   0.29   0.03
dm-1              0.00     0.00    0.00    0.00     0.07     0.00    50.09     0.00    0.10    0.10    0.00   0.06   0.00
```

这个工具对于理解块设备（比如磁盘）很有用，展示了请求负载和性能数据。具体的数据看下面字段的解释：

1. r/s, w/s, rkB/s, wkB/s：这些表示设备上每秒钟的读写次数和读写的字节数（单位是 k 字节）。这些可以看出设备的负载情况。性能问题可能就是简单的因为大量的文件加载请求。
2. await：I/O 等待的平均时间（单位是毫秒）。这是应用程序所等待的时间，包含了等待队列中的时间和被调度服务的时间。过大的平均等待时间就预示着设备超负荷了或者说设备有问题了。
3. avgqu-sz：设备上请求的平均数。数值大于 1 可能表示设备饱和了（虽然设备通常都是可以支持并行请求的，特别是在背后挂了多个磁盘的虚拟设备）。
4. %util：设备利用率。是使用率的百分数，展示每秒钟设备工作的时间。这个数值大于 60% 则会导致性能很低（可以在 await 中看），当然这也取决于设备特点。这个数值接近 100% 则表示设备饱和了。

如果存储设备是一个逻辑磁盘设备，后面挂载了多个磁盘，那么 100% 的利用率则只是表示有些 I/O 是在 100% 处理，然而后端的磁盘或许远远没有饱和，还可以处理更多的请求。

请记住，磁盘 I/O 性能低不一定是应用程序的问题。许多技术通常都被用来实现异步执行 I/O，所以应用程序不会直接阻塞和承受延时（比如：预读取和写缓冲技术）。

### 2.8  free -m

```bash
[root@localhost ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1819         302         243           9        1273        1334
Swap:          2047           0        2047
```

右面两列展示的是：

1. buffers：用于块设备 I/O 缓冲的缓存。
2. cached：用于文件系统的页缓存。

我们只想检测这些缓存的数值是否接近 0 。不为 0 的可能导致较高的磁盘 I/O（通过 iostat 命令来确认）和较差的性能问题。上面的例子看起来没问题，都还有很多 M 字节。

“-/+ buffers/cache” 这一行提供了对已使用和空闲内存明确的统计。Linux 用空闲内存作为缓存，如果应用程序需要，可以快速拿回去。所以应该包含空闲内存那一列，这里就是这么统计的。甚至有一个网站专门来介绍 Linux 内存消耗的问题：[linuxatemyram](https://www.linuxatemyram.com/)。

如果在 Linux 上使用了 ZFS 文件系统，则可能会更乱，因为当我们在开发一些服务的时候，ZFS 有它自己的文件系统缓存，而这部分内存的消耗是不会在 `free -m` 这个命令中合理的反映的。显示了系统内存不足，但是 ZFS 的这部分缓存是可以被应用程序使用的。

### 2.9 sar -n DEV

```bash
[root@localhost ~]# sar -n DEV 1
Linux 3.10.0-1062.el7.x86_64 (localhost.localdomain) 	06/11/2020 	_x86_64_	(2 CPU)

05:30:49 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
05:30:50 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
05:30:50 PM     ens33      0.00      0.00      0.00      0.00      0.00      0.00      0.00
05:30:50 PM   docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00

05:30:50 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
05:30:51 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
05:30:51 PM     ens33      0.99      0.99      0.06      0.45      0.00      0.00      0.00
05:30:51 PM   docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00

05:30:51 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
05:30:52 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
05:30:52 PM     ens33      1.00      1.00      0.06      0.46      0.00      0.00      0.00
05:30:52 PM   docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```

使用这个工具是可以检测网络接口的吞吐：rxkB/s 和 txkB/s，作为收发数据负载的度量，也是检测是否达到收发极限。

### 2.10 sar -n TCP,ETCP

```bash
[root@localhost ~]# sar -n TCP,ETCP 1
Linux 3.10.0-1062.el7.x86_64 (localhost.localdomain) 	06/11/2020 	_x86_64_	(2 CPU)

05:32:39 PM  active/s passive/s    iseg/s    oseg/s
05:32:40 PM      0.00      0.00      1.00      1.00

05:32:39 PM  atmptf/s  estres/s retrans/s isegerr/s   orsts/s
05:32:40 PM      0.00      0.00      0.00      0.00      0.00

05:32:40 PM  active/s passive/s    iseg/s    oseg/s
05:32:41 PM      0.00      0.00      1.00      1.00

05:32:40 PM  atmptf/s  estres/s retrans/s isegerr/s   orsts/s
05:32:41 PM      0.00      0.00      0.00      0.00      0.00
```

这是对 TCP 关键指标的统计，它包含了以下内容：

1. active/s：每秒本地发起的 TCP 连接数（例如通过 connect() 发起的连接）。
2. passive/s：每秒远程发起的连接数（例如通过 accept() 接受的连接）。
3. retrans/s：每秒 TCP 重传数。

这种主动和被动统计数通常用作对系统负载的粗略估计：新接受连接数（被动），下游连接数（主动）。可以把主动看作是外部的，被动的是内部，但是这个通常也不是非常准确（例如：当有本地到本地的连接时）。

重传是网络或者服务器有问题的一个信号；可能是一个不可靠的网络（例如：公网），或者可能是因为服务器过载了开始丢包。上面这个例子可以看出是每秒新建一个 TCP 连接。

### 2.11 top

```bash
[root@localhost ~]# top
top - 17:34:56 up  8:51,  1 user,  load average: 0.00, 0.01, 0.05
Tasks: 106 total,   1 running, 105 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.2 sy,  0.0 ni, 99.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1863104 total,   249176 free,   310032 used,  1303896 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.  1366140 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                                    
  1117 root      20   0  508068  40668  13980 S   0.3  2.2   5:17.32 containerd                                                                                                                                                                 
     1 root      20   0  128188   6768   4192 S   0.0  0.4   0:02.82 systemd                                                                                                                                                                    
     2 root      20   0       0      0      0 S   0.0  0.0   0:00.05 kthreadd                                                                                                                                                                   
     3 root      20   0       0      0      0 S   0.0  0.0   1:57.31 kworker/0:0                                                                                                                                                                
     4 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/0:0H                                                                                                                                                               
     5 root      20   0       0      0      0 S   0.0  0.0   0:08.65 kworker/u256:0                                                                                                                                                             
     6 root      20   0       0      0      0 S   0.0  0.0   0:02.99 ksoftirqd/0  
```

top 命令包含了很多我们前面提到的指标。这个命令可以很容易看出指标的变化表示负载的变化，这个看起来和前面的命令有很大不同。

top 的一个缺陷也比较明显，很难看出变化趋势，其它像 vmstat 和 pidstat  这样的工具就会很清晰，它们是以滚动的方式输出统计信息。所以如果你在看到有问题的信息时没有及时的暂停下来（Ctrl-S 是暂停, Ctrl-Q  是继续），那么这些有用的信息就会被清屏。

### 2.12 htop

yum install htop

### 2.13 iotop

yun install iotop

### 2.14 IPTraf

yum install IPTraf

## 3.网络

- tcp/ip
- BIO
- NIO
- 多路复用器
- netty

### 3.1 计算机组成

![](./assets/计组.png)

- 总线传输单位：4字节（32位），8字节（64位）
- 下面是 CPU 可能执行简单操作的几个步骤：
  - 加载（Load）：从主存中拷贝一个字节或一个字到内存中，覆盖寄存器先前的内容
  - 存储（Store）：将寄存器中的字节或字复制到主存的某个位置，覆盖该位置先前的内容
  - 操作（Operate）：把两个寄存器的内容复制到ALU，把两个字进行算数运算，并把结果存储到寄存器中，覆盖寄存器先前的内容
  - 跳转（Jump）：从指令中抽取一个字，把这个字复制到程序计数器（PC），覆盖原来的值
- 参考：https://mp.weixin.qq.com/s?__biz=MzI4MDEwNzAzNg==&mid=2649448120&idx=2&sn=99df7378dbde78c054c7e8c93d58ef32&chksm=f3a27fcbc4d5f6dd7a65fbbe9dfeb2d8ebe33418803e3303956a44fe621fd3921701c4ce8018&scene=126&sessionid=1595482537&key=649ec29a953848f471da363b57c2ad48591a32f308f625c9fa14e49a4e59b1d1035bfb8f93aaa2574ea225a0830b310a1e10f7edb2089bd6ba36faa1266aaada901d44b182cc42ba0aee477387a9e71d&ascene=1&uin=MTY3NzczNjk0MA%3D%3D&devicetype=Windows+10+x64&version=62090529&lang=zh_CN&exportkey=AXfRrgpxo4ms8VG8sXlo6S8%3D&pass_ticket=lKwZXeHx3NQ0r2wr2FOp%2F6CWXSMuEqJpSBw%2BoqjtexHFRzAhtky2trK6mETSmolC

#### 3.1.1 内存

##### 3.1.1.1 内核空间

- 内核分配一个绝对安全的内存空间，叫内核空间
- 保护模式，app不能直接访问内核空间的内存地址

##### 3.1.1.2 用户空间-APP

#### 3.1.2  CPU

- 晶振：触发系统中断
- 时钟中断：CPU执行下一个程序的指令，涉及到从内存拷贝到CPU缓存
- IO中断：鼠标、键盘等
- 程序多，CPU切换多，效率下降

#### 3.1.3 内存

#### 3.1.4 网卡

#### 3.1.5 硬盘

#### 3.1.6 外设

### 3.2 TCP/IP

- OSI：7层

- TCP/IP：

  - 4层，面向连接的、可靠的传输层协议。

  - 三次握手，数据传输，四次挥手。

    - 端口号数量：65535
    - 三次握手：sync（C）、ack（S）、ack（C）。确保双方确认对方收到了。双方开辟了资源为对方服务。

    ```bash
    02:05:18.776115 IP 192.168.1.113.44198 > 61.135.169.121.80: Flags [S], seq 2979211728, win 29200, options [mss 1460,sackOK,TS val 59615798 ecr 0,nop,wscale 7], length 0
    02:05:18.807046 IP 61.135.169.121.80 > 192.168.1.113.44198: Flags [S.], seq 4263402181, ack 2979211729, win 8192, options [mss 1452,sackOK,nop,nop,nop,nop,nop,nop,nop,nop,nop,nop,nop,wscale 5], length 0
    02:05:18.807163 IP 192.168.1.113.44198 > 61.135.169.121.80: Flags [.], ack 1, win 229, length 0
    ```

    

    - 数据传输：也是交互三次，双方确认

    ```bash
    02:05:18.807751 IP 192.168.1.113.44198 > 61.135.169.121.80: Flags [P.], seq 1:78, ack 1, win 229, length 77: HTTP: GET / HTTP/1.1
    02:05:18.838742 IP 61.135.169.121.80 > 192.168.1.113.44198: Flags [.], ack 78, win 916, length 0
    02:05:18.840891 IP 61.135.169.121.80 > 192.168.1.113.44198: Flags [.], seq 1:1461, ack 78, win 916, length 1460: HTTP: HTTP/1.1 200 OK
    02:05:18.840936 IP 192.168.1.113.44198 > 61.135.169.121.80: Flags [.], ack 1461, win 251, length 0
    02:05:18.841746 IP 61.135.169.121.80 > 192.168.1.113.44198: Flags [P.], seq 1461:2782, ack 78, win 916, length 1321: HTTP
    02:05:18.841812 IP 192.168.1.113.44198 > 61.135.169.121.80: Flags [.], ack 2782, win 274, length 0
    ```

    

    - 四次挥手：fin（C）、fin+ack（S）、fin（S）、ack（C）。确保双方释放为对方开辟的资源。客户端有可能不给ack，服务端需要有超时机制

    ```bash
    02:47:55.510958 IP 192.168.1.113.44200 > 61.135.169.121.80: Flags [F.], seq 78, ack 2782, win 272, length 0
    02:47:55.511097 IP 61.135.169.121.80 > 192.168.1.113.44200: Flags [P.], seq 1461:2782, ack 78, win 916, length 1321: HTTP
    # 02:47:55.511152 IP 192.168.1.113.44200 > 61.135.169.121.80: Flags [.], ack 2782, win 272, options [nop,nop,sack 1 {1461:2782}], length 0
    # 02:47:55.537597 IP 61.135.169.121.80 > 192.168.1.113.44200: Flags [.], ack 79, win 916, length 0
    02:47:55.537699 IP 61.135.169.121.80 > 192.168.1.113.44200: Flags [F.], seq 2782, ack 79, win 916, length 0
    02:47:55.537728 IP 192.168.1.113.44200 > 61.135.169.121.80: Flags [.], ack 2783, win 272, length 0
    ```

    

- tcpdump

```bash
yum install tcpdump
tcpdump -nn -i ens33 port 80
# curl www.baidu.com
exec 9<> /dev/tcp/www.baidu.com/80
echo -e "GET / HTTP/1.1\n" >& 9 
```



### 3.3 I/O

- strace：可以查看进程的线程

```bash
# yum isntall strace
strace -ff -o out {运行进程}
```

- netstat

```bash
# yum isntall net-tools
netstat -antp
```



- 模拟客户端

```bash
# yum isntall nc
nc localhost 8090
```



- 查看linux内核函数

```bash
# yum install -y man-pages
man 2 bind
```

**一个网络程序必然调用内核的三个函数：socket -> bind -> listen**

#### 3.3.1 同步I/O

##### 3.3.1.1 BIO

- 内核给阻塞的
- 每线程，每连接
- 优点：可以接收很多连接
- 缺点：
  - 线程内存浪费
  - CPU调度消耗
  - 根源：阻塞 accept recv

##### 3.3.1.2 NIO

- java nio：java new io 一套新的io体系，api
- linux nio：内核进化了，可以传个参数
- 非阻塞：一个线程处理多个连接
- 优点：
  - 规避多线程
  - C10K问题
- 缺点：
  - 假设1w个连接，只有一个发来数据，那么每循环一次，必须想内核发送1w次recv
  - 用户空间向内核空间的循环遍历，复杂度在系统调用上

##### 3.3.1.3 多路复用器 select/poll

- 内核又进化了
- 一个程序可以监听多个文件描述符
- 同步I/O多路复用器
- 优点：
  - 通过一次系统调用，把fds传给内核，内核进行遍历，减少系统调用的次数
- 缺点：
  - 重复传递文件描述符fd，解决方法：内核开辟空间保存fd
  - 每次select、poll都需要全新遍历全量fd

##### 3.3.1.4 多路复用器 epoll

- socket => 4
- bind(4,8080)
- listen(4)
- epoll_create => 7
- epoll_ctl(7,ADD,3,accept)
- epoll_wait(7) 阻塞，可设超时时间
- accept(4) = 8
- epoll_ctl(7,ADD,8,read)  7里面有4、8
- epoll_wait(4,8)

**如果程序自己调用recv读取I/O，那么这个I/O模型无论是BIO、NIO、多路复用器，统一叫同步I/O模型**

#### 3.3.2 异步I/O windows IOCP

#### 3.3.3 netty

- boss线程：如果只监听一个端口，则无论配置多少都只会启动一个线程，负责接收连接请求，向worker派发连接
- worker线程：配置多少，启动多少，负责处理连接的读写

### 3.4 秒杀

![](D:/Project/typora/tech/assets/秒杀.png)

### 3.5 分布式锁

- redis：CAP问题，如果强一致，则只能CP，如果弱一致，则只能AP，破坏一致性。
- zookeeper：
  - 分布式协调中心，分布式的基础。
  - 最终一致性：3、5、7等单数节点，异常节点下线，不提供服务，恢复后同步数据，数据一致后才可以提供服务。
  - 过半节点，如果不过半，可能出现脑裂。
  - 分布式更喜欢主从集群，paxos
- etcd：

