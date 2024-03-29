# 脚本评分   
最后，如果所有 function_score 内置的函数都无法满足应用场景，可以使用 script_score 函数自行实现逻辑。
举个例子，想将利润空间作为因子加入到相关度评分计算，在业务中，利润空间和以下三点相关：     
 - price 度假屋每晚的价格。   
 - 会员用户的级别——某些等级的用户可以在每晚房价高于某个 threshold 阀值价格的时候享受折扣 discount 。   
 - 用户享受折扣后，经过议价的每晚房价的利润 margin 。      
计算每个度假屋利润的算法如下：    
```
if (price < threshold) {
  profit = price * margin
} else {
  profit = price * (1 - discount) * margin;
}
```    
我们很可能不想用绝对利润作为评分，这会弱化其他如地点、受欢迎度和特性等因子的作用，
而是将利润用目标利润 target 的百分比来表示，
高于 目标的利润空间会有一个正向评分（大于 1.0 ），
低于目标的利润空间会有一个负向分数（小于 1.0 ）：     
```
if (price < threshold) {
  profit = price * margin
} else {
  profit = price * (1 - discount) * margin
}
return profit / target
```    
Elasticsearch 里使用 [Groovy](http://groovy.codehaus.org/) 
作为默认的脚本语言，它与JavaScript很像，上面这个算法用 Groovy 脚本表示如下：      
```
price  = doc['price'].value    (1)
margin = doc['margin'].value   (1) 

if (price < threshold) {       (2) 
  return price * margin / target
}
return price * (1 - discount) * margin / target    (2)   
```
(1) price 和 margin 变量可以分别从文档的 price 和 margin 字段提取。     
(2) threshold 、 discount 和 target 是作为参数 params 传入的。      
最终我们将 script_score 函数与其他函数一起使用：     
```
GET /_search
{
  "function_score": {
    "functions": [
      { ...location clause... },     (1)
      { ...price clause... },        (1)    
      {
        "script_score": {
          "params": {                (2)
            "threshold": 80,
            "discount": 0.1,
            "target": 10
          },
          "script": "price  = doc['price'].value; margin = doc['margin'].value;
          if (price < threshold) { return price * margin / target };
          return price * (1 - discount) * margin / target;"      (3)
        }
      }
    ]
  }
}
```     
(1) location 和 price 语句在 [衰减函数](https://www.elastic.co/guide/cn/elasticsearch/guide/current/decay-functions.html) 中解释过。      
(2) 将这些变量作为参数 params 传递，我们可以查询时动态改变脚本无须重新编译。   
(3) JSON 不能接受内嵌的换行符，脚本中的换行符可以用 \n 或 ; 符号替代。     

 这个查询根据用户对地点和价格的需求，返回用户最满意的文档，同时也考虑到我们对于盈利的要求。     
 
> TIP  
>>  script_score 函数提供了巨大的灵活性，可以通过脚本访问文档里的所有字段、
>> 当前评分 _score 甚至词频、逆向文档频率和字段长度规范值这样的信息（参见 see [脚本对文本评分](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/modules-advanced-scripting.html)）。

>> 有人说使用脚本对性能会有影响，如果确实发现脚本执行较慢，可以有以下三种选择：
     (1) 尽可能多的提前计算各种信息并将结果存入每个文档中。       
     (2) Groovy 很快，但没 Java 快。可以将脚本用原生的 Java 脚本重新实现。（参见 [原生 Java 脚本](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/modules-scripting-native.html)）。      
     (3)仅对那些最佳评分的文档应用脚本，使用 重新评分 中提到的 [rescore](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_Improving_Performance.html#rescore-api) 功能。          
