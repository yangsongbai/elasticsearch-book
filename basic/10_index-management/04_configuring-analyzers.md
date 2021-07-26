# 配置分析器   
第三个重要的索引设置是 analysis 部分，用来配置已存在的分析器或针对你的索引创建新的自定义分析器。      
在 [分析与分析器](https://www.elastic.co/guide/cn/elasticsearch/guide/current/analysis-intro.html) ，
我们介绍了一些内置的分析器，用于将全文字符串转换为适合搜索的倒排索引。   

standard 分析器是用于全文字段的默认分析器，对于大部分西方语系来说是一个不错的选择。 它包括了以下几点：

 - standard 分词器，通过单词边界分割输入的文本。   
 - standard 语汇单元过滤器，目的是整理分词器触发的语汇单元（但是目前什么都没做）。   
 - lowercase 语汇单元过滤器，转换所有的语汇单元为小写。   
 - stop 语汇单元过滤器，删除停用词—​对搜索相关性影响不大的常用词，如 a ， the ， and ， is 。   

默认情况下，停用词过滤器是被禁用的。如需启用它，你可以通过创建一个基于 standard 分析器的自定义分析器并设置 stopwords 参数。 
可以给分析器提供一个停用词列表，或者告知使用一个基于特定语言的预定义停用词列表。

在下面的例子中，我们创建了一个新的分析器，叫做 es_std ， 并使用预定义的西班牙语停用词列表：   
```
PUT /spanish_docs
{
    "settings": {
        "analysis": {
            "analyzer": {
                "es_std": {
                    "type":      "standard",
                    "stopwords": "_spanish_"
                }
            }
        }
    }
}

```   
es_std 分析器不是全局的—​它仅仅存在于我们定义的 spanish_docs 索引中。 
为了使用 analyze API来对它进行测试，我们必须使用特定的索引名：     
```
GET /spanish_docs/_analyze?analyzer=es_std
El veloz zorro marrón
```      
简化的结果显示西班牙语停用词 El 已被正确的移除：     
```
{
  "tokens" : [
    { "token" :    "veloz",   "position" : 2 },
    { "token" :    "zorro",   "position" : 3 },
    { "token" :    "marrón",  "position" : 4 }
  ]
}
```
