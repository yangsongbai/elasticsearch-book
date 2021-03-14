# [排序](01_sorting.md) 
&emsp;&emsp;为了按照相关性来排序，需要将相关性表示为一个数值。在 Elasticsearch 中， 
相关性得分 由一个浮点数进行表示，并在搜索结果中通过 `_score` 参数返回， 
默认排序是 `_score` 降序。

&emsp;&emsp;有时，相关性评分对你来说并没有意义。例如，下面的查询返回所有 `user_id` 字段包含 `1` 的结果：   
```
GET /_search
{
    "query" : {
        "bool" : {
            "filter" : {
                "term" : {
                    "user_id" : 1
                }
            }
        }
    }
}
```
&emsp;&emsp;这里没有一个有意义的分数：因为我们使用的是 `filter` （过滤），
这表明我们只希望获取匹配 `user_id: 1` 的文档，并没有试图确定这些文档的相关性。 
实际上文档将按照随机顺序返回，并且每个文档都会评为零分。   
>Note
>>如果评分为零对你造成了困扰，你可以使用 constant_score 查询进行替代：
>>
```
GET /_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "term" : {
                    "user_id" : 1
                }
            }
        }
    }
}
```
>>&emsp;&emsp;这将让所有文档应用一个恒定分数（默认为 `1` ）。它将执行与前述查询相同的查询，
并且所有的文档将像之前一样随机返回，这些文档只是有了一个分数而不是零分。

## 按照字段的值排序  
&emsp;&emsp;在这个案例中，通过时间来对 `tweets` 进行排序是有意义的，最新的 `tweets` 排在最前。
我们可以使用 `sort `参数进行实现：  
```
GET /_search
{
    "query" : {
        "bool" : {
            "filter" : { "term" : { "user_id" : 1 }}
        }
    },
    "sort": { "date": { "order": "desc" }}
}
```
你会注意到结果中的两个不同点：
```
"hits" : {
    "total" :           6,
    "max_score" :       null,   (a)
    "hits" : [ {
        "_index" :      "us",
        "_type" :       "tweet",
        "_id" :         "14",
        "_score" :      null,   (a)
        "_source" :     {
             "date":    "2014-09-24",
             ...
        },
        "sort" :        [ 1411516800000 ]   (b)
    },
    ...
}
```
`(a)_score 不被计算, 因为它并没有用于排序。`    

`(b)date 字段的值表示为自 epoch (January 1, 1970 00:00:00 UTC)以来的毫秒数，通过 sort 字段的值进行返回。`     

&emsp;&emsp;首先我们在每个结果中有一个新的名为 `sort` 的元素，它包含了我们用于排序的值。 
在这个案例中，我们按照 `date` 进行排序，在内部被索引为 自 `epoch` 以来的毫秒数 。
 `long` 类型数 `1411516800000` 等价于日期字符串 `2014-09-24 00:00:00 UTC` 。

&emsp;&emsp;其次 `_score` 和 `max_score` 字段都是 `null` 。计算 `_score` 的花销巨大，
通常仅用于排序； 我们并不根据相关性排序，所以记录` _score` 是没有意义的。
如果无论如何你都要计算` _score` ， 你可以将 `track_scores` 参数设置为 `true` 。  
>Tip
>>一个简便方法是, 你可以指定一个字段用来排序：  
>>`    "sort": "number_of_children"`
>>&emsp;&emsp;字段将会默认升序排序，而按照 _score 的值进行降序排序。  

## 多级排序  
&emsp;&emsp;假定我们想要结合使用 `date` 和 `_score` 进行查询，
并且匹配的结果首先按照日期排序，然后按照相关性排序：
```
GET /_search
{
    "query" : {
        "bool" : {
            "must":   { "match": { "tweet": "manage text search" }},
            "filter" : { "term" : { "user_id" : 2 }}
        }
    },
    "sort": [
        { "date":   { "order": "desc" }},
        { "_score": { "order": "desc" }}
    ]
}
```
&emsp;&emsp;排序条件的顺序是很重要的。结果首先按第一个条件排序，仅当结果集的第一个 `sort` 
值完全相同时才会按照第二个条件进行排序，以此类推。

&emsp;&emsp;多级排序并不一定包含 `_score` 。
你可以根据一些不同的字段进行排序，如地理距离或是脚本计算的特定值。
>Note
>>`Query-string` 搜索 也支持自定义排序，可以在查询字符串中使用 `sort` 参数：   
>>
```
GET /_search?sort=date:desc&sort=_score&q=search
```

## 多值字段的排序
&emsp;&emsp;一种情形是字段有多个值的排序， 需要记住这些值并没有固有的顺序；
一个多值的字段仅仅是多个值的包装，这时应该选择哪个进行排序呢？

&emsp;&emsp;对于数字或日期，你可以将多值字段减为单值，
这可以通过使用 `min` 、` max `、 `avg` 或是 `sum` 排序模式 。 
例如你可以按照每个 `date` 字段中的最早日期进行排序，通过以下方法：
```
"sort": {
    "dates": {
        "order": "asc",
        "mode":  "min"
    }
}
```


