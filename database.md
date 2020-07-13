## 0.概述

- 基本思想：分治，例如hashmap
- 存储技术：
  - 进程内：hashmap
  - 文件：全量IO
  - 数据库：
    - 分治，datapage（4KB）索引：
    - 索引也是数据，普通的索引还是全量IO。怎么优化？建树。如果是百亿行的表，增删改一定会慢，因为要维护索引；并发量小的时候查询依然是毫秒级，并发量大的时候，瓶颈在磁盘IO，可能是秒级甚至更大。如果数据放硬盘比放内存占空间大，因为内存可以存指针。
  - 内存数据库：冷、热分治：热数据放内存，冷数据并发量小，放硬盘。--> 内存数据库（redis）。redis为什么不支持SQL，因为redis不存全量数据。
  - 分布式文件系统（hdfs）：
    - 切成小块，并行存储，偷摸的将副本复制给其它节点
    - 两个角色：namenode、datanode，namenode记录文件存储位置信息，以datanode回执为准，不能以客户端回执为准。
  - hbase：
    - 大表，处理快，列存储
    - 列族：每个列族存在一个htable文件，相当于垂直切分
    - region：每个region相当于水平切分，相当于分库分表
  - ES：无主集群

## 1.mysql

### 1.1 安装

#### 1.1.1 yum方式安装

- 下载yum源

```bash
# 下载yum源
cd ~
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
# 安装yum源
yum localinstall mysql80-community-release-el7-3.noarch.rpm
# 检查yum源
yum repolist enabled | grep "mysql.*-community.*"
# 安装rpm包
yum install mysql-community-server
```



- 启动、停止、重启命令

```bash
systemctl start mysqld

systemctl stop mysqld

systemctl status mysqld
```



- 修改密码

```bash
grep 'temporary password' /var/log/mysqld.log

mysql -uroot -p 

ALTER USER 'root'@'localhost' IDENTIFIED BY '850417Sdy@';

GRANT ALL ON *.* TO 'root'@'%';

flush privileges;
```



- 更改加密方式

```bash
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '850417Sdy@';
```





#### 1.1.2 docker方式安装

- 安装并启动docker（略）

- 拉取 mysql 镜像

```bash
# 如果失败就多试几次或配置国内镜像仓库

docker pull mysql:latest

Using default tag: latest

latest: Pulling from library/mysql

Digest: sha256:e1b0fd480a11e5c37425a2591b6fbd32af886bfc6d6f404bd362be5e50a2e632

Status: Image is up to date for mysql:latest

docker.io/library/mysql:latest
```



- 运行 mysql 容器

```bash
# 1.这种方式没有挂载数据卷，容器每次重启或删除后数据会丢失

docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql --default-authentication-plugin=mysql_native_password

ab9b21764f583e58995e9f1466816d6295e5ec519c025b073cdf5898b4832c81

# 进入容器命令行

docker exec -it 236b2624632d bash

# 修改默认的密码验证插件

# mysql -uroot -p 

# ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';

# 2.挂载数据卷

mkdir -p /root/docker-mysql/data

chmod -R 777 /root/docker-mysql/*

docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -v /root/docker-mysql/data:/var/lib/mysql -d mysql --default-authentication-plugin=mysql_native_password
```



### 1.2 mysql调优

// TODO



## 2.mongo

### 2.1 原生安装

- 创建yum源

（1）创建/etc/yum.repos.d/mongodb-org-4.2.repo

（2）写入以下内容：

[mongodb-org-4.2]

name=MongoDB Repository

baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.2/x86_64/

gpgcheck=1

enabled=1

gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc

- yum安装

sudo yum install -y mongodb-org

- ulimit Settings

ulimit -f unlimited

ulimit -t unlimited

ulimit -v unlimited

ulimit -l unlimited

ulimit -n 64000

ulimit -m unlimited

ulimit -u 64000

- 启动、停止、重启

systemctl start mongod

systemctl stop mongod

systemctl restart mongod

systemctl status mongod

systemctl enable mongod

systemctl disable mongod

- 配置文件

/etc/mongod.conf



### 2.2 镜像安装

- 下载MongoDB的Docker镜像；

docker pull mongo:4.2.5

- 使用Docker命令启动MongoDB服务；

docker run -p 27017:27017 --name mongo  -v /mydata/mongo/db:/data/db  -d mongo:4.2.5

- 有时候我们需要为MongoDB设置账号，可以使用如下命令启动；

docker run -p 27017:27017 -name mongo -v /mydata/mongo/db:/data/db  -d mongo:4.2.5 --auth

- 然后我们需要进入容器中的MongoDB客户端；

docker exec -it mongo mongo

- 之后在admin集合中创建一个账号用于连接，这里创建的是基于root角色的超级管理员帐号；

use admin

db.createUser({

​    user: 'mongoadmin',

​    pwd: 'secret',

​    roles: [ { role: "root", db: "admin" } ] });

- 创建完成后验证是否可以登录；

db.auth("mongoadmin","secret")




