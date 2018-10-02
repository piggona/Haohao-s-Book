# Elasticsearch.js API文档1：API通用共识

## 1. 通用参数变量：

默认情况下，下面的所有参数可以被所有的API方法接收：

| 通用变量   |                                                              |
| ---------- | ------------------------------------------------------------ |
| method     | String—当前请求所使用的HTTP方法，所有的API函数各有各的默认方法 |
| body       | String，Anything --- 通过请求发送过去的内容，一般可以是一个简单的查询字符，也可以是序列化的JSON |
| ignore     | Number,Number[] — 返回的HTTP状态码中不应该被认为是error的    |
| filterPath | String\|String[] --- 用于部分显示查询到的数据                |

## 2. 请求时可以重写设置的参数

- requestTimeout(请求超时时间)：

- maxRetries(最多的重试次数)：

  **可以在client初始化设置时找到设置的详情。**

## 3. Callback与Promise函数

当任意的API方法使用callback函数时，函数的格式为：

```javascript
(err,response,status)
```

如果没有使用callback，就会默认返回promise方式的返回。