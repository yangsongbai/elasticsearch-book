# [映射和分析](README.md)  
&emsp;&emsp;当摆弄索引里面的数据时，我们发现一些奇怪的事情。一些事情看起来被打乱了：
在我们的索引中有12条推文，其中只有一条包含日期 `2014-09-15` ，但是看一看下面查询命中的 总数 （`total`）：
```$xslt
GET /_search?q=2014              # 12 results
GET /_search?q=2014-09-15        # 12 results !
GET /_search?q=date:2014-09-15   # 1  result
GET /_search?q=date:2014         # 0  results !
```
&emsp;&emsp;为什么在 _all 字段查询日期返回所有推文，而在 date 字段只查询年份却没有返回结果？
为什么我们在 _all 字段和 date 字段的查询结果有差别？

&emsp;&emsp;推测起来，这是因为数据在 _all 字段与 date 字段的索引方式不同。
所以，通过请求 gb 索引中 tweet 类型的映射（或模式定义），让我们看一看 Elasticsearch 是如何解释我们文档结构的：
```$xslt
GET /gb/_mapping/tweet
```
&emsp;&emsp;这将得到如下结果：
```$xslt
{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "strict_date_optional_time||epoch_millis"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}
```
&emsp;&emsp;基于对字段类型的猜测， `Elasticsearch` 动态为我们产生了一个映射。
这个响应告诉我们 `date` 字段被认为是 `date` 类型的。由于 `_all` 是默认字段，所以没有提及它。
但是我们知道 `_all` 字段是 `string` 类型的。

&emsp;&emsp;所以 `date` 字段和 `string` 字段索引方式不同，因此搜索结果也不一样。这完全不令人吃惊。
你可能会认为核心数据类型 `strings`、`numbers`、`Booleans` 和 `dates` 的索引方式有稍许不同。没错，他们确实稍有不同。

&emsp;&emsp;但是，到目前为止，最大的差异在于代表 精确值 （它包括 string 字段）的字段和代表 全文 的字段。
这个区别非常重要——它将搜索引擎和所有其他数据库区别开来。


