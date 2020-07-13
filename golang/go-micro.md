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

#### 1.项目结构

略

#### 2.修改 .proto 文件，增加双向流服务接口，并重新生成代码

```protobuf
// 略......

service HelloService{
  // 单向流
  rpc Hello(HelloRequest) returns (HelloResponse);

  // 双向流
  rpc HelloStream(HelloRequest) returns (stream HelloResponse);
}
```

#### 3.服务实现

```go
func (srv *HelloService) HelloStream(stream proto.HelloService_HelloStreamServer) error {
	for {
		req, err := stream.Recv()
		if err != nil {
			// 客户端关闭，服务端关闭流
			if err == io.EOF {
				return nil
			}
			return err
		}

		resp := &proto.HelloResponse{FullName: req.Name + " Si"}
		if err := stream.Send(resp); err != nil {
			return err
		}
	}
}
```



#### 4.服务端

略

#### 5.客户端

```go
package main

import (
	"context"
	"github/stone955/go-grpc/proto"
	"io"
	"log"
	"sync"

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

	// 获取客户端
	client := proto.NewHelloServiceClient(conn)

    // 单向流调用 略......

	// 双向流调用
	// 获取流
	stream, err := client.HelloStream(context.Background())
	if err != nil {
		log.Fatal(err)
	}

	// 确保发送线程，接收线程都结束才退出
	var wg sync.WaitGroup
	wg.Add(2)

	// 发送线程
	go func() {
		defer wg.Done()
		for i := 0; i < 10; i++ {
			if err := stream.Send(&proto.HelloRequest{Name: "stone "}); err != nil {
				log.Fatal(err)
			}
		}
	}()

	// 接收线程
	go func() {
		defer wg.Done()
		for i := 0; i < 10; i++ {
			resp, err := stream.Recv()
			if err != nil {
				if err == io.EOF {
					break
				}
				log.Fatal(err)
			}
			log.Printf("stream response fullname: %v\n", resp.FullName)
		}
	}()

	wg.Wait()
}
```



#### 6.运行

```bash
2020/06/17 19:35:03 stream response fullname: stone  Si
2020/06/17 19:35:03 stream response fullname: stone  Si
2020/06/17 19:35:03 stream response fullname: stone  Si
2020/06/17 19:35:03 stream response fullname: stone  Si
2020/06/17 19:35:03 stream response fullname: stone  Si
2020/06/17 19:35:03 stream response fullname: stone  Si
2020/06/17 19:35:03 stream response fullname: stone  Si
2020/06/17 19:35:03 stream response fullname: stone  Si
2020/06/17 19:35:03 stream response fullname: stone  Si
2020/06/17 19:35:03 stream response fullname: stone  Si
```



### 发布订阅

略

### 证书

#### 自签证书

- 生成服务端证书

```bash
openssl genrsa -out server.key 2048
openssl req -new -x509 -days 3650 -key server.key -out server.crt
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Heilongjiang
Locality Name (eg, city) []:Harbin
Organization Name (eg, company) [Internet Widgits Pty Ltd]:stone
Organizational Unit Name (eg, section) []:stone
Common Name (e.g. server FQDN or YOUR name) []:localhost
Email Address []:
```



- 生成客户端证书

```bash
openssl genrsa -out client.key 2048
openssl req -new -x509 -days 3650 -key client.key -out client.crt
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Heilongjiang
Locality Name (eg, city) []:Harbin
Organization Name (eg, company) [Internet Widgits Pty Ltd]:stone
Organizational Unit Name (eg, section) []:stone
Common Name (e.g. server FQDN or YOUR name) []:localhost
Email Address []:
```

- 服务端

```go
package main

import (
	"google.golang.org/grpc/credentials"
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
	cred, err := credentials.NewServerTLSFromFile("crt/server.crt", "crt/server.key")
	if err != nil {
		log.Fatal(err)
	}

	// 创建 grpc 服务
	s := grpc.NewServer(grpc.Creds(cred))
	// 略......
}
```



- 客户端

```go
package main

import (
	"context"
	"google.golang.org/grpc/credentials"
	"io"
	"log"
	"sync"

	"github/stone955/go-grpc/proto"
	"google.golang.org/grpc"
)

func main() {
	cred, err := credentials.NewClientTLSFromFile("crt/server.crt", "localhost")
	if err != nil {
		log.Fatal(err)
	}
	// 建立连接
	conn, err := grpc.Dial("localhost:8001", grpc.WithTransportCredentials(cred))
	if err != nil {
		log.Fatal(err)
	}

	defer func() {
		_ = conn.Close()
	}()

	// 略......
}
```



#### CA证书

- 生成根证书

```bash
openssl genrsa -out ca.key 2048
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Heilongjiang
Locality Name (eg, city) []:Harbin
Organization Name (eg, company) [Internet Widgits Pty Ltd]:stone
Organizational Unit Name (eg, section) []:stone
Common Name (e.g. server FQDN or YOUR name) []:localhost
Email Address []:
```



