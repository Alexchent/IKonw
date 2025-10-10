# docker 部署mysql

## 方式一
1. 拉取mysql镜像 `docker pull mysql:latest`
2. 查看本地容器  `docker images`
3. 运行容器 
```bash
docker run -itd --name mysql-docker -p 3306:3306 \
  -v ./conf:/etc/mysql/conf.d \
  -e MYSQL_ROOT_PASSWORD=123456 mysql 
```

参数说明：

- -p 3306:3306 ：映射容器服务的 3306 端口到宿主机的 3306 端口，外部主机可以直接通过 宿主机ip:3306 访问到 MySQL 的服务。
- MYSQL_ROOT_PASSWORD=123456：设置 MySQL 服务 root 用户的密码。

4. 进入容器 `docker exec -it mysql bash`
5. 登录mysql
```
mysql -u root -p
```

## 方式二

生成 docker-compose.yml 文件
```
version: '3.1'

services:
  db:
    image: mysql
    restart: unless-stopped
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: blog
      MYSQL_USER: 'test'
      MYSQL_PASS: 'yourpassword'
    volumes:
    - ./data:/var/lib/mysql
    - ./conf:/etc/mysql/conf.d
    - ./conf/my.cnf:/etc/my.cnf
    # 数据库还原目录 可将需要还原的sql文件放在这里
    - ./source:/docker-entrypoint-initdb.d
    ports:
      - "3306:3306"
    container_name: docker-mysql
```