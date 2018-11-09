# ElasticSearch文档解析-Document APIs(文档操作APIs)2:Index API

Index API将一个格式化的Json document插入或更新到指定的Index中，使得它之后可以被搜索到。下面的例子进行的操作是：将一个Json document插入类型(type)为`_doc`的`twitter`index中，指定它的id为1

```json
PUT twitter/_doc/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

进行上面操作的返回结果是：

```json
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 1,
    "_seq_no" : 0,
    "_primary_term" : 1,
    "result" : "created"
}
```

返回值中的`_shard`项提供了elasticsearch在进行indexing的时候的replication操作进程：

- `total` - 说明这个indexing操作要在多少个shard备份中执行
- `successful` - 说明indexing 成功的shard数量
- `failed` - 对应successful的数量

只要successful值大于等于1，那么indexing操作就算成功。

> 只有primary shard返回indexing成功的信号，Replica shard才会被启动来进行操作。（这种规则可以被配置改变）这种情况下，`total`字段会等于在系统中设定的`number of replicas`。而`successful`字段等于系统中启用的shard数量(primary加replicas)。如果启动及indexing的过程中没有失败，则`failed`字段为0

## 自动index创建

index操作将新建一个index如果这个index没有被建立过，index操作也会新建一个新的类型如果这种类型没有被创建。

`mapping`本身是一个非常灵活没有固定架构的数据结构，如果用户在使用PUT命令时加入了mapping无法识别的格式，那么相应更新的字段及对象会加入到对应mapping的定义中。

**自动index创建**可以通过在config file中设定`action.auto_create_index`为`false`的方式禁止。

**自动修改mapping**的设定也可以通过在每个index的设置中设置`index.mapper.dynamic`为false来禁止。

**自动index创建**也可以使用黑名单白名单的方式来配置，例：

设置`action.auto_create_index`为`+aaa*,-bbb*,+ccc*,-*`(其中+为允许的规则，-为不允许的规则)

## 版本控制

