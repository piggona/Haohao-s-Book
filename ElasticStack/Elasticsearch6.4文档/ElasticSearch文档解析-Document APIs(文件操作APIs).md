# ElasticSearch文档解析-Document APIs(文件操作APIs)

## 1. Elasticsearch的文件读写模型（主体为*data replication model*-数据备份模型）

> 本小节主要介绍elasticsearch的data replication model及根据其原理讨论在进行读写操作时的操作技巧建议

1. ### data replication model:

   > 在elasticsearch中，每个index(索引)被分为若干shard，而每个shard都被复制为若干份。

   这些shard的备份被称为*replication group*，ElasticSearch需要保证这些replica shard在文档的读写中保持同步。

   **保持shard同步并提供读写功能的模型就叫做data replication model**

   其中data replication model是基于微软研究的技术[primary-backup model](https://www.microsoft.com/en-us/research/publication/pacifica-replication-in-log-based-distributed-storage-systems/)

   > primary-backup model:
   >
   > 选择一个replication group中的shard作为primary shard，其它shard称作replica shards。primary shard作为所有索引操作(indexing operations)的统一入口，它负责实现这些操作并保证操作正确有效。当一个索引操作被primary shard认为有效，它就会将这个操作执行到其它的replica shards中。

   

2. ### 基础写模型(Basic write model)：

   - #### 正常写入流程

     每一个对索引(index)进行的操作都经过 [routing](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-routing)的方法交给一个**replication group**来进行处理（一般使用document ID来确定分配的replication group)

     当确定好分配的replication group之后，这个操作将会在内部发往group的当前(current)**主分片(primary shard)**，这个主分片的任务是==在本地实施这个操作并将这个操作给备份分片(replicas)==。

     因为分片(replicas)可能出现离线的情况，主分片(primary)不需要将操作发布到所有replica才算完成。Elasticsearch维护着一个**需要接收操作的shard**的表，这个表叫做**“在同步”备份(in-sync copies)**存储在主node中。

     in-sync copies表中存储有**功能完整的shard**的列表，primary shard需要将操作信息发布给这个表上的所有的shard。

     #### primary shard中的写入操作流程：

     1. 检查操作结构及内容的合法性（主要是**传入结构的合法性检查**）
     2. 在本地执行这个操作，包括indexing与delete操作，在执行过程中也会**验证操作数据的合法性**。（indexing操作：主要是lucene中的添加/更新操作+分词过程）
     3. 并行的将操作发给in-sync copies表中的所有replica
     4. 当所有的replicas已经进行完操作并返回信息给主分片(primary)，主分片就会确认操作完成并告知用户

   - #### 失败处理

     #### 1. primary shard fails:

     当出现primary shard的错误，负责管理primary的node就会将信息发给master，master会在一分钟内(默认情况)将指定一个新的primary shard，之后指令将发往新的primary shard上。

     > master会自动的检测节点(nodes)的状态，自动将一个不符合要求的primary shard降级（主要发生于该shard由于网络原因与其它节点隔离),主动降级(proactively demote)的[算法细节](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-replication.html#demoted-primary)

     #### 2. replica fails:

     当因为replica自身的问题或网络问题无法成功执行操作，primary shard会把==**“将该replica移除出in-sync replica set”**==的请求发给master。当得到允许的指令后，primary会进行删除in-sync replica set中相应项目的动作。

     当in-sync replica set中删除fail项目后，elasticsearch集群将进行shard分配的调整（因为少了一个replica,需要再复制一份replica根据算法分配到另外的健康node上)

     > 当向replica推送指令时，primary shard也在确认自己作为primary shard的身份，当向replica推送指令返回失败时，primary shard就会意识到身份的转变(已经被proactively demote),将自己route到新的primary shard上

3. ### 基础读模型(Basic read model):

   > 在elasticsearch中，读操作可能非常轻量（通过id搜索）也有高级操作（聚合搜索），而elasticsearch primary backup算法的优势也就在于它的每一个shard都是一个完整的个体，可以完成整个请求，相结合又可以形成高可用性，高性能的elasticsearch整体。

   #### 1. 读操作流程（Basic flow)

   - 确定需要发送请求到哪些shard中。因为许多搜索请求需要跨越多个索引，需要从多个shard中读取结果，每个shard中搜索到的都是结果的子集。生成shard level read request(对应于各个shard的读请求)
   - 从replication group中选择一个shard，可以是primary也可以是replica。默认是轮换的方式。
   - 将每个层次（shard level read request）读请求发送到对应shard中
   - 将结果整合并返回结果

   #### 2. 失败处理

   失败时的处理一般是将shard level read request发送给另一个可用的shard

4. ### 一些启示

   - 只有当出现错误时，一个读请求会使用用一个replication group的两个shard
   - 因为primary首先在本地进行搜索/创建，可能出现还没有得到确认，操作却已经发生的情况
   - 只要保证有两个数据副本，就可以实现elasticsearch的高可用性。

5. ### 常见的失败原因

   - 低速的shard会影响到整体的查询/修改的速度
   - 脏读：因为种种原因被孤立的primary有可能会错误的进行没有被认可的写操作而使数据库出错（因为旧primary只有向replica发送消息的时候才知道自己被孤立了），在读的时候也会出现错误的数据(脏读)。Elasticsearch为了解决这种问题：1. 每秒primary都会与master进行ping操作以确认身份及状态(默认)。2. primary会拒绝进行master不知道的indexing操作。