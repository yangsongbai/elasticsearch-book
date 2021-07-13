# 最后的修改   
让我们回到话题的原点，在进入新话题之前，对我们的示例做最后一个修改， 为每个汽车生成商计算最低和最高的价格：   
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
            "avg_price": { "avg": { "field": "price" }
            },
            "make" : {
                "terms" : {
                    "field" : "make"
                },
                "aggs" : { 
                    "min_price" : { "min": { "field": "price"} }, 
                    "max_price" : { "max": { "field": "price"} } 
                }
            }
         }
      }
   }
}
```
我们需要增加另外一个嵌套的 aggs 层级。   
然后包括 min 最小度量。   
以及 max 最大度量。     

得到以下输出（只显示部分结果）：   
```
{
...
   "aggregations": {
      "colors": {
         "buckets": [
            {
               "key": "red",
               "doc_count": 4,
               "make": {
                  "buckets": [
                     {
                        "key": "honda",
                        "doc_count": 3,
                        "min_price": {
                           "value": 10000 
                        },
                        "max_price": {
                           "value": 20000 
                        }
                     },
                     {
                        "key": "bmw",
                        "doc_count": 1,
                        "min_price": {
                           "value": 80000
                        },
                        "max_price": {
                           "value": 80000
                        }
                     }
                  ]
               },
               "avg_price": {
                  "value": 32500
               }
            },
...
```
min 和 max 度量现在出现在每个汽车制造商（ make ）下面。   
 
有了这两个桶，我们可以对查询的结果进行扩展并得到以下信息：    
 - 有四辆红色车。    
 - 红色车的平均售价是 $32，500 美元。   
 - 其中三辆红色车是 Honda 本田制造，一辆是 BMW 宝马制造。   
 - 最便宜的红色本田售价为 $10，000 美元。   
 - 最贵的红色本田售价为 $20，000 美元。   