## 0.前言

### 0.1 概括

- 一致：数据一致性，Paxos的实现ZAB协议
- 有头：必须选出一个Leader
- 数据树：每个数据节点都可以有子节点，但是区别于目录树的是每个数据节点都必须有数据

### 0.2 理论

- CAP：分布式系统CAP理论，一致性、可用性、分区容错性
- CBP：能力、美貌、性格，只能同时满足两个，哈哈哈

## 1.安装及使用

### 1.1 docker 安装

```bash
# 获取镜像
sudo docker pull wurstmeister/zookeeper
# 启动
sudo docker run --name zookeeper -p 12181:2181 -d wurstmeister/zookeeper:latest
```



### 1.2 普通安装



## 2.应用场景

- 配置一致
- HA
- pub/sub
- naming service
- load balance
- 分布式锁
- 。。。。。。

