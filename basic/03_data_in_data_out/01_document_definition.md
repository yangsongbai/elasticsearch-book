# [什么是文档](01_document_definition.md) 
&emsp;&emsp;在大多数应用中，多数实体或对象可以被序列化为包含键值对的 JSON 对象。 
一个 键 可以是一个字段或字段的名称，一个 值 可以是一个字符串，一个数字，一个布尔值， 
另一个对象，一些数组值，或一些其它特殊类型诸如表示日期的字符串，或代表一个地理位置的对象：    
```$xslt
{
    "name":         "John Smith",
    "age":          42,
    "confirmed":    true,
    "join_date":    "2014-06-01",
    "home": {
        "lat":      51.5,
        "lon":      0.1
    },
    "accounts": [
        {
            "type": "facebook",
            "id":   "johnsmith"
        },
        {
            "type": "twitter",
            "id":   "johnsmith"
        }
    ]
}
```
&emsp;&emsp;通常情况下，我们使用的术语 对象 和 文档 是可以互相替换的。不过，有一个区别： 
一个对象仅仅是类似于 hash 、 hashmap 、字典或者关联数组的 JSON 对象，对象中也可以嵌套其他的对象。
对象可能包含了另外一些对象。在 Elasticsearch 中，术语 文档 有着特定的含义。
它是指最顶层或者根对象, 这个根对象被序列化成 JSON 并存储到 Elasticsearch 中，指定了唯一 ID。    
>⚠️警示
>>&emsp;&emsp;字段的名字可以是任何合法的字符串，但 不可以 包含英文句号(.)。
