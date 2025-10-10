# docker-compose

命令行 `docker-compose -h|--help` 可以查看到详细信息

```
docker-compose [-f <arg>...] [--profile <name>...] [options] [--] [COMMAND] [ARGS...]
```

## 常见 options

- `-f, --file FILE` 指定compose文件(default: docker-compose.yml)
- `-p, --project-name NAME` 指定项目名(default: directory name)

## 常用Commands
```
Commands:
  build              创建或重建 services
  config             验证和查看compose文件
  
  up                 Create and start containers
  down               Stop and remove containers, networks, images, and volumes
  
  events             Receive real time events from containers
  exec               在一个正在运行中的容器内执行一个命令
  help               Get help on a command
  images             List images
  kill               Kill containers
  logs               View output from containers
  pause              暂停 services
  port               Print the public port for a port binding
  ps                 List containers
  pull               Pull service images
  push               Push service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  top                Display the running processes
  unpause            Unpause services

  version            Show the Docker-Compose version 
```

### 1. docker-compose config 
验证和查看compose文件

### 2. docker-compose up -d
创建并在后台开始containers

### 3. docker-compose down
停止并删除images，containers，networks，volumes
