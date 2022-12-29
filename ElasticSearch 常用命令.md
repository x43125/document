# ElasticSearch 常用命令

```shell
curl http://localhost:9200/_cat/health?v

curl -H "Content-Type: application/json" -XPUT 'http://localhost:9200/_xpack/security/user/elastic/_password?pretty' -d '{"password":"new_password"}'

curl -H "Content-Type: application/json" -XDELETE http://localhost:9200/testindex

# index名必须为全小写！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！
curl -H "Content-Type: application/json" -XPUT http://localhost:9200/testindex


curl -XGET http://localhost:9200/testindex/_mapping?pretty

curl -H "Content-Type: application/json" -XPUT 'http://localhost:9200/testindex/_mapping?pretty' -d '{"properties":{"mchnt_id" : {"type" : "keyword","index" : false},"mchnt_name" : {"type" : "text"}}}'

curl -H "Content-Type: application/json" -XPOST 'http://localhost:9200/testindex/_mapping?pretty' -d '{"properties": {"amount":{"type":"integer"}}}'

```



