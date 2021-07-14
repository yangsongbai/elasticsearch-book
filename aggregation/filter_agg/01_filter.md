# 过滤   
如果我们想找到售价在 $10,000 美元之上的所有汽车同时也为这些车计算平均售价， 
可以简单地使用一个 constant_score 查询和 filter 约束：    
```
GET /cars/transactions/_search
{
    "size" : 0,
    "query" : {
        "constant_score": {
            "filter": {
                "range": {
                    "price": {
                        "gte": 10000
                    }
                }
            }
        }
    },
    "aggs" : {
        "single_avg_price": {
            "avg" : { "field" : "price" }
        }
    }
}
```    
这正如我们在前面章节中讨论过那样，从根本上讲，使用 non-scoring 查询和使用 match 查询没有任何区别。
查询（包括了一个过滤器）返回一组文档的子集，聚合正是操作这些文档。
使用 filtering query 会忽略评分，并有可能会缓存结果数据等等。     