- 对服务端证书签名

```bash
openssl req -new -key server.key -out server.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Heilongjiang
Locality Name (eg, city) []:Harbin
Organization Name (eg, company) [Internet Widgits Pty Ltd]:stone
Organizational Unit Name (eg, section) []:stone
Common Name (e.g. server FQDN or YOUR name) []:localhost
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
openssl x509 -req -sha256 -CA ca.crt -CAkey ca.key -CAcreateserial -days 3650 -in server.csr -out server.crt
```



- 客户端

```go
package main

import (
	"context"
	"crypto/tls"
	"crypto/x509"
	"google.golang.org/grpc/credentials"
	"io"
	"io/ioutil"
	"log"
	"sync"

	"github/stone955/go-grpc/proto"
	"google.golang.org/grpc"
)

func main() {
	certificate, err := tls.LoadX509KeyPair("../crt/client.crt", "../crt/client.key")
	if err != nil {
		log.Fatal(err)
	}

	ca, err := ioutil.ReadFile("../crt/ca.crt")
	if err != nil {
		log.Fatal(err)
	}

	certPool := x509.NewCertPool()
	if ok := certPool.AppendCertsFromPEM(ca); !ok {
		log.Fatal(err)
	}

	cred := credentials.NewTLS(&tls.Config{
		Certificates: []tls.Certificate{certificate},
		RootCAs:      certPool,
		ServerName:   "localhost",
	})

	// 略......
}
```



### Token 认证

#### 1.实现grpc.PerRPCCredentials接口

```go
package auth

import "context"

type Authentication struct {
	User     string
	Password string
}

func (a *Authentication) GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error) {
	return map[string]string{
		"user":     a.User,
		"password": a.Password,
	}, nil
}

func (a *Authentication) RequireTransportSecurity() bool {
	return true
}
```

#### 2.客户端

```go
package main

import (
	"context"
	"crypto/tls"
	"crypto/x509"
	"github/stone955/go-grpc/internal/auth"
	"google.golang.org/grpc/credentials"
	"io"
	"io/ioutil"
	"log"
	"sync"

	"github/stone955/go-grpc/proto"
	"google.golang.org/grpc"
)

func main() {
	// 略......

	// 客户端信息
	a := &auth.Authentication{
		User:     "stone",
		Password: "123456",
	}

	// 建立连接
	conn, err := grpc.Dial("localhost:8001", grpc.WithTransportCredentials(cred), grpc.WithPerRPCCredentials(a))
	if err != nil {
		log.Fatal(err)
	}

	defer func() {
		_ = conn.Close()
	}()

	// 略......
}
```



#### 3.验证

```go
package auth

import (
	"context"
	"errors"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/metadata"
	"google.golang.org/grpc/status"
)

type Authentication struct {
	User     string
	Password string
}

func (a *Authentication) Auth(ctx context.Context) error {
	md, ok := metadata.FromIncomingContext(ctx)
	if !ok {
		return errors.New("missing credentials")
	}

	var (
		appID  string
		appKey string
	)

	if val, ok := md["user"]; ok {
		appID = val[0]
	}
	if val, ok := md["password"]; ok {
		appKey = val[0]
	}

	if appID != a.User || appKey != a.Password {
		return status.Errorf(codes.Unauthenticated, "invalid token")
	}
	return nil
}
```



#### 4.服务端

```go
package main

import (
	"github/stone955/go-grpc/internal/auth"
	"google.golang.org/grpc/credentials"
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
	cred, err := credentials.NewServerTLSFromFile("../crt/server.crt", "../crt/server.key")
	if err != nil {
		log.Fatal(err)
	}

	// 创建 grpc 服务
	s := grpc.NewServer(grpc.Creds(cred))
	
	// 注册服务
	proto.RegisterHelloServiceServer(s, &service.HelloService{
		// 加入身份验证
		Auth: &auth.Authentication{
			User:     "stone",
			Password: "123456",
		},
	})

	//  略......
}
```

#### 5.运行

```bash
// 验证失败
2020/06/18 17:00:53 rpc error: code = Unauthenticated desc = invalid token
// 验证成功
略
```



### 拦截器

#### 1.filter 函数

 ```go
// 普通方法拦截器
func unaryFilter(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (resp interface{}, err error) {
	log.Printf("unary filter  server= %v, fullmethod= %v\n", info.Server, info.FullMethod)
	return handler(ctx, req)
}

// 流失方法拦截器
func streamFilter(srv interface{}, ss grpc.ServerStream, info *grpc.StreamServerInfo, handler grpc.StreamHandler) error {
	log.Printf("stream filter  server= %v, fullmethod= %v\n", info.IsServerStream, info.FullMethod)
	return handler(srv, ss)
}
 ```

#### 2.服务端注册截取器

```go
// 添加截取器
s := grpc.NewServer(grpc.Creds(cred), grpc.UnaryInterceptor(unaryFilter), grpc.StreamInterceptor(streamFilter))
```



### 支持Web 服务



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