# Not Quite Not  
在互联网上搜索 “Apple”，返回的结果很可能是一个公司、水果和各种食谱。我们可以在 bool 
查询中用 must_not 语句来排除像 pie 、 tart 、 crumble 和 tree 这样的词，
从而将查询结果的范围缩小至只返回与 “Apple” （苹果）公司相关的结果：   
```
GET /_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "text": "apple"
        }
      },
      "must_not": {
        "match": {
          "text": "pie tart fruit crumble tree"
        }
      }
    }
  }
}
```    
但谁又敢保证在排除 tree 或 crumble 这种词后，不会错失一个与苹果公司特别相关的文档呢？有时， must_not 条件会过于严格。    
## 权重提升查询  
boosting 查询 恰恰能解决这个问题。它仍然允许我们将关于水果或甜点的结果包括到结果中，但是使它们降级——即降低它们原来可能应有的排名：   
```
GET /_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "text": "apple"
        }
      },
      "negative": {
        "match": {
          "text": "pie tart fruit crumble tree"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```    
它接受 positive 和 negative 查询。只有那些匹配 positive 查询的文档罗列出来，
对于那些同时还匹配 negative 查询的文档将通过文档的原始 _score 与 negative_boost 相乘的方式降级后的结果。

为了达到效果， negative_boost 的值必须小于 1.0 。在这个示例中，所有包含负向词的文档评分 _score 都会减半。

