## 使用 Docker 搭建

### 拉取镜像

````bash
docker pull wurstmeister/kafka
docker pull wurstmeister/zookeeper
````

### 启动 zookeeper 容器

wurstmeister/zookeeper镜像拥有默认命令 /usr/sbin/sshd && bash /usr/bin/start-zk.sh，所以只需启动一个守护式容器即可

````bash
docker run --name zookeeper -p 12181:2181 -d wurstmeister/zookeeper:latest
````

### 启动 3 个 kafka 容器

````bash
docker run -p 19092:9092 --name kafka1 -d -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=宿主机IP:12181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://宿主机IP:19092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 wurstmeister/kafka:latest

docker run -p 19093:9093 --name kafka2 -d -e KAFKA_BROKER_ID=1 -e KAFKA_ZOOKEEPER_CONNECT=宿主机IP:12181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://宿主机IP:19093 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 wurstmeister/kafka:latest

docker run -p 19094:9094 --name kafka3 -d -e KAFKA_BROKER_ID=2 -e KAFKA_ZOOKEEPER_CONNECT=宿主机IP:12181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://宿主机IP:19094 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 wurstmeister/kafka:latest
````

### 进入容器内命令

````bash
docker exec -i -t kafka3 /bin/bash
````



https://www.cnblogs.com/answerThe/p/11267129.html