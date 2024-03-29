# [字符串排序与多字段](02_string_sorting_multifields.md) 
&emsp;&emsp;被解析的字符串字段也是多值字段， 但是很少会按照你想要的方式进行排序。
如果你想分析一个字符串，如 `fine old art` ， 这包含 `3` 项。
我们很可能想要按第一项的字母排序，然后按第二项的字母排序，
诸如此类，但是 `Elasticsearch `在排序过程中没有这样的信息。

&emsp;&emsp;你可以使用 `min` 和 `max` 排序模式（默认是 `min` ），
但是这会导致排序以 `art` 或是 `old` ，任何一个都不是所希望的。

&emsp;&emsp;为了以字符串字段进行排序，这个字段应仅包含一项： 
整个 `not_analyzed `字符串。 但是我们仍需要 `analyzed` 字段，这样才能以全文进行查询

&emsp;&emsp;一个简单的方法是用两种方式对同一个字符串进行索引，
这将在文档中包括两个字段： `analyzed` 用于搜索， `not_analyzed` 用于排序

&emsp;&emsp;但是保存相同的字符串两次在 `_source` 字段是浪费空间的。 
我们真正想要做的是传递一个 单字段 但是却用两种方式索引它。
所有的 `_core_field` 类型 (`strings`, `numbers`, `Booleans`, `dates`) 接收一个 `fields` 参数

&emsp;&emsp;该参数允许你转化一个简单的映射如：
```
"tweet": {
    "type":     "string",
    "analyzer": "english"
}
```
&emsp;&emsp;为一个多字段映射如：
```
"tweet": {  (a)
    "type":     "string",
    "analyzer": "english",
    "fields": {
        "raw": {   (b)
            "type":  "string",
            "index": "not_analyzed"
        }
    }
}
```
`(a)tweet 主字段与之前的一样: 是一个 analyzed 全文字段。`   
`(b)新的 tweet.raw 子字段是 not_analyzed.`   

&emsp;&emsp;现在，至少只要我们重新索引了我们的数据，使用 `tweet` 字段用于搜索，`tweet.raw` 字段用于排序：
>⚠️警示
>>以全文 `analyzed` 字段排序会消耗大量的内存。获取更多信息请看 聚合与分析 。