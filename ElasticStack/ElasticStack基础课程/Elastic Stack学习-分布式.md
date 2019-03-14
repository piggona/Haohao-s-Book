# Elastic Stack学习-分布式

![image-20190219080702169](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190219080702169.png)

## 分布式维护插件-Cerebro

### 安装配置：

在github的cerebro repository中下载release版本，到elasticsearch的主机中解压（因为cerebro的默认配置是连接本地的elasticsearch实例），之后运行`bin/cerebro`就可以运行了（监听来自任何host的9000端口）

![image-20190219082109849](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190219082109849.png)

> more中的功能解析：
>
> - create index:新建一个index
> - cluster settins：设置集群
> - analysis：测试分词
> - cat apis：常见的功能api(查看类)

## 构建集群

> 运行如下命令可以启动一个es节点实例：
>
> - `bin/elasticsearch -E cluster.name=my_cluster -E path.data=my_cluster_node1 -E node.name=node1 -E http.port=5200 -d`

![image-20190219183531292](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190219183531292.png)

![image-20190219183548977](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190219183548977.png)

![image-20190219183614192](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190219183614192.png)

![image-20190219183807010](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190219183807010.png)

![image-20190219183833360](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190219183833360.png)

## 新增节点

```bash
bin/elasticsearch -E cluster.name=my_cluster -E path.data=my_cluster_node2 -E node.name=node2 -E http.port=5300 -d
```

```bash
bin/elasticsearch -E cluster.name=my_cluster -E path.data=my_cluster_node3 -E node.name=node3 -E http.port=5400 -d
```



## 提高系统可用性

### 服务可用性

> 2个节点的情况下，允许其中一个节点停止服务

#### 分片：

![image-20190219190304979](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190219190304979.png)

![image-20190219190401349](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190219190401349.png)

- 加入节点是否能提高test_index的数据容量？

  不能。因为只有3个分片，已经分布在三台节点上，新增节点也不会在新的节点上分配分片。（**所以要在最初就安排好分片的数量，默认为5**）

  ![image-20190220220221096](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190220220221096.png)

- 增加副本数是否能提高test_index的读取吞吐量？

  不能，因为增加副本数也是将数据分布在这三个节点上，利用了同样的三个节点的资源。如果要增加吞吐量，还要增加节点（资源）

  ![image-20190220220605776](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190220220605776.png)

  ![image-20190220220747024](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190220220747024.png)


### 数据可用性

> - 引入副本(Replication)解决
> - 每个节点上都有完备的数据

![image-20190220221720710](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190220221720710.png)

### 故障转移：

![image-20190220221951238](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190220221951238.png)

最初，三个节点，三个分片，一个replica的系统为green状态。

关闭一个节点，之后就变成了yellow状态：

![image-20190220222603114](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190220222603114.png)

此时，系统发现丢失了主分片0，replica2。而且断开的是主节点，所以再次进行主分片选举。

故障转移之后，又恢复为了green状态：

![image-20190220222817500](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190220222817500.png)

## 文档分布式存储

> Document1是如何存储到分片P1的？选择P1的依据是什么？
>
> - 需要文档到分片的映射算法
>
> 目的：
>
> - 使得文档均匀分布在所有分片上，以充分利用资源
>
> 算法：
>
> - 随机选择或者round-robin算法(是不可取的，因为维护文档到分片的映射关系，成本巨大)
> - 需要一种可以根据文档值实时计算对应分片的算法

Elasticsearch使用的算法：
$$
shard = hash(routing)\%number\_of\_primary\_shard
$$

> routing是一个关键参数，默认是文档id，也可以自行指定

![image-20190221080729969](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190221080729969.png)

![image-20190221080903264](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190221080903264.png)

![image-20190221081037006](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190221081037006.png)

## 脑裂问题

![image-20190221081559341](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190221081559341.png)

脑裂问题通过限制选举master节点的情况的方式来进行解决。

![image-20190221081641993](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190221081641993.png)

## Shard

### 倒排索引：

![image-20190221082010513](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190221082010513.png)

![image-20190221082127002](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190221082127002.png)

![image-20190221082308056](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190221082308056.png)

![image-20190221082426187](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190221082426187.png)

![image-20190221083133999](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190221083133999.png)

![image-20190221083627060](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190221083627060.png)

![image-20190222084254509](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190222084254509.png)

> 重要的三大操作：
>
> 1. refresh:：将buffer中新增文档的操作执行到内存中生成segment（每1sec）
> 2. flush：将操作每5秒写入磁盘备份，防止refresh失败
> 3. commit Point更新：将认知的shard中的倒排索引文件信息更新

![image-20190222090341417](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190222090341417.png)

![image-20190222090806234](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190222090806234.png)

![image-20190222090905222](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190222090905222.png)

> 所以es中删除不是物理上的删除，只是一个过滤的机制

![image-20190222091037590](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190222091037590.png)

![image-20190222091056988](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190222091056988.png)

