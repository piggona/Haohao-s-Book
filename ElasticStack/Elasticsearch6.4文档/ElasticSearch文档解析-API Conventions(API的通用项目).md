# ElasticSearch文档解析-API Conventions(API的通用项目)

## 1. 跨索引查询

> 对于在多个索引(index)之间搜索的需求,elasticsearch提供了简单的逗号分隔的跨索引查询的方式。

- 多索引查询：使用逗号分隔指定的索引名称，或在搜索中使用"_all"搜索所有索引(index)

  ```bash
  GET /index_name,index_name2/_search
  {
      "query":{
          "match":{
              "test": "data"
          }
      }
  }
  ```

  

- 排除某个索引：使用减号（"-"）来表示排除某索引

  ```bash
  GET /index_name,-index_name2/_search
  {
      "query":{
          "match":{
              "test": "data"
          }
      }
  }
  ```

## 2. 使用时间名称代替索引(index)名称

- 使用时间范围查询的格式：

  ```bash
  <static_name{date_math_expr{date_format|time_zone}}>
  ```

  | 字段说明       |                             |
  | -------------- | --------------------------- |
  | static_name    | 字段的标识符                |
  | date_math_expr | 时间段的表达式              |
  | date_format    | 时间格式(默认为YYYY.MM.dd.) |
  | time_zone      | 时区(默认为utc)             |

  **在使用时间范围查询时要注意查询语句需要进行URI编码**

  ```bash
  # GET /<logstash-{now/d}>/_search
  
  GET /%3Clogstash-%7Bnow%2Fd%7D%3E/_search
  {
    "query" : {
      "match": {
        "test": "data"
      }
    }
  }
  ```

  | URI编码表 |      |
  | --------- | ---- |
  | <         | %3C  |
  | >         | %3E  |
  | /         | %2F  |
  | {         | %7B  |
  | }         | %7D  |
  | \|        | %7C  |
  | +         | %2B  |
  | :         | %3A  |
  | ,         | %2C  |

  

- 时间格式及时区选项

  假设当今时间为utc时区的2024年三月22日12：00

  | 表达格式                                | 解释                                    | 表示时间            |
  | --------------------------------------- | --------------------------------------- | ------------------- |
  | <logstash-{now/d}>                      | 表示以当前日期(day)为准的天             | logstash-2024.03.22 |
  | <logstash-{now/M}>                      | 表示以当前月份(month)为准的天（第一天） | logstash-2024.03.01 |
  | <logstash-{now/M{YYYY.MM}}>             | 表示月份为准的月份                      | logstash-2024.03    |
  | <logstash-{now/M-1M{YYYY.MM}}>          | 当前月份减1                             | logstash-2024.02    |
  | \<logstash-{now/d{YYYY.MM.dd\|+12:00}}> | 当前时间加12小时                        | logstash-2024.03.23 |

  **例：多重日期的查询**

  ```bash
  # GET /<logstash-{now/d-2d}>,<logstash-{now/d-1d}>,<logstash-{now/d}>/_search
  GET /%3Clogstash-%7Bnow%2Fd-2d%7D%3E%2C%3Clogstash-%7Bnow%2Fd-1d%7D%3E%2C%3Clogstash-%7Bnow%2Fd%7D%3E/_search
  {
    "query" : {
      "match": {
        "test": "data"
      }
    }
  }
  ```

  

## 3. 通用选项(对于elasticsearch中所有的API都有效的选项)

- **Pretty Result**(结果格式化)：

  在查询后面加上$/\_search?pretty=true$这样就会将结果格式化

  在查询后面加上$/\_search?format=yaml$这样就会将结果转化为yaml格式

- **Human readable output**(关闭人性化阅读)

  一般正常显示时，elasticsearch会将一些单位转化为易于用户阅读的单位（`"exists_time": "1h"` or `"size": "1kb"` ）

  当在查询后面加上$/?human=false$时，就会关闭这种设置，返回的是计算机得到的原始值（`"exists_time_in_millis": 3600000` or `"size_in_bytes": 1024` ）

- **Date Math**（时间计算）

  时间计算方式：

  - +1h 加一个小时
  - -1d 减一天
  - /d    离当前表达式的时间最近的一天（优先选择之前的时间）

  时间计量单位：

  | 计量单位 |      |
  | -------- | ---- |
  | y        | 年   |
  | M        | 月   |
  | w        | 星期 |
  | d        | 日   |
  | h        | 小时 |
  | H        | 小时 |
  | m        | 分钟 |
  | s        | 秒   |

  

