# [轻量检索](08_search_lite.md)
&emsp;&emsp;一个 GET 是相当简单的，可以直接得到指定的文档。 现在尝试点儿稍微高级的功能，比如一个简单的搜索！

&emsp;&emsp;第一个尝试的几乎是最简单的搜索了。我们使用下列请求来搜索所有雇员：  
```$xslt
GET /megacorp/employee/_search
```
&emsp;&emsp;可以看到，我们仍然使用索引库 megacorp 以及类型 employee，但与指定一个文档 ID 不同，这次使用 _search 。
返回结果包括了所有三个文档，放在数组 hits 中。一个搜索默认返回十条结果。   
```$xslt
{
   "took":      6,
   "timed_out": false,
   "_shards": { ... },
   "hits": {
      "total":      3,
      "max_score":  1,
      "hits": [
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "3",
            "_score":         1,
            "_source": {
               "first_name":  "Douglas",
               "last_name":   "Fir",
               "age":         35,
               "about":       "I like to build cabinets",
               "interests": [ "forestry" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "1",
            "_score":         1,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "2",
            "_score":         1,
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
&emsp;&emsp;注意：返回结果不仅告知匹配了哪些文档，还包含了整个文档本身：显示搜索结果给最终用户所需的全部信息。

&emsp;&emsp;接下来，尝试下搜索姓氏为 ``Smith`` 的雇员。为此，我们将使用一个 高亮 搜索，很容易通过命令行完成。
这个方法一般涉及到一个 查询字符串 （query-string） 搜索，因为我们通过一个URL参数来传递查询信息给搜索接口：
```$xslt
GET /megacorp/employee/_search?q=last_name:Smith
```
&emsp;&emsp;我们仍然在请求路径中使用 _search 端点，并将查询本身赋值给参数 q= 。返回结果给出了所有的 Smith：   
```$xslt
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.30685282,
      "hits": [
         {
            ...
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
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

