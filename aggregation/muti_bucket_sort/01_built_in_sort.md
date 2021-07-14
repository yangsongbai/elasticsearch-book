# 内置排序    
这些排序模式是桶 固有的 能力：它们操作桶生成的数据 ，比如 doc_count 。 
它们共享相同的语法，但是根据使用桶的不同会有些细微差别。   

让我们做一个 terms 聚合但是按 doc_count 值的升序排序：   
```
GET /cars/transactions/_search
{
    "size" : 0,
    "aggs" : {
        "colors" : {
            "terms" : {
              "field" : "color",
              "order": {
                "_count" : "asc" 
              }
            }
        }
    }
}
```     
我们为聚合引入了一个 order 对象， 它允许我们可以根据以下几个值中的一个值进行排序：

_count   
按文档数排序。对 terms 、 histogram 、 date_histogram 有效。    
_term     
按词项的字符串值的字母顺序排序。只在 terms 内使用。    
_key     
按每个桶的键值数值排序（理论上与 _term 类似）。 只在 histogram 和 date_histogram 内使用。    
