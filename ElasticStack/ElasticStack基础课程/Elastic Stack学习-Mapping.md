# Elastic Stack学习-Mapping

> Mapping的作用：定义Index下的字段名（Field Name），定义字段的类型，定义倒排索引相关的配置（比如是否索引，记录position等）

## 获取索引的Mapping

![image-20190123231040961](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190123231040961.png)

## 自定义Mapping

自定义Mapping通过PUT Api来实现，主要格式：

```
PUT index名{
    "mappings":{ // 说明是在自定义mapping
        "doc":{ // mapping的名字
            // mapping整体配置
            "properties":{ // mapping元素
                "title" :{ // 字段名
                    "type":“text",
                    // mapping字段配置
                },
                "name" :{
                    "type":"keyword"
                },
                "age" :{
                    "type":"integer"
                }
            }
        }
    }
}
```



![image-20190123231205369](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190123231205369.png)

- Mapping中的字段类型一旦设定后，禁止直接修改（Lucene实现的倒排索引生成后不允许修改）
- 重新建立新的索引，然后做reindex操作

### Mapping 整体配置选项

- #### dynamic

  ![image-20190123233203167](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190123233203167.png)

- 

### Mapping字段内部配置

- #### copy_to:将字段复制到目标字段

  ![image-20190124230458489](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190124230458489.png)

- #### index:控制当前字段是否索引（可不可以被查到）

  ![image-20190124230722862](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190124230722862.png)

  > 此配置主要用于开发者不想某些字段作为用户搜索的字段，如一些隐私字段。并且可以节省倒排索引的空间

- #### index_options:控制倒排索引记录的内容

  - docs：记录doc id
  - freqs：记录doc id、term frequencies
  - positions：记录doc id、term frequencies、term position
  - offsets：记录doc id、term frequencies、term position、character offsets

- #### null_value:控制遇到null值时的处理策略

  ![image-20190124231318181](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190124231318181.png)

- #### Mapping字段的type

  - 核心数据类型

    ![image-20190124231449397](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190124231449397.png)

  - 复杂数据类型

    - 数组类型array
    - 对象类型object（有子字段）
    - 嵌套类型nested object（文档与副文档都是独立存在的）
    - 地理位置类型

  - 专用类型

    ![image-20190124231723169](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190124231723169.png)

  - 多字段特性（multi_fields）

    ![image-20190124231926792](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190124231926792.png)

  - 

## Dynamic Mapping(自动生成mapping)

> elasticsearch依赖JSON文档的字段类型来实现自动识别字段类型

识别规则如下：

![image-20190214072849371](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190214072849371.png)

### Dynamic Mapping中的日期格式设置

![image-20190214073150438](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190214073150438.png)

![image-20190214073349869](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190214073349869.png)

### Dynamic Mapping中的数字自动识别

![image-20190214073535773](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190214073535773.png)

## Dynamic Template

![image-20190214074141024](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190214074141024.png)

![image-20190214075229131](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190214075229131.png)

![image-20190214075427882](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190214075427882.png)

![image-20190214075633592](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190214075633592.png)



