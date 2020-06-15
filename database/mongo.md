一.原生安装

1.创建yum源

（1）创建/etc/yum.repos.d/mongodb-org-4.2.repo

（2）写入以下内容：

[mongodb-org-4.2]

name=MongoDB Repository

baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.2/x86_64/

gpgcheck=1

enabled=1

gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc



2.yum安装

sudo yum install -y mongodb-org



3.ulimit Settings

ulimit -f unlimited

ulimit -t unlimited

ulimit -v unlimited

ulimit -l unlimited

ulimit -n 64000

ulimit -m unlimited

ulimit -u 64000



4.启动、停止、重启

systemctl start mongod

systemctl stop mongod

systemctl restart mongod

systemctl status mongod

systemctl enable mongod

systemctl disable mongod



5.配置文件

/etc/mongod.conf



二.镜像安装

  ● 下载MongoDB的Docker镜像；

docker pull mongo:4.2.5

  ● 使用Docker命令启动MongoDB服务；

docker run -p 27017:27017 --name mongo \

-v /mydata/mongo/db:/data/db \

-d mongo:4.2.5

  ● 有时候我们需要为MongoDB设置账号，可以使用如下命令启动；

docker run -p 27017:27017 --name mongo \

-v /mydata/mongo/db:/data/db \

-d mongo:4.2.5 --auth

  ● 然后我们需要进入容器中的MongoDB客户端；

docker exec -it mongo mongo

  ● 之后在admin集合中创建一个账号用于连接，这里创建的是基于root角色的超级管理员帐号；

use admin

db.createUser({

​    user: 'mongoadmin',

​    pwd: 'secret',

​    roles: [ { role: "root", db: "admin" } ] });

  ● 创建完成后验证是否可以登录；

db.auth("mongoadmin","secret")