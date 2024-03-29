# [分析与分析器](03_analysis_and_analyzers.md)   
分析 包含下面的过程：

 - 首先，将一块文本分成适合于倒排索引的独立的 词条 ，  
 - 之后，将这些词条统一化为标准格式以提高它们的“可搜索性”，或者 recall   
 
分析器执行上面的工作。 分析器 实际上是将三个功能封装到了一个包里：  

**字符过滤器**   
&emsp;&emsp;首先，字符串按顺序通过每个 字符过滤器 。他们的任务是在分词前整理字符串。
一个字符过滤器可以用来去掉`HTML`，或者将 `&` 转化成 `and`。
**分词器**   
&emsp;&emsp;其次，字符串被 分词器 分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条。
**`Token`过滤器**
&emsp;&emsp;最后，词条按顺序通过每个   `token`  过滤器 。这个过程可能会改变词条（例如，小写化 Quick ），
删除词条（例如， 像 `a`， `and`， `the` 等无用词），或者增加词条（例如，像 `jump` 和 `leap` 这种同义词）。    

&emsp;&emsp;Elasticsearch提供了开箱即用的字符过滤器、分词器和token 过滤器。 
这些可以组合起来形成自定义的分析器以用于不同的目的。我们会在 自定义分析器 章节详细讨论。  

## 内置分析器  

&emsp;&emsp;但是， Elasticsearch还附带了可以直接使用的预包装的分析器。接下来我们会列出最重要的分析器。
为了证明它们的差异，我们看看每个分析器会从下面的字符串得到哪些词条：

`"Set the shape to semi-transparent by calling set_trans(5)"`

**标准分析器**  
&emsp;&emsp;标准分析器是Elasticsearch默认使用的分析器。它是分析各种语言文本最常用的选择。
它根据 `Unicode` 联盟 定义的 单词边界 划分文本。删除绝大部分标点。最后，将词条小写。它会产生

`set, the, shape, to, semi, transparent, by, calling, set_trans, 5`

**简单分析器**  
&emsp;&emsp;简单分析器在任何不是字母的地方分隔文本，将词条小写。它会产生

`set, the, shape, to, semi, transparent, by, calling, set, trans`

**空格分析器**  
&emsp;&emsp;空格分析器在空格的地方划分文本。它会产生

`Set, the, shape, to, semi-transparent, by, calling, set_trans(5)`

**语言分析器**   
&emsp;&emsp;特定语言分析器可用于 很多语言。它们可以考虑指定语言的特点。
例如， 英语 分析器附带了一组英语无用词（常用单词，例如 `and` 或者 `the` ，它们对相关性没有多少影响），
它们会被删除。 由于理解英语语法的规则，这个分词器可以提取英语单词的 词干 。

英语 分词器会产生下面的词条：

`set, shape, semi, transpar, call, set_tran, 5`

&emsp;&emsp;注意看 `transparent`、 `calling `和 `set_trans` 已经变为词根格式。   

## 什么时候使用分词器  

&emsp;&emsp;当我们 `索引` 一个文档，它的全文域被分析成词条以用来创建倒排索引。 
但是，当我们在全文域 `搜索` 的时候，我们需要将查询字符串通过 相同的分析过程 ，
以保证我们搜索的词条格式与索引中的词条格式一致。

&emsp;&emsp;全文查询，理解每个域是如何定义的，因此它们可以做正确的事：

 - 当你查询一个 `全文` 域时， 会对查询字符串应用相同的分析器，以产生正确的搜索词条列表。  
 - 当你查询一个 `精确值` 域时，不会分析查询字符串，而是搜索你指定的精确值。   
 
现在你可以理解在 开始章节 的查询为什么返回那样的结果：

 - `date` 域包含一个精确值：单独的词条 `2014-09-15`。  
 - `_all` 域是一个全文域，所以分词进程将日期转化为三个词条： `2014`， `09`， 和 `15`。   
 
当我们在 `_all` 域查询 `2014`，它匹配所有的12条推文，因为它们都含有 `2014` ：
```$xslt
GET /_search?q=2014              # 12 results
```
&emsp;&emsp;当我们在 `_all` 域查询 `2014-09-15`，它首先分析查询字符串，
产生匹配 `2014`， `09`， 或 `15` 中 任意 词条的查询。这也会匹配所有12条推文，因为它们都含有 `2014` ：
```$xslt
GET /_search?q=2014-09-15        # 12 results !
```
&emsp;&emsp;当我们在 `date` 域查询 `2014-09-15`，它寻找 `精确` 日期，只找到一个推文：
```$xslt
GET /_search?q=date:2014-09-15   # 1  result
```
&emsp;&emsp;当我们在 `date` 域查询 `2014`，它找不到任何文档，因为没有文档含有这个精确日志：
```$xslt
GET /_search?q=date:2014         # 0  results !
```
## 测试分词器
&emsp;&emsp;有些时候很难理解分词的过程和实际被存储到索引中的词条，
特别是你刚接触`Elasticsearch`。为了理解发生了什么，
你可以使用 `analyze API` 来看文本是如何被分析的。在消息体里，指定分析器和要分析的文本：  
```$xslt
GET /_analyze
{
  "analyzer": "standard",
  "text": "Text to analyze"
}
```
结果中每个元素代表一个单独的词条：
```$xslt
{
   "tokens": [
      {
         "token":        "text",
         "start_offset": 0,
         "end_offset":   4,
         "type":         "<ALPHANUM>",
         "position":     1
      },
      {
         "token":        "to",
         "start_offset": 5,
         "end_offset":   7,
         "type":         "<ALPHANUM>",
         "position":     2
      },
      {
         "token":        "analyze",
         "start_offset": 8,
         "end_offset":   15,
         "type":         "<ALPHANUM>",
         "position":     3
      }
   ]
}
```
&emsp;&emsp;`token` 是实际存储到索引中的词条。 `position `指明词条在原始文本中出现的位置。
 `start_offset` 和 `end_offset` 指明字符在原始字符串中的位置。
>NOTE
>>每个分析器的 `type` 值都不一样，可以忽略它们。它们在Elasticsearch中的唯一作用在于
[​keep_types token](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/analysis-keyword-marker-tokenfilter.html) 过滤器​。

&emsp;&emsp;analyze API 是一个有用的工具，它有助于我们理解Elasticsearch索引内部发生了什么，
随着深入，我们会进一步讨论它。

## 指定分词器  
&emsp;&emsp;当Elasticsearch在你的文档中检测到一个新的字符串域，
它会自动设置其为一个全文 `字符串` 域，使用 标准 分析器对它进行分析。

&emsp;&emsp;你不希望总是这样。可能你想使用一个不同的分析器，适用于你的数据使用的语言。
有时候你想要一个字符串域就是一个字符串域—​不使用分析，直接索引你传入的精确值，
例如用户ID或者一个内部的状态域或标签。

&emsp;&emsp;要做到这一点，我们必须手动指定这些域的映射。



