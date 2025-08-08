# elaticsearch 快手入门

## 写入数据

您将数据作为称为文档的 JSON 对象添加到 Elasticsearch。Elasticsearch 将这些文档存储在可搜索的索引中。

对于时间序列数据，例如日志和指标，您通常将文档添加到由多个自动生成的支持索引组成的数据流中。

数据流需要与其名称匹配的索引模板。Elasticsearch 使用这个模板来配置流的后备索引。发送到数据流的文档必须有一个@timestamp字段。

### 添加单个文档编辑
提交以下索引请求以将单个日志条目添加到 logs-my_app-default数据流。由于logs-my_app-default不存在，请求使用内置logs-*-*索引模板自动创建它。
```
POST logs-my_app-default/_doc
{
  "@timestamp": "2099-05-06T16:21:15.000Z",
  "event": {
    "original": "192.0.2.42 - - [06/May/2099:16:21:15 +0000] \"GET /images/bg.jpg HTTP/1.0\" 200 24736"
  }
}
```

响应包括 Elasticsearch 为文档生成的元数据：

- _index包含文档 的支持。Elasticsearch 会自动生成支持索引的名称。
- _id索引中文档 的唯一性。

```
{
  "_index": ".ds-logs-my_app-default-2099-05-06-000001",
  "_id": "gl5MJXMBMk1dGnErnBW8",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

### 添加多个文档
使用_bulk端点在一个请求中添加多个文档。批量数据必须是换行符分隔的 JSON (NDJSON)。每行必须以换行符 ( \n) 结尾，包括最后一行。
```
PUT logs-my_app-default/_bulk
{ "create": { } }
{ "@timestamp": "2099-05-07T16:24:32.000Z", "event": { "original": "192.0.2.242 - - [07/May/2020:16:24:32 -0500] \"GET /images/hm_nbg.jpg HTTP/1.0\" 304 0" } }
{ "create": { } }
{ "@timestamp": "2099-05-08T16:25:42.000Z", "event": { "original": "192.0.2.255 - - [08/May/2099:16:25:42 +0000] \"GET /favicon.ico HTTP/1.0\" 200 3638" } }
```

## 搜索数据
索引文档可用于近乎实时的搜索。以下搜索匹配所有日志条目logs-my_app-default并按降序对它们进行 @timestamp排序。

```
GET logs-my_app-default/_search
{
  "query": {
    "match_all": { }
  },
  "sort": [
    {
      "@timestamp": "desc"
    }
  ]
}
```

默认情况下，hits响应部分最多包含与搜索匹配的前 10 个文档。每个_source命中的 包含索引期间提交的原始 JSON 对象。
```
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": null,
    "hits": [
      {
        "_index": ".ds-logs-my_app-default-2099-05-06-000001",
        "_id": "PdjWongB9KPnaVm2IyaL",
        "_score": null,
        "_source": {
          "@timestamp": "2099-05-08T16:25:42.000Z",
          "event": {
            "original": "192.0.2.255 - - [08/May/2099:16:25:42 +0000] \"GET /favicon.ico HTTP/1.0\" 200 3638"
          }
        },
        "sort": [
          4081940742000
        ]
      },
      ...
    ]
  }
}
```

### 获取特定字段
对于大型文档，解析整个_source文件很笨拙。要将其从响应中排除，请将_source参数设置为false。相反，使用fields 参数来检索您想要的字段。
```
GET logs-my_app-default/_search
{
  "query": {
    "match_all": { }
  },
  "fields": [
    "@timestamp"
  ],
  "_source": false,
  "sort": [
    {
      "@timestamp": "desc"
    }
  ]
}
```
响应包含每个命中的fields值作为平面数组。
```
{
  ...
  "hits": {
    ...
    "hits": [
      {
        "_index": ".ds-logs-my_app-default-2099-05-06-000001",
        "_id": "PdjWongB9KPnaVm2IyaL",
        "_score": null,
        "fields": {
          "@timestamp": [
            "2099-05-08T16:25:42.000Z"
          ]
        },
        "sort": [
          4081940742000
        ]
      },
      ...
    ]
  }
}
```

### 搜索日期范围
要在特定时间或 IP 范围内搜索，请使用range查询。
```
GET logs-my_app-default/_search
{
  "query": {
    "range": {
      "@timestamp": {
        "gte": "2099-05-05",
        "lt": "2099-05-08"
      }
    }
  },
  "fields": [
    "@timestamp"
  ],
  "_source": false,
  "sort": [
    {
      "@timestamp": "desc"
    }
  ]
}
```
您可以使用日期数学来定义相对时间范围。以下查询搜索过去一天的数据
```
GET logs-my_app-default/_search
{
  "query": {
    "range": {
      "@timestamp": {
        "gte": "now-1d/d",
        "lt": "now/d"
      }
    }
  },
  "fields": [
    "@timestamp"
  ],
  "_source": false,
  "sort": [
    {
      "@timestamp": "desc"
    }
  ]
}
```

### 从非结构化内容中提取字段
您可以在搜索期间从非结构化内容（例如日志消息）中提取运行时字段。

使用以下搜索从 中提取source.ip运行时字段 event.original。要将其包含在响应中，请添加source.ip到fields 参数中。
```
GET logs-my_app-default/_search
{
  "runtime_mappings": {
    "source.ip": {
      "type": "ip",
      "script": """
        String sourceip=grok('%{IPORHOST:sourceip} .*').extract(doc[ "event.original" ].value)?.sourceip;
        if (sourceip != null) emit(sourceip);
      """
    }
  },
  "query": {
    "range": {
      "@timestamp": {
        "gte": "2099-05-05",
        "lt": "2099-05-08"
      }
    }
  },
  "fields": [
    "@timestamp",
    "source.ip"
  ],
  "_source": false,
  "sort": [
    {
      "@timestamp": "desc"
    }
  ]
}
```

### 组合查询
您可以使用bool查询来组合多个查询。以下搜索结合了两个range查询：一个在运行时字段上@timestamp，一个在source.ip 运行时字段上。
```
GET logs-my_app-default/_search
{
  "runtime_mappings": {
    "source.ip": {
      "type": "ip",
      "script": """
        String sourceip=grok('%{IPORHOST:sourceip} .*').extract(doc[ "event.original" ].value)?.sourceip;
        if (sourceip != null) emit(sourceip);
      """
    }
  },
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "@timestamp": {
              "gte": "2099-05-05",
              "lt": "2099-05-08"
            }
          }
        },
        {
          "range": {
            "source.ip": {
              "gte": "192.0.2.0",
              "lte": "192.0.2.240"
            }
          }
        }
      ]
    }
  },
  "fields": [
    "@timestamp",
    "source.ip"
  ],
  "_source": false,
  "sort": [
    {
      "@timestamp": "desc"
    }
  ]
}
```

### 聚合数据
使用聚合将数据汇总为指标、统计数据或其他分析。

以下搜索使用聚合来计算 average_response_size使用http.response.body.bytes运行时字段。聚合仅在与query.
```
GET logs-my_app-default/_search
{
  "runtime_mappings": {
    "http.response.body.bytes": {
      "type": "long",
      "script": """
        String bytes=grok('%{COMMONAPACHELOG}').extract(doc[ "event.original" ].value)?.bytes;
        if (bytes != null) emit(Integer.parseInt(bytes));
      """
    }
  },
  "aggs": {
    "average_response_size":{
      "avg": {
        "field": "http.response.body.bytes"
      }
    }
  },
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "@timestamp": {
              "gte": "2099-05-05",
              "lt": "2099-05-08"
            }
          }
        }
      ]
    }
  },
  "fields": [
    "@timestamp",
    "http.response.body.bytes"
  ],
  "_source": false,
  "sort": [
    {
      "@timestamp": "desc"
    }
  ]
}
```
响应的aggregations对象包含聚合结果。
```
{
  ...
  "aggregations" : {
    "average_response_size" : {
      "value" : 12368.0
    }
  }
}
```


## 清理
完成后，删除您的测试数据流及其支持索引。
```
DELETE _data_stream/logs-my_app-default
```