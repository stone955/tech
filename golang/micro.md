设置环境变量

git config --global http.postBuffer  524288000

go env -w GO111MODULE=on

go env -w GOPROXY=https://goproxy.cn,direct



安装依赖 (开启go mod模式可以省略)

Linux:

git clone https://github.com/golang/net.git ~/go/src/golang.org/x/net

git clone https://github.com/golang/text.git ~/go/src/golang.org/x/text

git clone https://github.com/golang/sys.git ~/go/src/golang.org/x/sys

git clone https://github.com/golang/crypto.git ~/go/src/golang.org/x/crypto

git clone https://github.com/golang/lint.git ~/go/src/golang.org/x/lint

git clone https://github.com/golang/tools.git ~/go/src/golang.org/x/tools

git clone https://github.com/golang/time.git ~/go/src/golang.org/x/time

git clone https://github.com/grpc/grpc-go.git ~/go/src/google.golang.org/grpc

git clone https://github.com/google/go-genproto.git ~/go/src/google.golang.org/genproto



Windows:

git clone https://github.com/golang/net.git C:\Users\stone\go/src/golang.org/x/net

git clone https://github.com/golang/text.git C:\Users\stone\go/src/golang.org/x/text

git clone https://github.com/golang/sys.git C:\Users\stone\go/src/golang.org/x/sys

git clone https://github.com/golang/crypto.git C:\Users\stone\go/src/golang.org/x/crypto

git clone https://github.com/golang/lint.git C:\Users\stone\go/src/golang.org/x/lint

git clone https://github.com/golang/tools.git C:\Users\stone\go/src/golang.org/x/tools

git clone https://github.com/golang/time.git C:\Users\stone\go/src/golang.org/x/time

git clone https://github.com/grpc/grpc-go.git C:\Users\stone\go/src/google.golang.org/grpc

cd C:\Users\stone\go/src

go install google.golang.org/grpc

git clone https://github.com/google/go-genproto.git C:\Users\stone\go/src/google.golang.org/genproto



安装protoc的go插件

go get -u -v github.com/golang/protobuf/proto

go get -u -v github.com/golang/protobuf/protoc-gen-go



安装protobuf工具

https://github.com/google/protobuf/releases

配置protoc环境变量



安装go-micro框架

go get -u -v github.com/micro/go-micro

go get -u -v github.com/micro/protoc-gen-micro



安装micro工具包

go get -u -v github.com/micro/micro



生成代码

protoc --proto_path={.proto文件目录} --micro_out={.proto文件目录} --go_out={生成代码路径} {.proto文件名}

示例：protoc --proto_path=./hello/proto --micro_out=./hello/proto --go_out=./hello/proto hello.proto