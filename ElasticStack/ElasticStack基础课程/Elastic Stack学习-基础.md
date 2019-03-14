# Elastic Stack学习

## 1. Elastic Stack的组成

- Kibana:数据探索与可视化分析
- Elasticsearch：数据存储、查询与分析
- Beats：数据收集与处理
- Logstash：数据收集与处理

## 2. Beats与Logstash

> 主要工作就是数据收集与处理，即ETL:Extract Transform Load

收集数据的数据源可以是多样的 ：

1. 数据文件，如日志、excel
2. 数据库，如mysql、oracle
3. http服务：抓包等
4. 网络数据
5. 自定义扩展 

## 3. ElasticSearch中的常见术语

- 文档（Document）：用户存储在es中的文档数据，是存储的最小单元。相当于关系型数据库中的一行

  > 文档中的数据类型是json Object，由字段(Field)组成

  **常见的数据类型**:

  - 字符串：text,keyword
  - 数值型：long,integer,short,byte,double,float
  - 布尔：boolean
  - 日期：date
  - 二进制：binary
  - 范围类型(针对范围的查询）：integer_range等

  **每个文档都有特定唯一的id标识**:

  - 自行指定
  - es自动生成

  **Document MetaData**文件元数据:

  - _index：文档所在的索引名
  - _type：文档所在的类型名
  - _id：文档唯一id
  - _uid：6.0取消type之后同\_id一样
  - _source：文档最原始的Json数据，如果查询结果是部分显示的，则可以通过\_source获取完整的数据
  - _all（默认禁用）：对文档中所有字段进行查询

  

- 索引（Index）：具有相同段的文档列表组成。相当与关系型数据库中的表。6.0中一个index只能有一个type

  - 索引中存储具有相同结构的文档(Document)：每个索引都有自己的mapping定义，用于定义<em>字段名和类型</em>。
  - 一个集群可以有多个索引

- 节点（node）：一个ElasticSearch的运行实例，是集群的构成单元。
- 集群（Cluster）：多个节点组成的ElasticSearch实例（由一个或多个节点组成，对外提供服务。

## 4. ElasticSearch的RestfulAPI

- 关键点：

  1. URI指定资源
  2. Http Method指明资源操作类型（GET、POST、PUT、DELETE）

- 两种交互方式：

  - Curl命令行

    ```bash
    curl -XPUT 'http://localhost:9200/employee/doc/1' -i -H "Content-Type:application/json" -d 
    '{
        "username":"rockybean",
        "job":"software engineer"
    }'
    ```

  - Kibana DevTools

    ```bash
    PUT /employee/doc/1
    {
        "username":"rockybean",
        "job":"software engineer"
    }
    ```

    