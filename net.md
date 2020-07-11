## 0.前言

- BIO
- NIO
- 多路复用器
- netty

## 1.计算机组成

### 1.1 内存

#### 1.1.1 内核空间

- 内核分配一个绝对安全的内存空间，叫内核空间
- 保护模式，app不能直接访问内核空间的内存地址

#### 1.1.2 用户空间-APP

#### 1.2 CPU

- 晶振：触发系统中断
- 时钟中断：CPU执行下一个程序的指令，涉及到从内存拷贝到CPU缓存
- IO中断：鼠标、键盘等
- 程序多，CPU切换多，效率下降

### 1.3 内存

### 1.4 网卡

### 1.5 硬盘

### 1.6 外设

## 2.网络I/O

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

### 2.1 同步I/O

#### 2.1.1 BIO

- 内核给阻塞的
- 每线程，每连接
- 优点：可以接收很多连接
- 缺点：
  - 线程内存浪费
  - CPU调度消耗
  - 根源：阻塞 accept recv

#### 2.1.2 NIO

- java nio：java new io 一套新的io体系，api
- linux nio：内核进化了，可以传个参数
- 非阻塞：一个线程处理多个连接
- 优点：
  - 规避多线程
  - C10K问题
- 缺点：
  - 假设1w个连接，只有一个发来数据，那么每循环一次，必须想内核发送1w次recv
  - 用户空间向内核空间的循环遍历，复杂度在系统调用上

#### 2.1.3 多路复用器 select/poll

- 内核又进化了
- 一个程序可以监听多个文件描述符
- 同步I/O多路复用器
- 优点：
  - 通过一次系统调用，把fds传给内核，内核进行遍历，减少系统调用的次数
- 缺点：
  - 重复传递文件描述符fd，解决方法：内核开辟空间保存fd
  - 每次select、poll都需要全新遍历全量fd

**如果程序自己调用recv读取I/O，那么这个I/O模型无论是BIO、NIO、多路复用器，统一叫同步I/O模型**

#### 2.1.4 多路复用器 epoll

- socket => 4
- bind(4,8080)
- listen(4)
- epoll_create => 7
- epoll_ctl(7,ADD,3,accept)
- epoll_wait(7) 阻塞，可设超时事件
- accept(4) = 8
- epoll_ctl(7,ADD,8,read)  7里面有4、8
- epoll_wait(4,8)

### 2.2 异步I/O windows IOCP

### 2.3 netty

- boss线程：如果只监听一个端口，则无论配置多少都只会启动一个线程，负责接收连接请求，向worker派发连接
- worker线程：配置多少，启动多少，负责处理连接的读写

