# [全文搜索](11_full_text_search.md)

&emsp;&emsp;截止目前的搜索相对都很简单：单个姓名，通过年龄过滤。现在尝试下稍微高级点儿的全文搜索——一项 传统数据库确实很难搞定的任务。

&emsp;&emsp;搜索下所有喜欢攀岩（rock climbing）的员工：
```$xslt
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```
&emsp;&emsp;显然我们依旧使用之前的 match 查询在`about` 属性上搜索 “rock climbing” 。得到两个匹配的文档：   
```$xslt
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.16273327,
      "hits": [
         {
            ...
            "_score":         0.16273327, (a)
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
            "_score":         0.016878016, (a)
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
(a) 相关性得分

&emsp;&emsp;Elasticsearch 默认按照相关性得分排序，即每个文档跟查询的匹配程度。
第一个最高得分的结果很明显：John Smith 的 about 属性清楚地写着 “rock climbing” 。

&emsp;&emsp;但为什么 Jane Smith 也作为结果返回了呢？原因是她的 about 属性里提到了 “rock” 。
因为只有 “rock” 而没有 “climbing” ，所以她的相关性得分低于 John 的。

&emsp;&emsp;这是一个很好的案例，阐明了 Elasticsearch 如何 在 全文属性上搜索并返回相关性最强的结果。
Elasticsearch中的 相关性 概念非常重要，也是完全区别于传统关系型数据库的一个概念，数据库中的一条记录要么匹配要么不匹配。

