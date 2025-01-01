# ES

ES安装路径： https://www.elastic.co/cn/downloads/elasticsearch

kibana安装路径：https://www.elastic.co/guide/en/kibana/7.17/deb.html#deb-repo

## 聚合

查询每个州的平均薪水

```json
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

