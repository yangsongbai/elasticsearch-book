# [轻量 搜索](04_search_lite.md)   
&emsp;&emsp;有两种形式的 搜索 API：一种是 “轻量的” 查询字符串 版本，要求在查询字符串中传递所有的参数，
另一种是更完整的 请求体 版本，要求使用 JSON 格式和更丰富的查询表达式作为搜索语言。

&emsp;&emsp;查询字符串搜索非常适用于通过命令行做即席查询。
例如，查询在 tweet 类型中 tweet 字段包含 elasticsearch 单词的所有文档：
```$xslt
GET /_all/tweet/_search?q=tweet:elasticsearch
```
&emsp;&emsp;下一个查询在 name 字段中包含 john 并且在 tweet 字段中包含 mary 的文档。实际的查询就是这样

`+name:john +tweet:mary`
但是查询字符串参数所需要的 百分比编码 （译者注：URL编码）实际上更加难懂：
```$xslt
GET /_search?q=%2Bname%3Ajohn+%2Btweet%3Amary
```
&emsp;&emsp;`+` 前缀表示必须与查询条件匹配。类似地， `-` 前缀表示一定不与查询条件匹配。
没有 `+` 或者 `-` 的所有其他条件都是可选的——匹配的越多，文档就越相关。     

# _all
&emsp;&emsp;这个简单搜索返回包含 `mary` 的所有文档：
```$xslt
GET /_search?q=mary
```
之前的例子中，我们在 `tweet` 和 `name` 字段中搜索内容。然而，这个查询的结果在三个地方提到了 `mary` ：

 - 有一个用户叫做 Mary    
 - 6条微博发自 Mary    
 - 一条微博直接 @mary  

&emsp;&emsp;Elasticsearch 是如何在三个不同的字段中查找到结果的呢？

&emsp;&emsp;当索引一个文档的时候，`Elasticsearch` 取出所有字段的值拼接成一个大的字符串，
作为 `_all` 字段进行索引。例如，当索引这个文档时：
```$xslt
{
    "tweet":    "However did I manage before Elasticsearch?",
    "date":     "2014-09-14",
    "name":     "Mary Jones",
    "user_id":  1
}
```

这就好似增加了一个名叫 `_all` 的额外字段：
```$xslt
"However did I manage before Elasticsearch? 2014-09-14 Mary Jones 1"
```
> Tip
>>&emsp;&emsp;在刚开始开发一个应用时，_all 字段是一个很实用的特性。
之后，你会发现如果搜索时用指定字段来代替 _all 字段，将会更好控制搜索结果。
当 _all 字段不再有用的时候，可以将它置为失效，正如在 元数据: _all 字段 中所解释的。

## 更复杂的查询
下面的查询针对`tweents`类型，并使用以下的条件：

 - name 字段中包含 mary 或者 john
 - date 值大于 2014-09-10
 - _all 字段包含 aggregations 或者 geo

查询字符串在做了适当的编码后，可读性很差：
```$xslt
?q=%2Bname%3A(mary+john)+%2Bdate%3A%3E2014-09-10+%2B(aggregations+geo)
```
&emsp;&emsp;从之前的例子中可以看出，这种 轻量 的查询字符串搜索效果还是挺让人惊喜的。 
它的查询语法在相关参考文档中有详细解释，以便简洁的表达很复杂的查询。
对于通过命令做一次性查询，或者是在开发阶段，都非常方便。

&emsp;&emsp;但同时也可以看到，这种精简让调试更加晦涩和困难。
而且很脆弱，一些查询字符串中很小的语法错误，像 `- `， `:` ， `/` 或者 `" `不匹配等，将会返回错误而不是搜索结果。

&emsp;&emsp;最后，查询字符串搜索允许任何用户在索引的任意字段上执行可能较慢且重量级的查询，这可能会暴露隐私信息，甚至将集群拖垮。

>Tip 
>>&emsp;&emsp;因为这些原因，不推荐直接向用户暴露查询字符串搜索功能，除非对于集群和数据来说非常信任他们。

&emsp;&emsp;相反，我们经常在生产环境中更多地使用功能全面的 request body 查询API，
除了能完成以上所有功能，还有一些附加功能。但在到达那个阶段之前，首先需要了解数据在 Elasticsearch 中是如何被索引的。
