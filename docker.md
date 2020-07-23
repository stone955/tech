## 安装 Docker


### 卸载旧版本

````bash
yum remove -y docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-selinux docker-engine-selinux docker-engine

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
````
### 安装并启动 docker

````bash
yum install -y docker-ce

systemctl enable docker

systemctl status docker

systemctl start docker

systemctl stop docker

systemctl restart docker
````







\# 安装 Kubernetes 单Master节点



\# 检查 centos / hostname(所有节点)

\```

\# 在 master 节点和 worker 节点都要执行

cat /etc/redhat-release



\# 此处 hostname 的输出将会是该机器在 Kubernetes 集群中的节点名字

\# 不能使用 localhost 作为节点的名字

hostname



\# 请使用 lscpu 命令，核对 CPU 信息

\# Architecture: x86_64    本安装文档不支持 arm 架构

\# CPU(s):       2         CPU 内核数量不能低于 2

lscpu

\```



\#修改 hostname(所有节点)

\```

\# 修改 hostname

hostnamectl set-hostname your-new-host-name

\# 查看修改结果

hostnamectl status

\# 设置 hostname 解析

echo "127.0.0.1   $(hostname)" >> /etc/hosts

\```



\#安装 docker / kubelet(所有节点)

\```

\# 在 master 节点和 worker 节点都要执行



\# 安装 docker

\# 参考文档如下

\# https://docs.docker.com/install/linux/docker-ce/centos/ 

\# https://docs.docker.com/install/linux/linux-postinstall/



\# 卸载旧版本

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



\# 设置 yum repository

yum install -y yum-utils device-mapper-persistent-data lvm2

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo



\# 安装并启动 docker

yum install -y docker-ce{-18.09.9} 

systemctl enable docker

systemctl start docker



\# 安装 nfs-utils

\# 必须先安装 nfs-utils 才能挂载 nfs 网络存储

yum install -y nfs-utils



\# 关闭 防火墙

systemctl stop firewalld

systemctl disable firewalld



\# 关闭 SeLinux

setenforce 0

sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config



\# 关闭 swap

swapoff -a

yes | cp /etc/fstab /etc/fstab_bak

cat /etc/fstab_bak |grep -v swap > /etc/fstab



\# 修改 /etc/sysctl.conf

\# 如果有配置，则修改

sed -i "s#^net.ipv4.ip_forward.*#net.ipv4.ip_forward=1#g"  /etc/sysctl.conf

sed -i "s#^net.bridge.bridge-nf-call-ip6tables.*#net.bridge.bridge-nf-call-ip6tables=1#g"  /etc/sysctl.conf

sed -i "s#^net.bridge.bridge-nf-call-iptables.*#net.bridge.bridge-nf-call-iptables=1#g"  /etc/sysctl.conf

\# 可能没有，追加

echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf

echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf

echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf

\# 执行命令以应用

sysctl -p



\# 配置K8S的yum源

cat <<EOF > /etc/yum.repos.d/kubernetes.repo

[kubernetes]

name=Kubernetes

baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64

enabled=1

gpgcheck=0

repo_gpgcheck=0

gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

EOF



\# 卸载旧版本

yum remove -y kubelet kubeadm kubectl



\# 安装kubelet、kubeadm、kubectl

yum install -y kubelet kubeadm kubectl



\# 修改docker Cgroup Driver为systemd

\# # 将/usr/lib/systemd/system/docker.service文件中的这一行 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

\# # 修改为 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd

\# 如果不修改，在添加 worker 节点时可能会碰到如下错误

\# [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". 

\# Please follow the guide at https://kubernetes.io/docs/setup/cri/

sed -i "s#^ExecStart=/usr/bin/dockerd.*#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd#g" /usr/lib/systemd/system/docker.service



\# 设置 docker 镜像，提高 docker 镜像下载速度和稳定性

\# 如果您访问 https://hub.docker.io 速度非常稳定，亦可以跳过这个步骤

vi /etc/docker/daemon.json



{

​    "registry-mirrors": ["https://registry.docker-cn.com"]

}



\# 重启 docker，并启动 kubelet

systemctl daemon-reload

systemctl restart docker

systemctl enable kubelet 

systemctl start kubelet



docker version



如果此时执行 service status kubelet 命令，将得到 kubelet 启动失败的错误提示，请忽略此错误，因为必须完成后续步骤中 kubeadm init 的操作，kubelet 才能正常启动



\```



\# 初始化 master 节点(只在master节点)

\## 关于初始化时用到的环境变量

APISERVER_NAME 不能是 master 的 hostname

APISERVER_NAME 必须全为小写字母、数字、小数点，不能包含减号

POD_SUBNET 所使用的网段不能与 master节点/worker节点 所在的网段重叠。该字段的取值为一个 CIDR 值，如果您对 CIDR 这个概念还不熟悉，请仍然执行 export POD_SUBNET=10.100.0.1/16 命令，不做修改



\```

\# 只在 master 节点执行

\# 替换 x.x.x.x 为 master 节点实际 IP（请使用内网 IP）

\# export 命令只在当前 shell 会话中有效，开启新的 shell 窗口后，如果要继续安装过程，请重新执行此处的 export 命令

export MASTER_IP=x.x.x.x

\# 替换 apiserver.demo 为 您想要的 dnsName

export APISERVER_NAME=apiserver.demo

\# Kubernetes 容器组所在的网段，该网段安装完成后，由 kubernetes 创建，事先并不存在于您的物理网络中

export POD_SUBNET=10.100.0.1/16

echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts



\# 查看完整配置选项 https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2

rm -f ./kubeadm-config.yaml

cat <<EOF > ./kubeadm-config.yaml

apiVersion: kubeadm.k8s.io/v1beta2

kind: ClusterConfiguration

kubernetesVersion: v1.16.0

imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers

controlPlaneEndpoint: "${APISERVER_NAME}:6443"

networking:

  serviceSubnet: "10.96.0.0/16"

  podSubnet: "${POD_SUBNET}"

  dnsDomain: "cluster.local"

EOF





\# kubeadm init

\# 根据您服务器网速的情况，您需要等候 3 - 10 分钟

kubeadm init --config=kubeadm-config.yaml --upload-certs



\# 配置 kubectl

rm -rf /root/.kube/

mkdir /root/.kube/

cp -i /etc/kubernetes/admin.conf /root/.kube/config



\# 安装 calico 网络插件

\# 参考文档 https://docs.projectcalico.org/v3.8/getting-started/kubernetes/

rm -f calico.yaml

wget https://docs.projectcalico.org/v3.8/manifests/calico.yaml

sed -i "s#192\.168\.0\.0/16#${POD_SUBNET}#" calico.yaml

kubectl apply -f calico.yaml



\```



\# 检查 master 初始化结果(只在master节点)



\```

\# 执行如下命令，等待 3-10 分钟，直到所有的容器组处于 Running 状态

watch kubectl get pod -n kube-system -o wide



\# 查看 master 节点初始化结果

kubectl get nodes -o wide

\```



\# 初始化 worker节点(只在master节点)



\```

\# 只在 master 节点执行

kubeadm token create --print-join-command

\```



\# 初始化worker(只在所有worker节点)



\```

\# 只在 worker 节点执行

\# 替换 x.x.x.x 为 master 节点实际 IP（请使用内网 IP）

export MASTER_IP=x.x.x.x

\# 替换 apiserver.demo 为初始化 master 节点时所使用的 APISERVER_NAME

export APISERVER_NAME=apiserver.demo

echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts



\# 替换为 master 节点上 kubeadm token create 命令的输出

kubeadm join apiserver.k8s:6443 --token 6yb5bp.5l62g3gjnemqsag7     --discovery-token-ca-cert-hash sha256:044876a918799092a4b37f259202eb3dc5a419da054db18448a0dc3c515c5901

\```



\# 检查初始化结果



\```

\# 只在 master 节点执行

kubectl get nodes -o wide

\```



\# 移除 worker 节点



\```

\# 只在 worker 节点执行

kubeadm reset



\# 只在 master 节点执行

kubectl delete node demo-worker-x-x

\```





\# 安装 Kuboard



\```

\# 在master节点执行

kubectl apply -f https://kuboard.cn/install-script/kuboard.yaml

\```



\# 获取Token



\# 管理员用户



\```

kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kuboard-user | awk '{print $1}')   

\```

\# 只读用户



\```

kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kuboard-viewer | awk '{print $1}')

\```



\# 访问Kuboard



\# 通过NodePort访问

http://任意一个Worker节点的IP地址:32567

输入前一步骤中获得的 token，可进入 Kuboard 集群概览页









\# 实战：部署 nginx Deployment



\# 1.使用 kubectl

\```

\# 创建文件 nginx-deployment.yaml，内容如下：

apiVersion: apps/v1	#与k8s集群版本有关，使用 kubectl api-versions 即可查看当前集群支持的版本

kind: Deployment	#该配置的类型，我们使用的是 Deployment

metadata:	        #译名为元数据，即 Deployment 的一些基本属性和信息

  name: nginx-deployment	#Deployment 的名称

  labels:	    #标签，可以灵活定位一个或多个资源，其中key和value均可自定义，可以定义多组，目前不需要理解

​    app: nginx	#为该Deployment设置key为app，value为nginx的标签

spec:	        #这是关于该Deployment的描述，可以理解为你期待该Deployment在k8s中如何使用

  replicas: 1	#使用该Deployment创建一个应用程序实例

  selector:	    #标签选择器，与上面的标签共同作用，目前不需要理解

​    matchLabels: #选择包含标签app:nginx的资源

​      app: nginx

  template:	    #这是选择或创建的Pod的模板

​    metadata:	#Pod的元数据

​      labels:	#Pod的标签，上面的selector即选择包含标签app:nginx的Pod

​        app: nginx

​    spec:	    #期望Pod实现的功能（即在pod中部署）

​      containers:	#生成container，与docker中的container是同一种

​      \- name: nginx	#container的名称

​        image: nginx:1.7.9	#使用镜像nginx:1.7.9创建container，该container默认80端口可访问

​        

\# 应用 YAML 文件   

kubectl apply -f nginx-deployment.yaml



\# 查看 Deployment

kubectl get deployments



\# 查看 Pod

kubectl get pods



\```

\# 2.使用 Kuboard









\# 实战：故障排除



\```

\# kubectl get 资源类型



\#获取类型为Deployment的资源列表

kubectl get deployments



\#获取类型为Pod的资源列表

kubectl get pods



\#获取类型为Node的资源列表

kubectl get nodes



\# kubectl describe 资源类型 资源名称



\#查看名称为nginx-XXXXXX的Pod的信息

kubectl describe pod nginx-XXXXXX	



\#查看名称为nginx-XXXXXXX的Deployment的信息

kubectl describe deployment nginx-XXXXXXX



\# kubectl logs Pod名称



\#查看名称为nginx-pod-XXXXXXX的Pod内的容器打印的日志

kubectl logs -f nginx-pod-XXXXXXX



\# kubectl exec Pod名称 操作命令



\# 在名称为nginx-pod-xxxxxx的Pod中运行bash

kubectl exec -it nginx-pod-xxxxxx /bin/bash

\```



\# 实战：为nginx Deployment 创建一个 Service



\```

\# 创建文件 nginx-service.yaml

vi nginx-service.yaml



\# 文件内容如下：

apiVersion: v1

kind: Service

metadata:

  name: nginx-service

  labels:

​    app: nginx

spec:

  selector:

​    app: nginx

  ports:

  \- name: nginx-port

​    protocol: TCP

​    port: 80

​    nodePort: 32600

​    targetPort: 80

  type: NodePort



\# 执行命令

kubectl apply -f nginx-service.yaml



\# 检查执行结果

kubectl get services -o wide



\# 访问服务

curl <任意节点的 IP>:32600



\```



\# 实战：将 nginx Deployment 扩容到 2 个副本

\## 修改 nginx-deployment.yaml 文件（只在master节点）

将 replicas 修改为 4



\```

kubectl apply -f nginx-deployment.yaml

\```



\```

watch kubectl get pods -o wide

\```





\# 实战：滚动更新 nginx Deployment

\## 修改 nginx-deployment.yaml 文件

\```

\#使用镜像nginx:1.8替换原来的nginx:1.7.9

\```



\```

kubectl apply -f nginx-deployment.yaml

\```



\```

watch kubectl get pods -o wide

\```