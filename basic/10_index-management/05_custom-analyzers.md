# 自定义分析器      
虽然Elasticsearch带有一些现成的分析器，然而在分析器上Elasticsearch真正的强大之处在于，
你可以通过在一个适合你的特定数据的设置之中组合字符过滤器、分词器、词汇单元过滤器来创建自定义的分析器。     

在 [分析与分析器](https://www.elastic.co/guide/cn/elasticsearch/guide/current/analysis-intro.html) 
我们说过，一个 分析器 就是在一个包里面组合了三种函数的一个包装器， 三种函数按照顺序被执行:   

 - 字符过滤器
字符过滤器 用来 整理 一个尚未被分词的字符串。例如，如果我们的文本是HTML格式的，它会包含像 <p> 或者 <div> 这样的HTML标签，
这些标签是我们不想索引的。我们可以使用 [html清除 字符过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-htmlstrip-charfilter.html) 
来移除掉所有的HTML标签，并且像把 &Aacute; 
转换为相对应的Unicode字符 Á 这样，转换HTML实体。      

一个分析器可能有0个或者多个字符过滤器。     

 - 分词器       
一个分析器 必须 有一个唯一的分词器。 分词器把字符串分解成单个词条或者词汇单元。 
标准 分析器里使用的 [标准 分词器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-standard-tokenizer.html) 
把一个字符串根据单词边界分解成单个词条，并且移除掉大部分的标点符号，
然而还有其他不同行为的分词器存在。      

例如， [关键词 分词器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-keyword-tokenizer.html) 
完整地输出 接收到的同样的字符串，并不做任何分词。 [空格 分词器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-whitespace-tokenizer.html) 
只根据空格分割文本 。 
正则 分词器 根据匹配正则表达式来分割文本 。

 - 词单元过滤器       
经过分词，作为结果的 词单元流 会按照指定的顺序通过指定的词单元过滤器 。         

词单元过滤器可以修改、添加或者移除词单元。我们已经提到过 [lowercase](http://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lowercase-tokenizer.html) 
和 [stop 词过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-stop-tokenfilter.html) ，
但是在 Elasticsearch 里面还有很多可供选择的词单元过滤器。 词干过滤器 把单词 [遏制](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-stemmer-tokenfilter.html) 
为 词干。 
[ascii_folding](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-asciifolding-tokenfilter.html) 
过滤器移除变音符，把一个像 "très" 这样的词转换为 "tres" 。 [ngram](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-ngram-tokenfilter.html) 
和 [edge_ngram 词单元过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-edgengram-tokenfilter.html) 
可以产生 适合用于部分匹配或者自动补全的词单元。

在 [深入搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/current/search-in-depth.html)，
我们讨论了在哪里使用，以及怎样使用分词器和过滤器。但是首先，我们需要解释一下怎样创建自定义的分析器。   
## 创建一个自定义分词器       
和我们之前配置 es_std 分析器一样，我们可以在 analysis 下的相应位置设置字符过滤器、分词器和词单元过滤器:     
```
PUT /my_index
{
    "settings": {
        "analysis": {
            "char_filter": { ... custom character filters ... },
            "tokenizer":   { ...    custom tokenizers     ... },
            "filter":      { ...   custom token filters   ... },
            "analyzer":    { ...    custom analyzers      ... }
        }
    }
}
```     
作为示范，让我们一起来创建一个自定义分析器吧，这个分析器可以做到下面的这些事:    
 1. 使用 html清除 字符过滤器移除HTML部分。       
 2. 使用一个自定义的 映射 字符过滤器把 & 替换为 " and " ：     
```
"char_filter": {
    "&_to_and": {
        "type":       "mapping",
        "mappings": [ "&=> and "]
    }
}
```    
 3. 使用 标准 分词器分词。     
 4. 小写词条，使用 小写 词过滤器处理。     
 5. 使用自定义 停止 词过滤器移除自定义的停止词列表中包含的词：     
```
"filter": {
    "my_stopwords": {
        "type":        "stop",
        "stopwords": [ "the", "a" ]
    }
}
```    
我们的分析器定义用我们之前已经设置好的自定义过滤器组合了已经定义好的分词器和过滤器：        
```
"analyzer": {
    "my_analyzer": {
        "type":           "custom",
        "char_filter":  [ "html_strip", "&_to_and" ],
        "tokenizer":      "standard",
        "filter":       [ "lowercase", "my_stopwords" ]
    }
}
```        
汇总起来，完整的 创建索引 请求 看起来应该像这样：      
```
PUT /my_index
{
    "settings": {
        "analysis": {
            "char_filter": {
                "&_to_and": {
                    "type":       "mapping",
                    "mappings": [ "&=> and "]
            }},
            "filter": {
                "my_stopwords": {
                    "type":       "stop",
                    "stopwords": [ "the", "a" ]
            }},
            "analyzer": {
                "my_analyzer": {
                    "type":         "custom",
                    "char_filter":  [ "html_strip", "&_to_and" ],
                    "tokenizer":    "standard",
                    "filter":       [ "lowercase", "my_stopwords" ]
            }}
}}}
```         
索引被创建以后，使用 analyze API 来 测试这个新的分析器：       
```
GET /my_index/_analyze?analyzer=my_analyzer
The quick & brown fox
```      
下面的缩略结果展示出我们的分析器正在正确地运行：       
```
{
  "tokens" : [
      { "token" :   "quick",    "position" : 2 },
      { "token" :   "and",      "position" : 3 },
      { "token" :   "brown",    "position" : 4 },
      { "token" :   "fox",      "position" : 5 }
    ]
}
```       
这个分析器现在是没有多大用处的，除非我们告诉 Elasticsearch在哪里用上它。我们可以像下面这样把这个分析器应用在一个 string 字段上：        
```
PUT /my_index/_mapping/my_type
{
    "properties": {
        "title": {
            "type":      "string",
            "analyzer":  "my_analyzer"
        }
    }
}
```