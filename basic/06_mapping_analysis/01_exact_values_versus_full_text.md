# [精确值 VS 全文](01_exact_values_versus_full_text.md)  
&emsp;&emsp;Elasticsearch 中的数据可以概括的分为两类：精确值和全文。

&emsp;&emsp;精确值 如它们听起来那样精确。例如日期或者用户 `ID`，但字符串也可以表示精确值，
例如用户名或邮箱地址。对于精确值来讲，`Foo` 和 `foo` 是不同的，`2014` 和 `2014-09-15` 也是不同的。

&emsp;&emsp;另一方面，全文 是指文本数据（通常以人类容易识别的语言书写），例如一个推文的内容或一封邮件的内容。
> NOTE
>>&emsp;&emsp;全文通常是指非结构化的数据，但这里有一个误解：自然语言是高度结构化的。
问题在于自然语言的规则是复杂的，导致计算机难以正确解析。例如，考虑这条语句：

>>&emsp;&emsp;`May is fun but June bores me.`     

>>&emsp;&emsp;它指的是月份还是人？

&emsp;&emsp;精确值很容易查询。结果是二进制的：要么匹配查询，要么不匹配。这种查询很容易用 SQL 表示：
```$xslt
WHERE name    = "John Smith"
  AND user_id = 2
  AND date    > "2014-09-15"
```

&emsp;&emsp;查询全文数据要微妙的多。我们问的不只是“这个文档匹配查询吗”，
而是“该文档匹配查询的程度有多大？”换句话说，该文档与给定查询的相关性如何？

&emsp;&emsp;我们很少对全文类型的域做精确匹配。
相反，我们希望在文本类型的域中搜索。不仅如此，我们还希望搜索能够理解我们的 意图 ：

 - 搜索 `UK` ，会返回包含 `United Kindom` 的文档。    
 - 搜索 `jump` ，会匹配 `jumped` ， `jumps` ， `jumping` ，甚至是 `leap` 。    
 - 搜索 `johnny walker` 会匹配 `Johnnie Walker` ， `johnnie depp` 应该匹配 `Johnny Depp` 。   
 
&emsp;&emsp;`fox news hunting` 应该返回福克斯新闻（ `Foxs News` ）中关于狩猎的故事，
同时， `fox hunting news `应该返回关于猎狐的故事。
为了促进这类在全文域中的查询，`Elasticsearch` 首先 `分析` 文档，之后根据结果创建 倒排索引 。
在接下来的两节，我们会讨论倒排索引和分析过程。