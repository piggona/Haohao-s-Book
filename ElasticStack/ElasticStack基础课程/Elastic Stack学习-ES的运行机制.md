# Elastic Stack学习-ES的运行机制

## Search的运行机制：Query+Fetch

![image-20190223151807128](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190223151807128.png)

> 在query阶段中，因为系统不知道前from+size个文档在三个shard中的分布，所以三个shard都会搜索from+size个文档，之后再统一整合。
>
> 此阶段获得的是文档id和排序值。
>
> 下一阶段Fetch是根据query的结果得到查询到的Documents

![image-20190223152034847](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190223152034847.png)

## 相关性算分：

> 相关性算分在一个shard上是通过BM25算法进行一个搜索结果的算分，而在分布式系统上，它是不一样的

![image-20190223162649997](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190223162649997.png)

![image-20190223162718826](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190223162718826.png)

### 解决方法1：

```json
PUT my_index
{
    "settings":
    {
        "index":
        {
            "number_of_shards":1
        }
    }
}
```

### 解决方法2：

> DFS Query-then-Fetch:一定要在数据量不是特别大的时候使用，当数据量大的时候相关性算分已经比较准确，没必要dfs_query_then_fetch

![image-20190223163035939](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190223163035939.png)

## 排序：

![image-20190227083843878](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190227083843878.png)

> 先按照自定义的birth字段的值倒序排序，如果birth字段相等，则按照_score(相关性算分排序)，如果\_score相等，则按照\_doc(文档内部id)大小排序。

有两种排序的查询方式：

```json
GET test_search_index/_search
{
    "query":
    {
        "match":
        {
            "username":"alfred"
        }
    },
    "sort":
    {
        "birth" :"desc"
    }
}
```

> 这样的查询排序方式，是不会有相关性算分的

```
GET test_search_index/_search
{
    "query":
    {
        "match":
        {
            "username":"alfred"
        }
    },
    "sort":
    [
        {"birth":"desc"},
        {"_score":"desc"},
        {"_doc":"desc"}
    ]
}
```

> 这样就会有相关性算分

### 对字符串类型的排序

对字符串类型，一般用keyword字段（text字段中一般会自带的类型）来进行排序

```json
GET test_search_index/_search
{
    "sort":
    [
    {
        "username.keyword": "desc"
    }
    ]
}
```

### elasticsearch的排序过程

> 排序的过程实质是对字段的原始内容进行排序的过程，这个过程中倒排索引（term->document id)是无法发挥作用的，需要使用正排索引（document id->content)

#### ES的两种实现方式

![image-20190227093751019](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190227093751019.png)

- fieldData:默认禁用，一般在聚合分析时开启，只能在字符串类型中开启

  ![image-20190227094231702](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190227094231702.png)

- doc values:默认启用，除了text类型

  > 实现的时候会用列式存储的实现技术，对字段值存储的时候会使用压缩算法来节省存储空间

  ![image-20190227095530783](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190227095530783.png)


![image-20190227093927293](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190227093927293.png)

![image-20190227095636423](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190227095636423.png)

## 分页与遍历

### from/size

- from指明开始位置
- size指明获取总数

![image-20190227100254971](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190227100254971.png)

#### 分页带来的问题（分布式的问题）

##### 深度分页

![image-20190227100612289](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190227100612289.png)



### scroll

![image-20190227103116026](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190227103116026.png)

#### 此方式采用迭代的方式进行：

1. 发起一个scroll_search

   ```json
   GET test_search_index/_search?scroll=5m
   {
       "size":1
   }
   ```

   es在收到请求（建立一个5m时间的快照并返回一个文档）会根据查询条件创建文档Id合集的快照

   ![image-20190227104833189](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190227104833189.png)

2. 调用scroll_search的api，获取文档集合，迭代建立快照返回结果：

   ```json
   POST _search/scroll
   {
       "scroll":"5m",
       "scroll_id":"....上一步返回的id"
   }
   ```

   ![image-20190227105051642](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190227105051642.png)

   这样就可以没有限定一直获取文档。

### search_after

![image-20190227105546808](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190227105546808.png)

 