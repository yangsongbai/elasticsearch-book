# 按度量排序  
有时，我们会想基于度量计算的结果值进行排序。 在我们的汽车销售分析仪表盘中，
我们可能想按照汽车颜色创建一个销售条状图表，但按照汽车平均售价的升序进行排序。   

我们可以增加一个度量，再指定 order 参数引用这个度量即可：    
```
GET /cars/transactions/_search
{
    "size" : 0,
    "aggs" : {
        "colors" : {
            "terms" : {
              "field" : "color",
              "order": {
                "avg_price" : "asc" 
              }
            },
            "aggs": {
                "avg_price": {
                    "avg": {"field": "price"} 
                }
            }
        }
    }
}
```    
计算每个桶的平均售价。     
桶按照计算平均值的升序排序。   

我们可以采用这种方式用任何度量排序，只需简单的引用度量的名字。不过有些度量会输出多个值。 
extended_stats 度量是一个很好的例子：它输出好几个度量值。   

如果我们想使用多值度量进行排序， 我们只需以关心的度量为关键词使用点式路径：    
```
GET /cars/transactions/_search
{
    "size" : 0,
    "aggs" : {
        "colors" : {
            "terms" : {
              "field" : "color",
              "order": {
                "stats.variance" : "asc" 
              }
            },
            "aggs": {
                "stats": {
                    "extended_stats": {"field": "price"}
                }
            }
        }
    }
}
```   
使用 . 符号，根据感兴趣的度量进行排序。     

在上面这个例子中，我们按每个桶的方差来排序，所以这种颜色售价方差最小的会排在结果集最前面。    