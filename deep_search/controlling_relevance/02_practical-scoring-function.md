# Lucene 的实用评分函数    
对于多词查询， Lucene 使用 布尔模型（Boolean model） 、 TF/IDF 以及 向量空间模型（vector space model） ，
然后将它们组合到单个高效的包里以收集匹配文档并进行评分计算。   
一个多词查询   
```
GET /my_index/doc/_search
{
  "query": {
    "match": {
      "text": "quick fox"
    }
  }
}
```    
会在内部被重写为：   
```
GET /my_index/doc/_search
{
  "query": {
    "bool": {
      "should": [
        {"term": { "text": "quick" }},
        {"term": { "text": "fox"   }}
      ]
    }
  }
}
```      
bool 查询实现了布尔模型，在这个例子中，它会将包括词 quick 和 fox 或两者兼有的文档作为查询结果。

只要一个文档与查询匹配，Lucene 就会为查询计算评分，然后合并每个匹配词的评分结果。
这里使用的评分计算公式叫做 实用评分函数（practical scoring function） 。
看似很高大上，但是别被吓到——多数的组件都已经介绍过，下一步会讨论它引入的一些新元素。   
