## 安装 Docker


### 卸载旧版本

````bash
yum remove -y docker \

docker-client \

docker-client-latest \

docker-common \

docker-latest \

docker-latest-logrotate \

docker-logrotate \

docker-selinux \

docker-engine-selinux \

docker-engine

````

### 设置 yum repository

````bash
yum install -y yum-utils device-mapper-persistent-data lvm2

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
````

### 配置镜像加速器

````bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://joowf6ge.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
````
### 安装并启动 docker

````bash
yum install -y docker-ce{-18.09.9} 

systemctl enable docker

systemctl status docker

systemctl start docker

systemctl stop docker

systemctl restart docker
````
