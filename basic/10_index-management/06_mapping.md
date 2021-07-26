# 类型和映射      
类型 在 Elasticsearch 中表示一类相似的文档。 类型由 名称 —比如 user 或 blogpost —和 映射 组成。

映射, 就像数据库中的 schema ，描述了文档可能具有的字段或 属性 、每个字段的数据类型—比如 string, integer 或 date 
—以及Lucene是如何索引和存储这些字段的。

类型可以很好的抽象划分相似但不相同的数据。但由于 Lucene 的处理方式，类型的使用有些限制。       

## Lucene 如何处理文档   
在 Lucene 中，一个文档由一组简单的键值对组成。 每个字段都可以有多个值，但至少要有一个值。 
类似的，一个字符串可以通过分析过程转化为多个值。
Lucene 不关心这些值是字符串、数字或日期—​所有的值都被当做 不透明字节 。

当我们在 Lucene 中索引一个文档时，每个字段的值都被添加到相关字段的倒排索引中。
你也可以将未处理的原始数据 存储 起来，以便这些原始数据在之后也可以被检索到。        
## 类型是如何实现的    
Elasticsearch 类型是以 Lucene 处理文档的这个方式为基础来实现的。一个索引可以有多个类型，这些类型的文档可以存储在相同的索引中。

Lucene 没有文档类型的概念，每个文档的类型名被存储在一个叫 _type 的元数据字段上。 
当我们要检索某个类型的文档时, Elasticsearch 通过在 _type 字段上使用过滤器限制只返回这个类型的文档。

Lucene 也没有映射的概念。 映射是 Elasticsearch 将复杂 JSON 文档 映射 成 Lucene 需要的扁平化数据的方式。

例如，在 user 类型中， name 字段的映射可以声明这个字段是 string 类型，
并且它的值被索引到名叫 name 的倒排索引之前，需要通过 whitespace 分词器分析：      
```
"name": {
    "type":     "string",
    "analyzer": "whitespace"
}
```
# 避免类型陷阱  
这导致了一个有趣的思想实验： 如果有两个不同的类型，每个类型都有同名的字段，但映射不同（例如：一个是字符串一个是数字），将会出现什么情况？

简单回答是，Elasticsearch 不会允许你定义这个映射。当你配置这个映射时，将会出现异常。

详细回答是，每个 Lucene 索引中的所有字段都包含一个单一的、扁平的模式。
一个特定字段可以映射成 string 类型也可以是 number 类型，但是不能两者兼具。
因为类型是 Elasticsearch 添加的 优于 Lucene 的额外机制（以元数据 _type 字段的形式），
在 Elasticsearch 中的所有类型最终都共享相同的映射。

以 data 索引中两种类型的映射为例：  
```
{
   "data": {
      "mappings": {
         "people": {
            "properties": {
               "name": {
                  "type": "string",
               },
               "address": {
                  "type": "string"
               }
            }
         },
         "transactions": {
            "properties": {
               "timestamp": {
                  "type": "date",
                  "format": "strict_date_optional_time"
               },
               "message": {
                  "type": "string"
               }
            }
         }
      }
   }
}
```  
每个类型定义两个字段 (分别是 "name"/"address" 和 "timestamp"/"message" )。它们看起来是相互独立的，但在后台 Lucene 将创建一个映射，如:    
```
{
   "data": {
      "mappings": {
        "_type": {
          "type": "string",
          "index": "not_analyzed"
        },
        "name": {
          "type": "string"
        }
        "address": {
          "type": "string"
        }
        "timestamp": {
          "type": "long"
        }
        "message": {
          "type": "string"
        }
      }
   }
}
```    
注: 这不是真实有效的映射语法，只是用于演示

对于整个索引，映射在本质上被 扁平化 成一个单一的、全局的模式。
这就是为什么两个类型不能定义冲突的字段：当映射被扁平化时，Lucene 不知道如何去处理。
 
## 类型结论  
那么，这个讨论的结论是什么？技术上讲，多个类型可以在相同的索引中存在，只要它们的字段不冲突（要么因为字段是互为独占模式，要么因为它们共享相同的字段）。    

重要的一点是: 类型可以很好的区分同一个集合中的不同细分。在不同的细分中数据的整体模式是相同的（或相似的）。  

类型不适合 完全不同类型的数据 。如果两个类型的字段集是互不相同的，这就意味着索引中将有一半的数据是空的（字段将是 稀疏的 ），最终将导致性能问题。在这种情况下，最好是使用两个单独的索引。

总结：

 - *正确*:     
   将 kitchen 和 lawn-care 类型放在 products 索引中, 因为这两种类型基本上是相同的模式         
 - *错误*:     
   将 products 和 logs 类型放在 data 索引中, 因为这两种类型互不相同。应该将它们放在不同的索引中。


