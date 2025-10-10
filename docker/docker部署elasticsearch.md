# docker 部署 elasticsearch

## 拉取 elasticsearch 镜像
```bash
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.2.0
```

## 使用Docker启动单节点集群

如果您在 Docker 容器中启动单节点 Elasticsearch 集群，则会自动为您启用和配置安全性。首次启动 Elasticsearch 时，会自动进行以下安全配置

- 为传输层和 HTTP 层生成 证书和密钥。
- 传输层安全 (TLS) 配置设置被写入 elasticsearch.yml.
- 为elastic用户生成密码。
- 为 Kibana 生成一个注册令牌。

以下命令启动单节点 Elasticsearch 集群以进行开发或测试。


1. 为 Elasticsearch 和 Kibana 创建一个新的 docker 网络
```bash
docker network create elastic
```
2. 在 Docker 中启动 Elasticsearch。为elastic用户生成密码并输出到终端，以及用于注册 Kibana 的注册令牌。
```bash
docker run --name es01 --net elastic -p 9200:9200 -p 9300:9300 -it docker.elastic.co/elasticsearch/elasticsearch:8.2.0
```
> 您可能需要在终端中向后滚动一点才能查看密码和注册令牌

3. 复制生成的密码和注册令牌并将其保存在安全位置。这些值仅在您第一次启动 Elasticsearch 时显示。
> 如果您需要为elastic用户或其他内置用户重置密码，请运行该elasticsearch-reset-password工具。该工具/bin在 Docker 容器的 Elasticsearch 目录中可用。例如：

```bash
docker exec -it es01 /usr/share/elasticsearch/bin/ elasticsearch -reset -password
```

4. 将http_ca.crt安全证书从 Docker 容器复制到本地计算机。
```bash
docker cp es01:/usr/share/elasticsearch/config/certs/http_ca.crt .
```
5. http_ca.crt打开一个新终端，并使用从 Docker 容器复制的文件进行经过身份验证的调用，验证您是否可以连接到 Elasticsearch 集群。elastic出现提示时输入用户的密码。
```bash
curl --cacert http_ca.crt -u elastic https://localhost:9200
```