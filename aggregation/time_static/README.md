# 按时间统计    
如果搜索是在 Elasticsearch 中使用频率最高的，那么构建按时间统计的 date_histogram 紧随其后。 为什么你会想用 date_histogram 呢？

假设你的数据带时间戳。 无论是什么数据（Apache 事件日志、股票买卖交易时间、棒球运动时间）只要带有时间戳都可以进行 date_histogram 分析。当你的数据有时间戳，你总是想在 时间 维度上构建指标分析：

 - 今年每月销售多少台汽车？     
 - 这只股票最近 12 小时的价格是多少？  
 - 我们网站上周每小时的平均响应延迟时间是多少？   
虽然通常的 histogram 都是条形图，但 date_histogram 倾向于转换成线状图以展示时间序列。 许多公司用 Elasticsearch 仅仅 只是为了分析时间序列数据。 
date_histogram 分析是它们最基本的需要。

date_histogram 与 通常的 histogram 类似。 但不是在代表数值范围的数值字段上构建 buckets，而是在时间范围上构建 buckets。
 因此每一个 bucket 都被定义成一个特定的日期大小 (比如， 1个月 或 2.5 天 )。
> 可以用通常的 histogram 进行时间分析吗？
>>从技术上来讲，是可以的。 通常的 histogram bucket（桶）是可以处理日期的。 但是它不能自动识别日期。 
>>而用 date_histogram ，你可以指定时间段如 1 个月 ，它能聪明地知道 2 月的天数比 12 月少。 
>>date_histogram 还具有另外一个优势，即能合理地处理时区，这可以使你用客户端的时区进行图标定制，而不是用服务器端时区。
通常的 histogram 会把日期看做是数字，这意味着你必须以微秒为单位指明时间间隔。另外聚合并不知道日历时间间隔，使得它对于日期而言几乎没什么用处。   

我们的第一个例子将构建一个简单的折线图来回答如下问题： 每月销售多少台汽车？   
```
GET /cars/transactions/_search
{
   "size" : 0,
   "aggs": {
      "sales": {
         "date_histogram": {
            "field": "sold",
            "interval": "month", 
            "format": "yyyy-MM-dd" 
         }
      }
   }
}
```    
时间间隔要求是日历术语 (如每个 bucket 1 个月)。   
我们提供日期格式以便 buckets 的键值便于阅读。   

我们的查询只有一个聚合，每月构建一个 bucket。这样我们可以得到每个月销售的汽车数量。 另外还提供了一个额外的 format 参数以便 buckets 有 "好看的" 键值。
 然而在内部，日期仍然是被简单表示成数值。这可能会使得 UI 设计者抱怨，因此可以提供常用的日期格式进行格式化以更方便阅读。   
结果既符合预期又有一点出人意料（看看你是否能找到意外之处）：   
```
{
   ...
   "aggregations": {
      "sales": {
         "buckets": [
            {
               "key_as_string": "2014-01-01",
               "key": 1388534400000,
               "doc_count": 1
            },
            {
               "key_as_string": "2014-02-01",
               "key": 1391212800000,
               "doc_count": 1
            },
            {
               "key_as_string": "2014-05-01",
               "key": 1398902400000,
               "doc_count": 1
            },
            {
               "key_as_string": "2014-07-01",
               "key": 1404172800000,
               "doc_count": 1
            },
            {
               "key_as_string": "2014-08-01",
               "key": 1406851200000,
               "doc_count": 1
            },
            {
               "key_as_string": "2014-10-01",
               "key": 1412121600000,
               "doc_count": 1
            },
            {
               "key_as_string": "2014-11-01",
               "key": 1414800000000,
               "doc_count": 2
            }
         ]
...
}
```
聚合结果已经完全展示了。正如你所见，我们有代表月份的 buckets，每个月的文档数目，以及美化后的 key_as_string 。   
   
