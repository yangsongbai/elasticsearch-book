# 忽略 TF/IDF  
有时候我们根本不关心 TF/IDF ，只想知道一个词是否在某个字段中出现过。可能搜索一个度假屋并希望它能尽可能有以下设施：
 - WiFi   
 - Garden（花园）   
 - Pool（游泳池）   
 
 这个度假屋的文档如下：     
 ```
{ "description": "A delightful four-bedroomed house with ... " }
```     
可以用简单的 match 查询进行匹配：    
```
GET /_search
{
  "query": {
    "match": {
      "description": "wifi garden pool"
    }
  }
}
```    
但这并不是真正的 全文搜索 ，此种情况下，TF/IDF 并无用处。
我们既不关心 wifi 是否为一个普通词，也不关心它在文档中出现是否频繁，关心的只是它是否曾出现过。
实际上，我们希望根据房屋不同设施的数量对其排名——设施越多越好。如果设施出现，则记 1 分，不出现记 0 分。    

## constant_score    
在 [constant_score](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/query-dsl-constant-score-query.html) 
查询中，它可以包含查询或过滤，
为任意一个匹配的文档指定评分 1 ，忽略 TF/IDF 信息：   
```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "constant_score": {
          "query": { "match": { "description": "wifi" }}
        }},
        { "constant_score": {
          "query": { "match": { "description": "garden" }}
        }},
        { "constant_score": {
          "query": { "match": { "description": "pool" }}
        }}
      ]
    }
  }
}
```    
或许不是所有的设施都同等重要——对某些用户来说有些设施更有价值。
如果最重要的设施是游泳池，那我们可以为更重要的设施增加权重：    
```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "constant_score": {
          "query": { "match": { "description": "wifi" }}
        }},
        { "constant_score": {
          "query": { "match": { "description": "garden" }}
        }},
        { "constant_score": {
          "boost":   2 
          "query": { "match": { "description": "pool" }}
        }}
      ]
    }
  }
} 
```    
pool 语句的权重提升值为 2 ，而其他的语句为 1 。   

>  注意  
>> 最终的评分并不是所有匹配语句的简单求和， 协调因子（coordination factor） 和 查询归一化因子（query normalization factor） 仍然会被考虑在内。   

我们可以给 features 字段加上 not_analyzed 类型来提升度假屋文档的匹配能力：
```
{ "features": [ "wifi", "pool", "garden" ] }
```
默认情况下，一个 not_analyzed 字段会禁用 字段长度归一值（field-length norms） 的功能，
并将 index_options 设为 docs 选项，禁用 词频 ，但还是存在问题：每个词的 [倒排文档频率](https://www.elastic.co/guide/cn/elasticsearch/guide/current/scoring-theory.html#idf) 
仍然会被考虑。    

可以采用与之前相同的方法 constant_score 查询来解决这个问题：
````
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "constant_score": {
          "query": { "match": { "features": "wifi" }}
        }},
        { "constant_score": {
          "query": { "match": { "features": "garden" }}
        }},
        { "constant_score": {
          "boost":   2
          "query": { "match": { "features": "pool" }}
        }}
      ]
    }
  }
}
````    
实际上，每个设施都应该看成一个过滤器，对于度假屋来说要么具有某个设施要么没有——过滤器因为其性质天然合适。
而且，如果使用过滤器，我们还可以利用缓存。     

这里的问题是：过滤器无法计算评分。这样就需要寻求一种方式将过滤器和查询间的差异抹平。 
function_score 查询不仅正好可以扮演这个角色，而且有更强大的功能。    
  