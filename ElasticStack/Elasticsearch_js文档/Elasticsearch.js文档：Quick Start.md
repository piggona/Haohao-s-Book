# Elasticsearch.js文档：Quick Start

> Elasticsearch.js是elasticsearch官方的javascript端连接elasticsearch服务的API工具。

## 特性

- One-to-one mapping with REST API(与elasticsearch的REST API一一映射对应)
- Configurable,automatic discovery of cluster nodes（可以进行自定义配置，可以自动发现并连接集群的nodes）
- Persistent,Keep-Alive connections（对elasticsearch服务持续的keep-alive连接)
- Intelligent hadling of <font color=#FF0033>node/connection failure</font>（对节点/连接错误进行智能处理）
- Load balancing(with plug-able selection strategy) across all available nodes（在所有的可用node之间进行负载均衡->使用插件化的优化选择策略）
- 可以在nodejs端与浏览器端完美的运行([browserify](https://github.com/substack/node-browserify))
- Generalized,plug-able,and highly configurable architecture(通用化，多插件可用并且高度可配置的应用架构)

##### 安装：

```bash
npm install --save elasticsearch
```

## 基础使用(Quick Start)

- ### 建立基础的Client实例(Creating a client)

  通过建立一个elasticsearch Client实例来开始使用`elasticsearch.js`，Client的构造函数接收对于连接elasticseach服务的[配置](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/configuration.html)

  ##### 例：

  ```javascript
  var elasticsearch = require('elasticsearch');
  var client = new elasticsearch.Client({
    host: 'localhost:9200',
    log: 'trace'
  });
  ```

- ### 简单使用配置

  所有client实例的方法一般都支持两种变量:

  - `params` - [可以配置的object/hash类型的变量](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-conventions.html)
  - `callback` - 接收查询/操作结果的callback函数，如果不配置callback函数，也可以使用Promise对象来接收返回。[更多信息](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-conventions.html#api-conventions-cb)

  #### 查询/操作之前要检查是否能连接到集群(elasticsearch 服务)

  > 发送一个HEAD request，并设置等待超时时间

  ```javascript
  client.ping({
      requestTimeout: 30000,//超时时间30s
  }, function (error){
      if(error){
          console.log('elasticsearch cluster is down!');
      } else{
          console.log('All is well');
      }
  })
  ```

  #### 使用Promise对象

  ```javascript
  client.search({
    q: 'pants'
  }).then(function (body) {
    var hits = body.hits.hits;
  }, function (error) {
    console.trace(error.message);
  });
  ```

  #### 配置忽略404错误并返回一个错误信息

  > client不应将404视为中断程序的错误

  ```javascript
  client.indicies.delete({
      index: 'test_index',
      ignore: [404]
  }).then(function (body){
      //当出现404错误时，在这里返回错误信息
      console.log('index was deleted or never existed');
  },function (error){
      //未知错误
  })
  ```

- ### 搜索方式示例(Elasticsearch Query DSL)

