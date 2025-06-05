# ElasticSearch 入门

> [ElasticSearch (ES从入门到精通一篇就够了)](https://www.cnblogs.com/buchizicai/p/17093719.html)
>
> https://github.com/elastic/elasticsearch


## mac homebrew 安装ElasticSearch

step 1
```/bin/bash
brew tap elastic/tap

brew install elastic/tap/elasticsearch-full
```
---
step 2

To start elastic/tap/elasticsearch-full now and restart at login:
  `brew services start elastic/tap/elasticsearch-full`
Or, if you don't want/need a background service you can just run:
  `/opt/homebrew/opt/elasticsearch-full/bin/elasticsearch`

---
step3
验证： curl -X GET "http://127.0.0.1:9200/"

## mac homebrew 安装kibana

step 1
```/bin/bash
brew tap elastic/tap

brew install elastic/tap/elasticsearch-full
```
---
step 2

==> Caveats
Config: /opt/homebrew/etc/kibana/
If you wish to preserve your plugins upon upgrade, make a copy of
`/opt/homebrew/opt/kibana-full/plugins` before upgrading, and copy it into the
new keg location after upgrading.

To start elastic/tap/kibana-full now and restart at login:
  `brew services start elastic/tap/kibana-full`
Or, if you don't want/need a background service you can just run:
  `/opt/homebrew/opt/kibana-full/bin/kibana`
  
---
Step3
访问： http://127.0.0.1:5601/