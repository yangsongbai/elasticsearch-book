# [倒排索引](02_inverted_index.md) 
&emsp;&emsp;`Elasticsearch` 使用一种称为 倒排索引 的结构，它适用于快速的全文搜索。
一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表。

例如，假设我们有两个文档，每个文档的 `content` 域包含如下内容：

 1. The quick brown fox jumped over the lazy dog     
 2. Quick brown foxes leap over lazy dogs in summer      
   
&emsp;&emsp;为了创建倒排索引，我们首先将每个文档的 `content` 域拆分成单独的 词（我们称它为 词条 或 `tokens` ），
创建一个包含所有不重复词条的排序列表，然后列出每个词条出现在哪个文档。结果如下所示：

|  Term   | Doc_1  | Doc_2  |
|  ----  | ----  | ----  |
| Quick  |       | X |
| The  | X |  |
| brown  | X | X |
| dog  | X |  |
| dogs  |  | X |
| fox  | X |  |
| foxes  |  | X |
| in  |  | X |
| jumped  | X |  |
| lazy  | X | X |
| leap  |  | X |
| over  | X | X |
| quick  | X |  |
| summer  |  | X |
| the  | X |    |

现在，如果我们想搜索 quick brown ，我们只需要查找包含每个词条的文档：

|  Term   | Doc_1  | Doc_2  |
|  ----  | ----  | ----  |
| brown  |    X   | X |
| quick  | X |  |
|   |  |  |
| Total  | 2 | 1 |

&emsp;&emsp;两个文档都匹配，但是第一个文档比第二个匹配度更高。
如果我们使用仅计算匹配词条数量的简单 相似性算法 ，
那么，我们可以说，对于我们查询的相关性来讲，第一个文档比第二个文档更佳。

但是，我们目前的倒排索引有一些问题：

 - Quick 和 quick 以独立的词条出现，然而用户可能认为它们是相同的词。       
 - fox 和 foxes 非常相似, 就像 dog 和 dogs ；他们有相同的词根。      
 - jumped 和 leap, 尽管没有相同的词根，但他们的意思很相近。他们是同义词。     
   
&emsp;&emsp;使用前面的索引搜索 +Quick +fox 不会得到任何匹配文档。（记住，+ 前缀表明这个词必须存在。）
只有同时出现 Quick 和 fox 的文档才满足这个查询条件，但是第一个文档包含 quick fox ，
第二个文档包含 Quick foxes 。

&emsp;&emsp;我们的用户可以合理的期望两个文档与查询匹配。我们可以做的更好。

&emsp;&emsp;如果我们将词条规范为标准模式，那么我们可以找到与用户搜索的词条不完全一致，但具有足够相关性的文档。例如：

 - Quick 可以小写化为 quick 。    
 - foxes 可以 词干提取 --变为词根的格式-- 为 fox 。类似的， dogs 可以为提取为 dog 。    
 - jumped 和 leap 是同义词，可以索引为相同的单词 jump 。  
  
现在索引看上去像这样： 

|  Term   | Doc_1  | Doc_2  |
|  ----  | ----  | ----  |
| brown  |   X    | X |
| dog  | X | X |
| fox  | X | X |
| in  |  | X |
| jump  | X | X |
| lazy  | X | X |
| over  | X | X |
| quick  | X | X |
| summer  |  | X |
| the  | X | X |
 
&emsp;&emsp;这还远远不够。我们搜索 +Quick +fox 仍然 会失败，因为在我们的索引中，
已经没有 Quick 了。但是，如果我们对搜索的字符串使用与 content 域相同的标准化规则，
会变成查询 +quick +fox ，这样两个文档都会匹配！

> NOTE
>>&emsp;&emsp;这非常重要。你只能搜索在索引中出现的词条，所以索引文本和查询字符串必须标准化为相同的格式。  

&emsp;&emsp;分词和标准化的过程称为 分析 ， 我们会在下个章节讨论。