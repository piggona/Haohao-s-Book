# Elastic Stack学习-索引

> Elasticsearch为了增强搜索效率，使用了多种的索引与辅助工具。

## 正排&倒排索引

- ### 正排索引（书籍的目录）

  文档Id到文档内容、单词的关联关系

- ### 倒排索引（书籍的索引：关键词->页数）

  单词到文档Id的关联关系

>  倒排索引的查询流程:
>
> - 通过倒排索引获得”搜索引擎对应的文档Id 有1和3
> - 通过正排索引查询1和3的完整内容
> - 返回用户最终结果

## 倒排索引

- ### 单词词典（Term Dictionary）

  - 所有文档的单词
  - 记录单词到倒排列表的关联信息

  单词字典的实现一般是用B+ Tree:

- ### 倒排列表（Posting List）

  > 记录了单词对应的文档集合，由倒排索引项（Posting）组成

  ### 倒排索引项：

  - 文档Id，用于获取原始信息
  - 单词频率（TF,Term Frequency），记录该单词在该文档中的出现次数，用于后续相关性分析
  - 位置（position），记录单词在文档中的分词位置（哪个单词在哪个单词前面），用于做词语搜索（Phrase Query）
  - 偏移(Offset)，记录单词在文档的开始和结束位置，用于高亮显示

  ![image-20190118225929849](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190118225929849.png)

  ![image-20190118230053612](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190118230053612.png)

## 分词：

### 分词器的组成：

- Character Filters:针对原始文本进行处理，比如去除html特殊标记符
- Tokenizer：将原始文本按照一定规则切分为单词
- Token Filters：针对tokenizer处理的单词进行再加工，比如转小写、删除或新增等处理

## Analyze API

> 用于排查分词的问题，分词的效果
>
> - 可以指定analyzer进行测试
> - 直接指定索引中的字段进行测试
> - 可以自定义分词器进行测试

```
POST _analyze
{
    "analyzer":"standard",
    "text":"hello world!"
}
```

![image-20190121211405995](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190121211405995.png)

![image-20190121211843120](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190121211843120.png)

![image-20190121211856033](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190121211856033.png)

## ES自带的分词器：

### Standard Analyzer:

![image-20190121211945106](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190121211945106.png)

> 按词切分，支持多语言
>
> 小写处理

### Simple Analyzer

![image-20190121212204541](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190121212204541.png)

> 按词切分（只要出现非单词就切分）
>
> 变为小写

### Whitespace Analyzer

按照空格来切分

### Stop Analyzer

![image-20190121212430075](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190121212430075.png)

>  比simple analyzer多了停用词。

### Keyword Analyzer

> 不分词，直接作为一个单词输出

![image-20190121212550118](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190121212550118.png)

### Pattern Analyzer

通过正则表达式自定义分隔符（默认是\W+）非字词的符号作为分隔符

![image-20190121212748529](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190121212748529.png)

## 中文分词

### 常见分词系统：

- IK

  https://github.com/medcl/elasticsearch-analysis-ik

- jieba

  https://github.com/sing1ee/elasticsearch-jieba-plugin