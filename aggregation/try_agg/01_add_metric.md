# 添加度量指标   
前面的例子告诉我们每个桶里面的文档数量，这很有用。但通常，我们的应用需要提供更复杂的文档度量。 例如，每种颜色汽车的平均价格是多少？   

为了获取更多信息，我们需要告诉 Elasticsearch 使用哪个字段，计算何种度量。 这需要将度量 嵌套 在桶内， 度量会基于桶内的文档计算统计结果。   

让我们继续为汽车的例子加入 average 平均度量：    

```
GET /cars/transactions/_search
{
   "size" : 0,
   "aggs": {
      "colors": {
         "terms": {
            "field": "color"
         },
         "aggs": { 
            "avg_price": { 
               "avg": {
                  "field": "price" 
               }
            }
         }
      }
   }
}
```  
为度量新增 aggs 层。
为度量指定名字： avg_price 。
最后，为 price 字段定义 avg 度量。

正如所见，我们用前面的例子加入了新的 aggs 层。这个新的聚合层让我们可以将 avg 度量嵌套置于 terms 桶内。实际上，这就为每个颜色生成了平均价格。    

正如 颜色 的例子，我们需要给度量起一个名字（ avg_price ）这样可以稍后根据名字获取它的值。
最后，我们指定度量本身（ avg ）以及我们想要计算平均值的字段（ price ）：     
```
{
...
   "aggregations": {
      "colors": {
         "buckets": [
            {
               "key": "red",
               "doc_count": 4,
               "avg_price": { 
                  "value": 32500
               }
            },
            {
               "key": "blue",
               "doc_count": 2,
               "avg_price": {
                  "value": 20000
               }
            },
            {
               "key": "green",
               "doc_count": 2,
               "avg_price": {
                  "value": 21000
               }
            }
         ]
      }
   }
...
}
```    
响应中的新字段 avg_price   

尽管响应只发生很小改变，实际上我们获得的数据是增长了。
之前，我们知道有四辆红色的车，现在，红色车的平均价格是 $32，500 美元。这个信息可以直接显示在报表或者图形中。   

  
