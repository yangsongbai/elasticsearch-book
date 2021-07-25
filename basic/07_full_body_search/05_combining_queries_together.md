# [组合多查询](05_combining_queries_together.md)   
&emsp;&emsp;现实的查询需求从来都没有那么简单；它们需要在多个字段上查询多种多样的文本，
并且根据一系列的标准来过滤。为了构建类似的高级查询，你需要一种能够将多查询组合成单一查询的查询方法。

&emsp;&emsp;你可以用 `bool` 查询来实现你的需求。这种查询将多查询组合在一起，成为用户自己想要的布尔查询。它接收以下参数：  

`must`   
&emsp;&emsp;文档 必须 匹配这些条件才能被包含进来。    
`must_not`    
&emsp;&emsp;文档 必须不 匹配这些条件才能被包含进来。     
`should`      
&emsp;&emsp;如果满足这些语句中的任意语句，将增加 `_score` ，否则，无任何影响。它们主要用于修正每个文档的相关性得分。    
`filter`     
&emsp;&emsp;必须 匹配，但它以不评分、过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。 

&emsp;&emsp;由于这是我们看到的第一个包含多个查询的查询，所以有必要讨论一下相关性得分是如何组合的。
每一个子查询都独自地计算文档的相关性得分。一旦他们的得分被计算出来， 
`bool` 查询就将这些得分进行合并并且返回一个代表整个布尔操作的得分。

&emsp;&emsp;下面的查询用于查找 `title` 字段匹配 `how to make millions` 并且不被标识为 `spam` 的文档。
那些被标识为 `starred` 或在`2014`之后的文档，将比另外那些文档拥有更高的排名。
如果 `两者` 都满足，那么它排名将更高：  

```
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }},
            { "range": { "date": { "gte": "2014-01-01" }}}
        ]
    }
}
```
>Tip
>>&emsp;&emsp;如果没有 must 语句，那么至少需要能够匹配其中的一条 should 语句。
但，如果存在至少一条 must 语句，则对 should 语句的匹配没有要求。   

## 增加带过滤器（filtering）的查询 
&emsp;&emsp;如果我们不想因为文档的时间而影响得分，可以用 filter 语句来重写前面的例子：   
```
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "range": { "date": { "gte": "2014-01-01" }}  (a)
        }
    }
}
```
`(a)range 查询已经从 should 语句中移到 filter 语句`  

&emsp;&emsp;通过将 `range` 查询移到 `filter` 语句中，我们将它转成不评分的查询，
将不再影响文档的相关性排名。由于它现在是一个不评分的查询，
可以使用各种对 `filter` 查询有效的优化手段来提升性能。

&emsp;&emsp;所有查询都可以借鉴这种方式。将查询移到 `bool` 查询的 `filter` 语句中，
这样它就自动的转成一个不评分的 `filter` 了。

&emsp;&emsp;如果你需要通过多个不同的标准来过滤你的文档，
`bool` 查询本身也可以被用做不评分的查询。
简单地将它放置到 `filter` 语句中并在内部构建布尔逻辑：  
```
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "bool": {   (a)
              "must": [
                  { "range": { "date": { "gte": "2014-01-01" }}},
                  { "range": { "price": { "lte": 29.99 }}}
              ],
              "must_not": [
                  { "term": { "category": "ebooks" }}
              ]
          }
        }
    }
}
```
`(a)将 bool 查询包裹在 filter 语句中，我们可以在过滤标准中增加布尔逻辑`      


&emsp;&emsp;通过混合布尔查询，我们可以在我们的查询请求中灵活地编写 `scoring` 和 `filtering` 查询逻辑。

## constant_score 查询
&emsp;&emsp;尽管没有 bool 查询使用这么频繁，constant_score 查询也是你工具箱里有用的查询工具。
它将一个不变的常量评分应用于所有匹配的文档。它被经常用于你只需要执行一个 filter 
而没有其它查询（例如，评分查询）的情况下。

&emsp;&emsp;可以使用它来取代只有 filter 语句的 bool 查询。在性能上是完全相同的，
但对于提高查询简洁性和清晰度有很大帮助。   
```
{
    "constant_score":   {
        "filter": {
            "term": { "category": "ebooks" }   (a)
        }
    }
}
```
``
(a) term 查询被放置在 constant_score 中，转成不评分的 filter。
这种方式可以用来取代只有 filter 语句的 bool 查询。
``

