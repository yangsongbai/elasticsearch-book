# [映射](04_mapping.md)    
&emsp;&emsp;为了能够将时间域视为时间，数字域视为数字，字符串域视为全文或精确值字符串， 
Elasticsearch 需要知道每个域中数据的类型。这个信息包含在映射中。   

&emsp;&emsp;如 [数据输入和输出](data_in_data_out/introduction.md) 中解释的，
索引中每个文档都有 类型 。每种类型都有它自己的 映射 ，或者 模式定义 。
映射定义了类型中的域，每个域的数据类型，以及Elasticsearch如何处理这些域。映射也用于配置与类型有关的元数据。

&emsp;&emsp;我们会在 类型和映射 详细讨论映射。本节，我们只讨论足够让你入门的内容。     
## 核心简单域类型  
Elasticsearch 支持如下简单域类型：
 - 字符串: string  
 - 整数 : byte, short, integer, long   
 - 浮点数: float, double   
 - 布尔型: boolean   
 - 日期: date  
 
&emsp;&emsp;当你索引一个包含新域的文档—​之前未曾出现-- Elasticsearch 会使用 动态映射 ，
 通过JSON中基本数据类型，尝试猜测域类型，使用如下规则：
 
 |  JSON type   | 域 type  | 
 |  ----  | ----  | 
 | 布尔型: true 或者 false  |   boolean    |
 | 整数: 123  | long |
 | 浮点数: 123.45  | double |  
 | 字符串，有效日期: 2014-09-15  | date |
 | 字符串: foo bar  | string |
>NOTE
>>&emsp;&emsp;这意味着如果你通过引号( "123" )索引一个数字，它会被映射为 `string` 类型，
而不是 `long` 。但是，如果这个域已经映射为 `long` ，那么 `Elasticsearch` 
会尝试将这个字符串转化为 `long` ，如果无法转化，则抛出一个异常。   

## 查看映射  

&emsp;&emsp;通过 /_mapping ，我们可以查看 Elasticsearch 在一个或多个索引中的一个或多个类型的映射。
在 开始章节 ，我们已经取得索引 gb 中类型 tweet 的映射：  
```$xslt
GET /gb/_mapping/tweet
```
&emsp;&emsp;Elasticsearch 根据我们索引的文档，为域(称为 属性 )动态生成的映射。
```$xslt
{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "strict_date_optional_time||epoch_millis"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}
```
>Tip
>>错误的映射，例如 将 age 域映射为 string 类型，而不是 integer ，会导致查询出现令人困惑的结果。
  
>>检查一下！而不是假设你的映射是正确的。   

## 自定义域映射  
&emsp;&emsp;尽管在很多情况下基本域数据类型已经够用，但你经常需要为单独域自定义映射，
特别是字符串域。自定义映射允许你执行下面的操作：

 - 全文字符串域和精确值字符串域的区别 
 - 使用特定语言分析器   
 - 优化域以适应部分匹配   
 - 指定自定义数据格式   
 - 还有更多   
 
&emsp;&emsp;域最重要的属性是 `type` 。对于不是 `string` 的域，你一般只需要设置 `type` ：   
```
{
    "number_of_clicks": {
        "type": "integer"
    }
}
```
&emsp;&emsp;默认， `string` 类型域会被认为包含全文。就是说，它们的值在索引前，会通过一个分析器，
针对于这个域的查询在搜索前也会经过一个分析器。

`string` 域映射的两个最重要属性是 `index` 和 `analyzer` 。   
## index
index 属性控制怎样索引字符串。它可以是下面三个值：    

**analyzed**   
&emsp;&emsp;首先分析字符串，然后索引它。换句话说，以全文索引这个域。    
**not_analyzed**   
&emsp;&emsp;索引这个域，所以它能够被搜索，但索引的是精确值。不会对它进行分析。   
**no**   
&emsp;&emsp;不索引这个域。这个域不会被搜索到。  

&emsp;&emsp;`string` 域 `index` 属性默认是 `analyzed` 。
如果我们想映射这个字段为一个精确值，我们需要设置它为 `not_analyzed `：
```
{
    "tag": {
        "type":     "string",
        "index":    "not_analyzed"
    }
}
```
&emsp;&emsp;其他简单类型（例如 `long` ， `double` ， `date` 等）也接受 index 参数，
但有意义的值只有 `no` 和 `not_analyzed` ， 因为它们永远不会被分析。  

## analyzer
&emsp;&emsp;对于 `analyzed` 字符串域，用 `analyzer` 属性指定在搜索和索引时使用的分析器。
默认， Elasticsearch 使用 `standard` 分析器， 
但你可以指定一个内置的分析器替代它，例如 `whitespace` 、 `simple` 和 `english`：  
```
{
    "tweet": {
        "type":     "string",
        "analyzer": "english"
    }
}
```
在 自定义分析器 ，我们会展示怎样定义和使用自定义分析器。 
## 更新映射  
&emsp;&emsp;当你首次创建一个索引的时候，可以指定类型的映射。
你也可以使用 `/_mapping` 为新类型（或者为存在的类型更新映射）增加映射。  
>
>>&emsp;&emsp;尽管你可以 增加 一个存在的映射，你不能 修改 存在的域映射。
如果一个域的映射已经存在，那么该域的数据可能已经被索引。
如果你意图修改这个域的映射，索引的数据可能会出错，不能被正常的搜索。

&emsp;&emsp;我们可以更新一个映射来添加一个新域，但不能将一个存在的域从 `analyzed` 改为 `not_analyzed` 。

为了描述指定映射的两种方式，我们先删除 gd 索引：  
```
DELETE /gb
```
然后创建一个新索引，指定 `tweet` 域使用 `english` 分析器：  
```
PUT /gb (a)
{
  "mappings": {
    "tweet" : {
      "properties" : {
        "tweet" : {
          "type" :    "string",
          "analyzer": "english"
        },
        "date" : {
          "type" :   "date"
        },
        "name" : {
          "type" :   "string"
        },
        "user_id" : {
          "type" :   "long"
        }
      }
    }
  }
}
```
`(a)通过消息体中指定的 mappings 创建了索引。`

&emsp;&emsp;稍后，我们决定在 tweet 映射增加一个新的名为 tag 的 not_analyzed 的文本域，使用 _mapping ：
```
PUT /gb/_mapping/tweet
{
  "properties" : {
    "tag" : {
      "type" :    "string",
      "index":    "not_analyzed"
    }
  }
}
```
&emsp;&emsp;注意，我们不需要再次列出所有已存在的域，因为无论如何我们都无法改变它们。新域已经被合并到存在的映射中。   
## 测试映射  
你可以使用 analyze API 测试字符串域的映射。比较下面两个请求的输出：
```
GET /gb/_analyze
{
  "field": "tweet",
  "text": "Black-cats"  (a)
}

GET /gb/_analyze
{
  "field": "tag",
  "text": "Black-cats"  (a)
}
```
`(a) 消息体里面传输我们想要分析的文本。`      

&emsp;&emsp;`tweet` 域产生两个词条 `black`和 `cat` ， `tag` 域产生单独的词条 `Black-cats` 。
换句话说，我们的映射正常工作。