- **Response Filtering**(返回结果选择性展示)

  通过在查询后面加上$/\_\{field\}?filter\_path=$字段,这样就可以将返回结果进行**选择字段**的展示。

  **查询1：**

  ```bash
  GET /_search?q=elasticsearch&filter_path=took,hits.hits._id,hits.hits._score
  ```

  **结果1：**

  ```bash
  {
    "took" : 3,
    "hits" : {
      "hits" : [
        {
          "_id" : "0",
          "_score" : 1.6375021
        }
      ]
    }
  }
  ```

  > filter_path字段中就是要显示的字段名称

  **查询2：**

  ```bash
  GET /_cluster/state?filter_path=metadata.indices.*.stat*
  ```

  > 查询表达式中的$ * $表示任意匹配的字符

  **结果2**:

  ```json
  {
    "metadata" : {
      "indices" : {
        "twitter": {"state": "open"}
      }
    }
  }
  ```

  **查询3**：

  ```bash
  GET /_cluster/state?filter_path=routing_table.indices.**.state
  ```

  > 查询表达式中的$ ** $表示任意匹配的**路径**

  **结果3**：

  ```json
  {
    "routing_table": {
      "indices": {
        "twitter": {
          "shards": {
            "0": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
            "1": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
            "2": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
            "3": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
            "4": [{"state": "STARTED"}, {"state": "UNASSIGNED"}]
          }
        }
      }
    }
  }
  ```

  **查询4**：

  ```bash
  GET /_cluster/state?filter_path=metadata.indices.*.state,-metadata.indices.logstash-*
  ```

  > 查询表达式中可以通过$ - $来选择不显示的选项,
  >
  > 当同时有排除语句与选择显示语句时，先进行排除操作，之后再在剩下的结果中进行选择显示

  **结果4**：

  ```json
  {
    "metadata" : {
      "indices" : {
        "index-1" : {"state" : "open"},
        "index-2" : {"state" : "open"},
        "index-3" : {"state" : "open"}
      }
    }
  }
  ```

  **查询5**：

  ```bash
  POST /library/book?refresh
  {"title": "Book #1", "rating": 200.1}
  POST /library/book?refresh
  {"title": "Book #2", "rating": 1.7}
  POST /library/book?refresh
  {"title": "Book #3", "rating": 0.1}
  GET /_search?filter_path=hits.hits._source&_source=title&sort=rating:desc
  ```

  查询时，_source会显示查询返回的所有数据，如果需要对__source中的数据进行选择显示，则需要进行如上操作

  **结果5**：

  ```json
  {
    "hits" : {
      "hits" : [ {
        "_source":{"title":"Book #1"}
      }, {
        "_source":{"title":"Book #2"}
      }, {
        "_source":{"title":"Book #3"}
      } ]
    }
  }
  ```

  

- **Flat Settings**(扁平化显示)

  当开启时：

  ```bash
  GET twitter/_settings?flat_settings=true
  ```

  ```json
  {
    "twitter" : {
      "settings": {
        "index.number_of_replicas": "1",
        "index.number_of_shards": "1",
        "index.creation_date": "1474389951325",
        "index.uuid": "n6gzFZTgS664GUfx0Xrpjw",
        "index.version.created": ...,
        "index.provided_name" : "twitter"
      }
    }
  }
  ```

  当关闭时：

  ```bash
  GET twitter/_settings?flat_settings=false
  ```

  ```json
  {
    "twitter" : {
      "settings" : {
        "index" : {
          "number_of_replicas": "1",
          "number_of_shards": "1",
          "creation_date": "1474389951325",
          "uuid": "n6gzFZTgS664GUfx0Xrpjw",
          "version": {
            "created": ...
          },
          "provided_name" : "twitter"
        }
      }
    }
  }
  ```

  

- **Fuzziness**(对可能出现的拼写错误的更正查询)

  > 有时候用户输入的搜索语句会有拼写错误

  在搜索语句后面加入$ ?fuzziness =度量值$的方式进行==更正查询==

  更正尺度：

  - 可以设置为0，1，2等精确值
  - 可以使用auto语句（AUTO:[low],[high]）
    - 0..2表示必须精确匹配
    - 3..5表示可以错误一个edit左右(edit度量见levenshtein distance)
    - \>5表示可以错误两个edit

- **Enabling stack traces**(开启堆栈返回追踪)

  > 用于更清楚的调试查询的错误

  ```
  POST /twitter/_search?size=surprise_me
  ```

  ```json
  {
    "error" : {
      "root_cause" : [
        {
          "type" : "illegal_argument_exception",
          "reason" : "Failed to parse int parameter [size] with value [surprise_me]"
        }
      ],
      "type" : "illegal_argument_exception",
      "reason" : "Failed to parse int parameter [size] with value [surprise_me]",
      "caused_by" : {
        "type" : "number_format_exception",
        "reason" : "For input string: \"surprise_me\""
      }
    },
    "status" : 400
  }
  ```

  ```bash
  POST /twitter/_search?size=surprise_me&error_trace=true
  ```

  ```
  {
    "error": {
      "root_cause": [
        {
          "type": "illegal_argument_exception",
          "reason": "Failed to parse int parameter [size] with value [surprise_me]",
          "stack_trace": "Failed to parse int parameter [size] with value [surprise_me]]; nested: IllegalArgumentException..."
        }
      ],
      "type": "illegal_argument_exception",
      "reason": "Failed to parse int parameter [size] with value [surprise_me]",
      "stack_trace": "java.lang.IllegalArgumentException: Failed to parse int parameter [size] with value [surprise_me]\n    at org.elasticsearch.rest.RestRequest.paramAsInt(RestRequest.java:175)...",
      "caused_by": {
        "type": "number_format_exception",
        "reason": "For input string: \"surprise_me\"",
        "stack_trace": "java.lang.NumberFormatException: For input string: \"surprise_me\"\n    at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)..."
      }
    },
    "status": 400
  }
  ```

  ## 4. URL-based access control(简单的防止索引覆盖设置)

  有时候需要设置用户不能覆盖已经存在的索引，此时可以通过在elasticsearch.yml文件中加上如下语句的方式进行配置：

  ```yaml
  rest.action.multi.allow_explicit_index: false
  ```

  