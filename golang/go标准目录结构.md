# go 标准目录结构
目录结构参考了[go standard project layout](https://github.com/golang-standards/project-layout/blob/master/README_zh.md)项目的推荐

从上往下
- `makefile`是整个工程构建的一个入口，可以理解相应的构建的命令都是统一封装，会相应的构建，都可以通过一些封装指令去做相应的执行。
- `build`目录主要会放工程相应的镜像构建的配置文件，比如`Dockerfile`
- `cmd`可以理解成整个工程的主干入口，每个服务使用`cobra`拆分成不同的`subcommand`
- `conf`目录存放整个工程的配置文件
- `idl`目录会存放`thrift`或`protobuf`定义的服务
- `internal`是业务逻辑层，里面会简单做一些分层；
  - `handler` 统一放handler
  - `server` 是服务的初始化
  - `service` 是服务启动相关的一些逻辑
- `Mainfest`是工程部署相关的配置文件，正常部署推荐使用`Helm Charts`
- `pkg`可以理解成高度封装，外部可以服用的并且跟业务逻辑没有任何耦合的包

### Makefile规范设计
要求每个项目的 `Makefile`得包含一些必要的元素，比如支持**代码检查**，也能支持执行**单元测试**，支持去做跨平台编译的二进制**构建**，支持容器的**镜像构建**
