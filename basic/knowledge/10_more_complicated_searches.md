# [更复杂的搜索](10_more_complicated_searches.md)
&emsp;&emsp;现在尝试下更复杂的搜索。 同样搜索姓氏为 Smith 的员工，但这次我们只需要年龄大于 30 的。
查询需要稍作调整，使用过滤器 filter ，它支持高效地执行一个结构化查询。

```$xslt
GET /megacorp/employee/_search
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith"  (a)
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 }  (b)
                }
            }
        }
    }
}
```
(a) 这部分与我们之前使用的 match 查询 一样。

(b) 这部分是一个 range 过滤器 ， 它能找到年龄大于 30 的文档，其中 gt 表示_大于_(great than)。

&emsp;&emsp;目前无需太多担心语法问题，后续会更详细地介绍。只需明确我们添加了一个 过滤器 用于执行一个范围查询，
并复用之前的 match 查询。现在结果只返回了一名员工，叫 Jane Smith，32 岁。
```$xslt
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.30685282,
      "hits": [
         {
            ...
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```