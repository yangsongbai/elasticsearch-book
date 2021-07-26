# [什么是相关性?](03_what_is_relevance.md) 
&emsp;&emsp;我们曾经讲过，默认情况下，返回结果是按相关性倒序排列的。 但是什么是相关性？ 相关性如何计算？   

&emsp;&emsp;每个文档都有相关性评分，用一个正浮点数字段 `_score` 来表示 。` _score` 的评分越高，相关性越高。   

&emsp;&emsp;查询语句会为每个文档生成一个 `_score` 字段。评分的计算方式取决于查询类型 
不同的查询语句用于不同的目的： `fuzzy` 查询会计算与关键词的拼写相似程度，
terms 查询会计算 找到的内容与关键词组成部分匹配的百分比，
但是通常我们说的 `relevance` 是我们用来计算全文本字段的值相对于全文本检索词相似程度的算法。

&emsp;&emsp;Elasticsearch 的相似度算法被定义为检索词频率/反向文档频率， `TF/IDF` ，包括以下内容：

**检索词频率**   
&emsp;&emsp;检索词在该字段出现的频率？出现频率越高，相关性也越高。 字段中出现过 `5` 次要比只出现过 `1` 次的相关性高。     
**反向文档频率**      
&emsp;&emsp;每个检索词在索引中出现的频率？频率越高，相关性越低。检索词出现在多数文档中会比出现在少数文档中的权重更低。    
**字段长度准则**     
&emsp;&emsp;字段的长度是多少？长度越长，相关性越低。 
检索词出现在一个短的 `title` 要比同样的词出现在一个长的 `content` 字段权重更大。 

&emsp;&emsp;单个查询可以联合使用 `TF/IDF` 和其他方式，比如短语查询中检索词的距离或模糊查询里的检索词相似度。

&emsp;&emsp;相关性并不只是全文本检索的专利。也适用于 `yes|no` 的子句，匹配的子句越多，相关性评分越高。

&emsp;&emsp;如果多条查询子句被合并为一条复合查询语句，比如 `bool` 查询，
则每个查询子句计算得出的评分会被合并到总的相关性评分中。

## 理解评分标准  
当调试一条复杂的查询语句时，想要理解 `_score` 究竟是如何计算是比较困难的。
`Elasticsearch` 在 每个查询语句中都有一个 `explain` 参数，将 `explain` 设为 `true` 就可以得到更详细的信息。