一.yum方式安装

1.下载yum源

cd ~

wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm

2.安装yum源

cd ~

yum localinstall mysql80-community-release-el7-3.noarch.rpm

3.检查yum源

yum repolist enabled | grep "mysql.*-community.*"

![img](C:/Users/stone/AppData/Local/YNote/data/qqF1FF3C9C0282220866BDCD152D18F42F/e1df28bf5dcc4226b5f35eaa960fc806/clipboard.png)

4.安装rpm包

yum install mysql-community-server

5.启动、停止、重启命令

systemctl start mysqld

systemctl stop mysqld

systemctl status mysqld

6.修改密码

grep 'temporary password' /var/log/mysqld.log

mysql -uroot -p 

ALTER USER 'root'@'localhost' IDENTIFIED BY '850417Sdy@';

GRANT ALL ON *.* TO 'root'@'%';

flush privileges;

7.更改加密方式

ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '850417Sdy@';



二.docker方式安装

1.安装并启动docker（略）

2.拉取 mysql 镜像

\# 如果失败就多试几次或配置国内镜像仓库

[root@localhost my-gin-blog]# docker pull mysql

Using default tag: latest

latest: Pulling from library/mysql

Digest: sha256:e1b0fd480a11e5c37425a2591b6fbd32af886bfc6d6f404bd362be5e50a2e632

Status: Image is up to date for mysql:latest

docker.io/library/mysql:latest



3.运行 mysql 容器

\# 这种方式没有挂载数据卷，容器每次重启或删除后数据会丢失

[root@localhost ~]# docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql --default-authentication-plugin=mysql_native_password

ab9b21764f583e58995e9f1466816d6295e5ec519c025b073cdf5898b4832c81

\# 进入容器命令行

docker exec -it 236b2624632d bash

\# 修改默认的密码验证插件

\# mysql -uroot -p 

\# ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';



\# 挂载数据卷

[root@localhost ~]# mkdir -p /root/docker-mysql/data

[root@localhost ~]# chmod -R 777 /root/docker-mysql/*

[root@localhost ~]# docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -v /root/docker-mysql/data:/var/lib/mysql -d mysql --default-authentication-plugin=mysql_native_password