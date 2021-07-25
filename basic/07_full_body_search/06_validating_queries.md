# [验证查询](06_validating_queries.md)   
&emsp;&emsp;查询可以变得非常的复杂，尤其和不同的分析器与不同的字段映射结合时，
理解起来就有点困难了。不过 `validate-query API` 可以用来验证查询是否合法。  
```
GET /gb/tweet/_validate/query
{
   "query": {
      "tweet" : {
         "match" : "really powerful"
      }
   }
}
```
&emsp;&emsp;以上 `validate` 请求的应答告诉我们这个查询是不合法的：
```
{
  "valid" :         false,
  "_shards" : {
    "total" :       1,
    "successful" :  1,
    "failed" :      0
  }
}
```
## 理解错误信息
&emsp;&emsp;为了找出 查询不合法的原因，可以将 `explain `参数 加到查询字符串中：
```
GET /gb/tweet/_validate/query?explain    (a)
{
   "query": {
      "tweet" : {
         "match" : "really powerful"
      }
   }
}
```
`(a)explain 参数可以提供更多关于查询不合法的信息。`    

&emsp;&emsp;很明显，我们将查询类型(`match`)与字段名称 (`tweet`)搞混了：
```
{
  "valid" :     false,
  "_shards" :   { ... },
  "explanations" : [ {
    "index" :   "gb",
    "valid" :   false,
    "error" :   "org.elasticsearch.index.query.QueryParsingException:
                 [gb] No query registered for [tweet]"
  } ]
}
```

## 理解查询语句  
&emsp;&emsp;对于合法查询，使用 `explain` 参数将返回可读的描述，
这对准确理解 `Elasticsearch` 是如何解析你的 `query` 是非常有用的：  
```
GET /_validate/query?explain
{
   "query": {
      "match" : {
         "tweet" : "really powerful"
      }
   }
}
```
&emsp;&emsp;我们查询的每一个 `index` 都会返回对应的 `explanation` ，因为每一个 `index` 都有自己的映射和分析器：
```
{
  "valid" :         true,
  "_shards" :       { ... },
  "explanations" : [ {
    "index" :       "us",
    "valid" :       true,
    "explanation" : "tweet:really tweet:powerful"
  }, {
    "index" :       "gb",
    "valid" :       true,
    "explanation" : "tweet:realli tweet:power"
  } ]
}
```
&emsp;&emsp;从 explanation 中可以看出，
匹配 `really powerful` 的 `match` 查询被重写为两个针对 `tweet` 字段的 `single-term` 查询，
一个`single-term`查询对应查询字符串分出来的一个`term`。

&emsp;&emsp;当然，对于索引 `us` ，这两个 `term` 分别是 `really` 和 `powerful` ，
而对于索引 `gb` ，`term` 则分别是 `realli` 和 `power` 。
之所以出现这个情况，是由于我们将索引 `gb` 中 `tweet` 字段的分析器修改为 `english` 分析器。



