## 0.前言

- 快
- epoll 多路复用
- 多种数据类型
- worker：单线程，原子行 -- 1个CPU核心，串行执行
- io thread：多线程 -- 多个CPU核心，相同连接内是有顺序的（如果客户端是多线程的，也不能保证顺序），不同连接无法保证顺序
- **通读配置文件**

## 1.安装

```bash
# 下载镜像
docker pull redis:latest
# 启动redis容器
docker run -itd --name redis-test -p 6379:6379 redis
# 进入交互式命令
docker exec -it redis-test /bin/bash
# 登录
redis-cli
```

## 2.应用场景

### 2.1 场景

- session
- kv缓存
- 数值计算计数器
- fs文件系统（内存级）

### 2.2 五数据类型

#### String

````bash
## 字符串
# 设置
set k1 aaa
# 查询
get k1
# 追加
append k1 bbb
# 返回字节个数
strlen k1

## 数值计算
# 数字
set k2 9
# +1
incr k2
# -1
decr k2

## 二进制 bitmap ascii
setbit k1 1 1
get k5 // "@" 
setbit k1 7 1
get k5 // "A"
# 二进制操作 
# k1, k2 按位 and
bitop and k3 k1 k2
# k1, k2 按位 or 日活跃用户数
setbit 20200708 1 1
setbit 20200709 1 1
setbit 20200709 9 1
bitop or total 20200708 20200709 
bitcount total
````

#### List

```bash
## 栈模型-同向
# 从左边push
lpush a b c d e
# 从左边top
lpop a b c d e
# 从右边push
rpush a b c d e
# 从右边push
rpop a b c d e

## 栈模型-同向
# 从左边push
lpush a b c d e
# 从左边top
rpop a b c d e
# 从右边push
rpush a b c d e
# 从右边push
lpop a b c d e

## ltrim 可以手动优化list
ltrim 0 -1

## 场景：评论、消息队列、容器
```



#### Hash

````bash
## 操作
# 设置
hset stone name sidy age 35
# 获取一个值
hget stone name
# 获取所有值
hgetall stone
# 计算
hincrby stone age 1

## 场景：聚集数据、详情页
````



#### Set

```bash
## 操作 无序、不重复、不推荐使用
# 返回一个集合，无序的，不重复的
srandmember k1 3 
# 返回一个集合，无序的，可重复的
srandmember k2 -8

## 交并差集（好友推荐）
# 只返回结果
sunion k3 k1 k2
# 结果存到k3
sunionstore k3 k1 k2 

## 场景：推荐系统（好友推荐、商品推荐等）、随机事件、抽奖、验证码
```



#### SortedSet（ZSet） 

- 有序集合
- 去重
- 排序
- 底层数据结构：ziplist、skiplist（随机造层，最高层为64）

```bash
## 操作

## 场景：排行榜、评论、分页
```



## 3.持久化

### 3.1 快照（时间点）-默认方式

#### 3.1.1 方式

- RDB
- Image
- Bak

#### 3.1.2 特点

- 全量
- 丢失量比较大
- 体积接近
- 恢复速度快 I/O Read

### 3.2 日志 AOF

- sync：影响效率
- OS Pagecache：丢失量：5秒/10%内存
- 可能有效数据很小，但是AOF巨大，4.x之前的版本会定期重写AOF进行精简，4.x以后版本优化为重写时先拍快照，再重写AOF日志

***Redis默认开启RDB方式，可手工开启AOF，如果开启了AOF，则重启时，只会恢复AOF，不会恢复RDB***

## 4.部署

- 单点故障：主从复制 （主备）集群：每台机器全量
- 解决压力：数据分片集群：不是全量，每台分片机器做主从复制

### 4.1主从复制

#### 4.1.1 数据同步

- 强一致性：同步复制，容器造成集群不可用性 CP
- 弱一致性：异步复制，容易数据不一致 AP
- 最终一致性：HDFS有 journalnode多机集群。（Redis暂时没有，开发的时候就没考虑分布式一致性的问题）。需要分布式协调，paxos：论文，zab、raft：paxos论文的实现

### 4.2 分片复制集

- 数据分片：借助分区 --> 机器的映射关系，为动态扩容提供灵活性
- 分片主从复制

  

