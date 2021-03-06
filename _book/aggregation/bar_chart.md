# 条形图  
聚合还有一个令人激动的特性就是能够十分容易地将它们转换成图表和图形。
本章中， 我们正在通过示例数据来完成各种各样的聚合分析，最终，我们将会发现聚合功能是非常强大的。

直方图 histogram 特别有用。 它本质上是一个条形图，如果有创建报表或分析仪表盘的经验，那么我们会毫无疑问的发现里面有一些图表是条形图。 
创建直方图需要指定一个区间，如果我们要为售价创建一个直方图，可以将间隔设为 20,000。这样做将会在每个 $20,000 档创建一个新桶，然后文档会被分到对应的桶中。

对于仪表盘来说，我们希望知道每个售价区间内汽车的销量。我们还会想知道每个售价区间内汽车所带来的收入，可以通过对每个区间内已售汽车的售价求和得到。

可以用 histogram 和一个嵌套的 sum 度量得到我们想要的答案：  

```
GET /cars/transactions/_search
{
   "size" : 0,
   "aggs":{
      "price":{
         "histogram":{ 
            "field": "price",
            "interval": 20000
         },
         "aggs":{
            "revenue": {
               "sum": { 
                 "field" : "price"
               }
             }
         }
      }
   }
}
```   
histogram 桶要求两个参数：一个数值字段以及一个定义桶大小间隔。   
sum 度量嵌套在每个售价区间内，用来显示每个区间内的总收入。  
 
如我们所见，查询是围绕 price 聚合构建的，它包含一个 histogram 桶。
它要求字段的类型必须是数值型的同时需要设定分组的间隔范围。 
间隔设置为 20,000 意味着我们将会得到如 [0-19999, 20000-39999, ...] 这样的区间。

接着，我们在直方图内定义嵌套的度量，这个 sum 度量，它会对落入某一具体售价区间的文档中 price 字段的值进行求和。 
这可以为我们提供每个售价区间的收入，从而可以发现到底是普通家用车赚钱还是奢侈车赚钱。

响应结果如下：  
```
{
...
   "aggregations": {
      "price": {
         "buckets": [
            {
               "key": 0,
               "doc_count": 3,
               "revenue": {
                  "value": 37000
               }
            },
            {
               "key": 20000,
               "doc_count": 4,
               "revenue": {
                  "value": 95000
               }
            },
            {
               "key": 80000,
               "doc_count": 1,
               "revenue": {
                  "value": 80000
               }
            }
         ]
      }
   }
}
```   
结果很容易理解，不过应该注意到直方图的键值是区间的下限。键 0 代表区间 0-19，999 ，键 20000 代表区间 20，000-39，999 ，等等。    
> 
>> 我们可能会注意到空的区间，比如：$40，000-60，000，没有出现在响应中。 histogram 桶默认会忽略它，因为它有可能会导致不希望的潜在错误输出。   
我们会在下一小节中讨论如何包括空桶。返回空桶 [返回空 Buckets](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_returning_empty_buckets.html) 。

可以在图 Figure 35, “Sales and Revenue per price bracket” 中看到以上数据直方图的图形化表示。

<img src="./images/figure_35.png" width = "800" height = "200" 
alt="Sales and Revenue per price bracket" align=center />

当然，我们可以为任何聚合输出的分类和统计结果创建条形图，而不只是 直方图 桶。
让我们以最受欢迎 10 种汽车以及它们的平均售价、标准差这些信息创建一个条形图。 我们会用到 terms 桶和 extended_stats 度量：   

```
GET /cars/transactions/_search
{
  "size" : 0,
  "aggs": {
    "makes": {
      "terms": {
        "field": "make",
        "size": 10
      },
      "aggs": {
        "stats": {
          "extended_stats": {
            "field": "price"
          }
        }
      }
    }
  }
}
```    
上述代码会按受欢迎度返回制造商列表以及它们各自的统计信息。我们对其中的 stats.avg 、 stats.count 和 stats.std_deviation 信息特别感兴趣，并用 它们计算出标准差：

std_err = std_deviation / count

创建图表如图 Figure 36, “Average price of all makes, with error bars” 。   
<img src="./images/figure_36.png" width = "800" height = "200" 
alt="Average price of all makes, with error bars" align=center />
