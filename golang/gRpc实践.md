# gRpc 实践

[什么是gRPC](https://grpc.io/docs/what-is-grpc/introduction/)

## 概述

在 gRPC 中，客户端应用程序可以直接调用不同机器上的服务器应用程序上的方法，就像它是本地对象一样，使您可以更轻松地创建分布式应用程序和服务。与许多 RPC 系统一样，gRPC 基于定义服务的思想，指定可以通过参数和返回类型远程调用的方法。在服务器端，服务器实现了这个接口并运行一个 gRPC 服务器来处理客户端调用。在客户端，客户端有一个存根（在某些语言中简称为客户端），它提供与服务器相同的方法。

![](https://grpc.io/img/landing-2.svg)

gRPC 客户端和服务器可以在各种环境中运行和相互通信 - 从 Google 内部的服务器到您自己的桌面 - 并且可以用任何 gRPC 支持的语言编写。因此，例如，您可以轻松地使用 Java、Go、Python 或 Ruby 中的客户端创建一个 gRPC 服务器。此外，最新的 Google API 将具有其接口的 gRPC 版本，让您可以轻松地将 Google 功能构建到您的应用程序中。



> [快速入门](https://grpc.io/docs/languages/go/quickstart/)
> 
> [协议缓冲区编译器安装](https://www.grpc.io/docs/protoc-installation/)


## 准备工作：

### 首先需要安装proto 

1. 安装go [去下载](https://golang.google.cn/)
2. 安装协议缓存区编译器protobuf `brew install protobuf` 参考[协议缓冲区编译器安装](https://www.grpc.io/docs/protoc-installation/)
```
protoc --version # 确保版本为 3+
```
3. 安装protoc-gen-go-grpc `brew install protoc-gen-go-grpc`
4. 协议编译器的go插件
    - 使用以下命令为 Go 安装协议编译器插件
    ```bash
    go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.26
    go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.1
    ```
    - 更新您的，PATH以便protoc编译器可以找到插件：
    ```bash
    export PATH="$PATH:$(go env GOPATH)/bin"
    ```

## 开始

### 1. 用 protocol buffer 定义服务 `helloworld.proto`

例子如下：
```protobuf
syntax = "proto3";

option go_package = "google.golang.org/grpc/examples/helloworld/helloworld";

package helloworld;

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}

  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}

```

### 2. 编译成 go 文件
```bash
protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    helloworld/helloworld.proto
```

说明：

`--go_out=` go文件的生成路径

`--go-grpc_out=` grpc-go文件的生成路径

目录的末尾是proto文件的路径

这会生成 `helloworld/helloworld.pb.go ` 和 `helloworld/helloworld_grpc.pb.go` 文件， 其中包含：
- Code for populating, serializing, and retrieving HelloRequest and HelloReply message types.
- 生成 client 和 server 代码


生成后的文件如下：
```
.
├── greeter_client
│   └── main.go
├── greeter_server
│   └── main.go
└── helloworld
    ├── helloworld.pb.go
    ├── helloworld.proto
    └── helloworld_grpc.pb.go
```