## 搭建环境

### 1.git 配置

```bash
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
git config --global ttp.sslVerify "false"
git config --global http.postBuffer 524288000
```

### 2.hosts （不一定好使）

```
140.82.114.4 github.com

185.199.108.153 assets-cdn.github.com
# 185.199.109.153 assets-cdn.github.com
# 185.199.110.153 assets-cdn.github.com
# 185.199.111.153 assets-cdn.github.com

199.232.69.194 github.global.ssl.fastly.net

ipconfig /flushdns
```

### 2. 直接安装（国内一般不会成功）
#### 2.1 安装 protobuf
```bash
go get -v -u github.com/golang/protobuf/protoc-gen-go
```
#### 2.2 安装 grpc
```bash
go get -u google.golang.org/grpc
```

### 3. 间接安装
#### 3.1 安装 protobuf
```bash
cd %GOPATH%/src
git clone https://github.com/golang/protobuf.git github.com/golang/protobuf
go install github.com/golang/protobuf/protoc-gen-go
```
#### 3.2 安装 grpc

```bash
cd %GOPATH%/src
git clone https://github.com/golang/net.git golang.org/x
git clone https://github.com/golang/text.git golang.org/x
git clone https://github.com/golang/sys.git golang.org/x
git clone https://github.com/golang/tools.git golang.org/x
git clone https://github.com/golang/crypto.git golang.org/x
git clone https://github.com/golang/time.git golang.org/x
git clone https://github.com/golang/lint.git golang.org/x
git clone https://github.com/golang/oauth2.git golang.org/x
git clone https://github.com/protocolbuffers/protobuf-go.git google.golang.org/protobuf
git clone https://github.com/googleapis/go-genproto.git google.golang.org/genproto
(git clone https://gitee.com/mirrors/go-genproto.git google.golang.org/genproto)
git clone https://github.com/grpc/grpc-go.git google.golang.org/grpc
go install google.golang.org/grpc
```

### 4. 安装编译工具

https://github.com/google/protobuf/releases 配置环境变量

## 编写 .proto 文件

```protobuf
syntax = "proto3";

message HelloRequest{
  string name = 1;
}

message HelloResponse{
  string fullName = 1;
}

service HelloService{
  rpc Hello(HelloRequest)returns(HelloResponse);
}
```



## 生成代码

protoc --proto_path={.proto文件目录} --go_out=plugins=grpc:{生成代码路径} {.proto文件路径}

```bash
protoc --proto_path=./proto --go_out=plugins=grpc:./proto hello.proto
```

## 简单示例

### 1.项目结构

### 2.定义 .proto 文件

### 3.服务端

### 4.客户端

### 5.运行

