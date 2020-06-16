## grpc

### 搭建环境

#### 1.git 配置

```bash
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
git config --global ttp.sslVerify "false"
git config --global http.postBuffer 524288000
// 临时开启
go env -w GO111MODULE=on
// 恢复默认
go env -w GO111MODULE=
```

#### 2.hosts （不一定好使）

```
140.82.114.4 github.com

185.199.108.153 assets-cdn.github.com
# 185.199.109.153 assets-cdn.github.com
# 185.199.110.153 assets-cdn.github.com
# 185.199.111.153 assets-cdn.github.com

199.232.69.194 github.global.ssl.fastly.net

ipconfig /flushdns
```

#### 2. 直接安装（国内一般不会成功）
##### 2.1 安装 protobuf
```bash
go get -v -u github.com/golang/protobuf/protoc-gen-go
```
##### 2.2 安装 grpc
```bash
go get -u google.golang.org/grpc
```

#### 3. 间接安装
##### 3.1 安装 protobuf
```bash
cd %GOPATH%/src
git clone https://github.com/golang/protobuf.git github.com/golang/protobuf
go install github.com/golang/protobuf/protoc-gen-go
```
##### 3.2 安装 grpc
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

#### 4. 安装编译工具

https://github.com/google/protobuf/releases 配置环境变量

### 编写 .proto 文件

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

### 生成代码

protoc --proto_path={.proto文件目录} --go_out=plugins=grpc:{生成代码路径} {.proto文件路径}

```bash
protoc --proto_path=./proto --go_out=plugins=grpc:./proto hello.proto
```

### 简单示例

#### 1.项目结构

|-- go-grpc (项目名)

	|-- client （客户端）
	
	|-- internal 
	
		|-- service （接口实现）
	
		|-- models （数据层）
	
	|-- proto （.proto文件及生成的代码）
	
	|-- server （服务端）

#### 2.定义 .proto 文件

```protobuf
syntax = "proto3";

option go_package = ".;proto";

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

#### 3.服务实现

```go
package service

import (
   "context"
   "github/stone955/go-grpc/proto"
)

type HelloService struct{}

func (srv *HelloService) Hello(context.Context, *proto.HelloRequest) (*proto.HelloResponse, error) {
   resp := proto.HelloResponse{
      FullName: "Si Dong yu",
   }
   return &resp, nil
}
```

#### 4.服务端

```go
package main

import (
   "log"
   "net"
   "os"
   "os/signal"
   "sync"

   "github/stone955/go-grpc/internal/service"
   "github/stone955/go-grpc/proto"
   "google.golang.org/grpc"
)

func main() {
   // 监听端口
   listen, err := net.Listen("tcp", ":8001")
   if err != nil {
      log.Fatal(err)
   }
   // 创建 grpc 服务
   s := grpc.NewServer()
   // 注册服务
   proto.RegisterHelloServiceServer(s, &service.HelloService{})

   //  监听退出信号，优雅关闭
   var wg sync.WaitGroup
   ch := make(chan os.Signal, 1)
   signal.Notify(ch)

   go func() {
      wg.Add(1)
      defer wg.Done()
      select {
      case <-ch:
         s.GracefulStop()
      }
   }()

   // 启动服务
   if err := s.Serve(listen); err != nil {
      log.Fatal(err)
   }

   wg.Wait()

   log.Println("server graceful stopped")
}
```

#### 5.客户端

```go
package main

import (
   "context"
   "github/stone955/go-grpc/proto"
   "log"

   "google.golang.org/grpc"
)

func main() {
   // 建立连接
   conn, err := grpc.Dial("localhost:8001", grpc.WithInsecure())
   if err != nil {
      log.Fatal(err)
   }

   defer func() {
      _ = conn.Close()
   }()

   client := proto.NewHelloServiceClient(conn)

   request := &proto.HelloRequest{
      Name: "sdy",
   }

   resp, err := client.Hello(context.TODO(), request)
   if err != nil {
      log.Fatal(err)
   }
   log.Printf("response fullname: %v\n", resp.FullName)
}
```

#### 6.运行

```bash
D:\Project\go\stone\go-grpc>go run server/server.go

D:\Project\go\stone\go-grpc>go run client/client.go
2020/06/16 11:35:55 response fullname: Si Dong yu
```



### 流式服务



### 发布订阅



### 证书



### 认证





---

## go-micro

### 搭建环境

#### 1.安装 go-micro

```bash
go get -u -v github.com/micro/go-micro
go get -u -v github.com/micro/protoc-gen-micro
```

#### 2.安装 micro

```bash
go get -u -v github.com/micro/micro
```

### 生成代码

protoc --proto_path={.proto文件目录} --micro_out={.proto文件目录} --go_out={生成代码路径} {.proto文件名}

```bash
protoc --proto_path=./proto --micro_out=./proto --go_out=./proto hello.proto
```

### 简单示例

#### 1.项目结构

#### 2.定义 .proto 文件

#### 3.服务实现

#### 4.服务端

#### 5.客户端

#### 6.运行