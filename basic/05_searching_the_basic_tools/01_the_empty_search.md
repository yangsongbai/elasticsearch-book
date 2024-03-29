# [空搜索](01_the_empty_search.md) 
&emsp;&emsp;搜索API的最基础的形式是没有指定任何查询的空搜索，它简单地返回集群中所有索引下的所有文档：
```$xslt
GET /_search
```
返回的结果（为了界面简洁编辑过的）像这样：
```$xslt
{
   "hits" : {
      "total" :       14,
      "hits" : [
        {
          "_index":   "us",
          "_type":    "tweet",
          "_id":      "7",
          "_score":   1,
          "_source": {
             "date":    "2014-09-17",
             "name":    "John Smith",
             "tweet":   "The Query DSL is really powerful and flexible",
             "user_id": 2
          }
       },
        ... 9 RESULTS REMOVED ...
      ],
      "max_score" :   1
   },
   "took" :           4,
   "_shards" : {
      "failed" :      0,
      "successful" :  10,
      "total" :       10
   },
   "timed_out" :      false
}
```
## hits
&emsp;&emsp;返回结果中最重要的部分是 `hits` ，它包含 `total` 字段来表示匹配到的文档总数，
并且一个 `hits` 数组包含所查询结果的前十个文档。

&emsp;&emsp;在 `hits` 数组中每个结果包含文档的 `_index` 、 `_type` 、 `_id` ，加上 `_source` 字段。
这意味着我们可以直接从返回的搜索结果中使用整个文档。这不像其他的搜索引擎，仅仅返回文档的`ID`，需要你单独去获取文档。

&emsp;&emsp;每个结果还有一个 `_score` ，它衡量了文档与查询的匹配程度。
默认情况下，首先返回最相关的文档结果，就是说，返回的文档是按照 `_score` 降序排列的。
在这个例子中，我们没有指定任何查询，故所有的文档具有相同的相关性，因此对所有的结果而言 `1` 是中性的 `_score` 。

&emsp;&emsp;`max_score` 值是与查询所匹配文档的 `_score` 的最大值。
## took
&emsp;&emsp;`took` 值告诉我们执行整个搜索请求耗费了多少毫秒。
## shards
&emsp;&emsp;`_shards` 部分告诉我们在查询中参与分片的总数，以及这些分片成功了多少个失败了多少个。
正常情况下我们不希望分片失败，但是分片失败是可能发生的。
如果我们遭遇到一种灾难级别的故障，在这个故障中丢失了相同分片的原始数据和副本，
那么对这个分片将没有可用副本来对搜索请求作出响应。假若这样，Elasticsearch 将报告这个分片是失败的，
但是会继续返回剩余分片的结果。
## timeout
&emsp;&emsp;`timed_out` 值告诉我们查询是否超时。默认情况下，搜索请求不会超时。
如果低响应时间比完成结果更重要，你可以指定 `timeout` 为 `10` 或者 `10ms`（10毫秒），或者 `1s`（1秒）：
```$xslt
GET /_search?timeout=10ms
```
&emsp;&emsp;在请求超时之前，Elasticsearch 将会返回已经成功从每个分片获取的结果。
>警示⚠️
>>&emsp;&emsp;应当注意的是 timeout 不是停止执行查询，它仅仅是告知正在协调的节点返回到目前为止收集的结果并且关闭连接。
在后台，其他的分片可能仍在执行查询即使是结果已经被发送了。

>>&emsp;&emsp;使用超时是因为 SLA(服务等级协议)对你是很重要的，而不是因为想去中止长时间运行的查询。

