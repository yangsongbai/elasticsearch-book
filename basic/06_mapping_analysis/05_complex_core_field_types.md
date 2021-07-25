# [复杂核心域类型](05_complex_core_field_types.md)   
&emsp;&emsp;除了我们提到的简单标量数据类型， `JSON` 还有 `null` 值，数组，和对象，
这些 Elasticsearch 都是支持的。
## 多值域  
&emsp;&emsp;很有可能，我们希望 tag 域包含多个标签。我们可以以数组的形式索引标签：  
```
{ "tag": [ "search", "nosql" ]}
```
&emsp;&emsp;对于数组，没有特殊的映射需求。任何域都可以包含0、1或者多个值，就像全文域分析得到多个词条。

&emsp;&emsp;这暗示 数组中所有的值必须是相同数据类型的 。你不能将日期和字符串混在一起。
如果你通过索引数组来创建新的域，Elasticsearch 会用数组中第一个值的数据类型作为这个域的 类型 。
>NOTE
>>&emsp;&emsp;当你从 `Elasticsearch` 得到一个文档，每个数组的顺序和你当初索引文档时一样。
你得到的 `_source` 域，包含与你索引的一模一样的 `JSON` 文档。

>>&emsp;&emsp;但是，数组是以多值域 索引的—可以搜索，但是无序的。 在搜索的时候，
你不能指定 “第一个” 或者 “最后一个”。 更确切的说，把数组想象成 装在袋子里的值 。

## 空域 
&emsp;&emsp;当然，数组可以为空。这相当于存在零值。 事实上，在 `Lucene` 中是不能存储 `null` 值的，
所以我们认为存在 `null` 值的域为空域。

下面三种域被认为是空的，它们将不会被索引：  
```
"null_value":               null,
"empty_array":              [],
"array_with_null_value":    [ null ]
```
##  多层级对象 
&emsp;&emsp;我们讨论的最后一个 `JSON` 原生数据类是 对象 `--` 
在其他语言中称为哈希，哈希 `map`，字典或者关联数组。

&emsp;&emsp;内部对象 经常用于嵌入一个实体或对象到其它对象中。
例如，与其在 `tweet` 文档中包含 `user_name` 和 `user_id` 域，我们也可以这样写：  
```
{
    "tweet":            "Elasticsearch is very flexible",
    "user": {
        "id":           "@johnsmith",
        "gender":       "male",
        "age":          26,
        "name": {
            "full":     "John Smith",
            "first":    "John",
            "last":     "Smith"
        }
    }
}
```
## 内部对象的映射
&emsp;&emsp;Elasticsearch 会动态监测新的对象域并映射它们为 对象 ，在 properties 属性下列出内部域：
```
{
  "gb": {
    "tweet": {   (a)
      "properties": {
        "tweet":            { "type": "string" },
        "user": {  (b)
          "type":             "object",
          "properties": {
            "id":           { "type": "string" },
            "gender":       { "type": "string" },
            "age":          { "type": "long"   },
            "name":   {    (b)
              "type":         "object",
              "properties": {
                "full":     { "type": "string" },
                "first":    { "type": "string" },
                "last":     { "type": "string" }
              }
            }
          }
        }
      }
    }
  }
}
```
`(a)根对象`   
`(b)内部对象`    
&emsp;&emsp;`user` 和 `name` 域的映射结构与 `tweet` 类型的相同。事实上，
 `type` 映射只是一种特殊的 对象 映射，我们称之为 根对象 。
 除了它有一些文档元数据的特殊顶级域，例如 `_source` 和 `_all` 域，它和其他对象一样。  
 
 ## 内部对象是如何索引的 
&emsp;&emsp;`Lucene` 不理解内部对象。 `Lucene` 文档是由一组键值对列表组成的。
为了能让 `Elasticsearch` 有效地索引内部类，它把我们的文档转化成这样：  
```
{
    "tweet":            [elasticsearch, flexible, very],
    "user.id":          [@johnsmith],
    "user.gender":      [male],
    "user.age":         [26],
    "user.name.full":   [john, smith],
    "user.name.first":  [john],
    "user.name.last":   [smith]
}
```
&emsp;&emsp;内部域 可以通过名称引用（例如， `first` ）。
为了区分同名的两个域，我们可以使用全 路径 （例如， `user.name.first` ） 
或 `type` 名加路径（ `tweet.user.name.first` ）。  
>Note
>>在前面简单扁平的文档中，没有 `user` 和 `user.name` 域。`Lucene` 索引只有标量和简单值，没有复杂数据结构。   
## 内部对象是如何被索引的 
&emsp;&emsp;最后，考虑包含内部对象的数组是如何被索引的。 假设我们有个 followers 数组：  
```
{
    "followers": [
        { "age": 35, "name": "Mary White"},
        { "age": 26, "name": "Alex Jones"},
        { "age": 19, "name": "Lisa Smith"}
    ]
}
```
&emsp;&emsp;这个文档会像我们之前描述的那样被扁平化处理，结果如下所示：   
```
{
    "followers.age":    [19, 26, 35],
    "followers.name":   [alex, jones, lisa, smith, mary, white]
}
```
&emsp;&emsp;`{age: 35}` 和 `{name: Mary White}` 之间的相关性已经丢失了，
因为每个多值域只是一包无序的值，而不是有序数组。这足以让我们问，“有一个26岁的追随者？”

&emsp;&emsp;但是我们不能得到一个准确的答案：“是否有一个26岁 名字叫 Alex Jones 的追随者？”

&emsp;&emsp;相关内部对象被称为 `nested` 对象，可以回答上面的查询，我们稍后会在嵌套对象中介绍它。
