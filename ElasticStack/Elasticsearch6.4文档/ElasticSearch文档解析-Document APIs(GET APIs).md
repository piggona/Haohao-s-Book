

# ElasticSearch文档解析-Document APIs(GET APIs)

## 1. GET API

用户可以通过get API去通过{index}/{type}/{id}或者高级query查询语句去获取对应的完整搜索结果，例：

```bash
//{index}:twitter
//{type}:_doc
//{id}:0

GET twitter/_doc/0
```

返回：

```json
{
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "0",
    "_version" : 1,
    "found": true,
    "_source" : {
        "user" : "kimchy",
        "date" : "2009-11-15T14:12:12",
        "likes": 0,
        "message" : "trying out Elasticsearch"
    }
}
```

如果没有部分显示选项，那么显示的就是_source即源数据。

用户也可以通过HEAD API来确认是否有对应id的document是否在该type的index中，例：

```bash
HEAD twitter/_doc/0
```

## 2. 实时性(RealTime)

默认情况下，get API都提供实时服务，这就意味着它不受index刷新频率（index数据改变到可供查询visible的频率）的影响。实现这种服务的方式，简单来说就是当elasticsearch发现搜索的document发生改变但没有visible的时候，立即发出in-place call使数据马上可见。如果需要关闭这种功能，那么可以通过调整参数$realtime$为false方式关闭这个特性。

## 3. Source字段的返回筛选(source filtering)

> elasticsearch查询中，如果要获取全部的_source内容，有时是非常耗费资源的。我们可以通过设置_source=false或者设置\_source_include&\_source_exclude来节省网络资源。

设置_source=false,我们将无法获得source字段

```bash
GET twitter/_doc/0?_source=false
```

设置\_source_include&_source_exclude来筛选我们需要的field作为_source返回项。

```bash
GET twitter/_doc/0?_source_include=*.id&_source_exclude=entities
```

## 4. Source字段的store选项（Store Fields)

在elasticsearch的搜索过程中，使用_source是默认也是最高效的方法，因为只需要一次IO。即使当我们只需要搜索几个字段(fields)时，也可以指定elasticsearch从\_source中取需要的字段返回。如果非常特殊的情况，<font color=#FF0033>文档（documents)的长度很长，存储_source或者从_source中获取field的代价很大，你可以显式的将某些field的store属性设置为yes.</font>

[用法详细解读](https://blog.csdn.net/jingkyks/article/details/41785887)

### 实现方法：

- 首先在mapping中要设置store fields字段

  ```bash
  PUT twitter
  {
     "mappings": {
        "_doc": {
           "properties": {
              "counter": {
                 "type": "integer",
                 "store": false
              },
              "tags": {
                 "type": "keyword",
                 "store": true
              }
           }
        }
     }
  }
  ```

- 加入一个doc

  ```
  PUT twitter/_doc/1
  {
      "counter" : 1,
      "tags" : ["red"]
  }
  ```

- 之后就可以直接查询field的值

  > 如果使用_source来存储查询，elasticsearch是不能直接存储或查询某个特定的field的，获取某field的过程是1.查询到对应index的source 2.从\_source中提取出相应的field 

  ```ba&amp;#39;s
  GET twitter/_doc/1?stored_fields=tags,counter
  ```

  查询结果：

  ```json
  {
     "_index": "twitter",
     "_type": "_doc",
     "_id": "1",
     "_version": 1,
     "found": true,
     "fields": {
        "tags": [
           "red"
        ]
     }
  }
  ```



## 5. Routing 功能

> 简而言之，就是对不同的routing显示不同的结果。输入不正确的routing或者不输入routing就会查询报错。

例：

```bash
GET twitter/_doc/2?routing=user1
```

## 6. 查询偏好（Preference)

有时候，primary shard的查询速度比较缓慢,所以elasticsearch也提供了为当前request指定shard的功能。

- \_primary(default):使用primary shard
- \_local:尽量使用使用本地的shard

## 7. 刷新(Refresh)

将refresh的值设置为true,则每次搜索之前elasticsearch都会刷新一遍相关的shard来保证数据最新(searchable)

## 8. 数据版本控制（Versioning Support)

我们可以使用设置version关键词的设置，进行documents的精确查询(retrive).

内部来说，旧版本的documents当被替换时不会立即被删除，但已经无法访问。