# Elastic Stack学习-Search API

> ES Search API实现对es中存储的数据进行查询分析，endpoint为_search，如下所示：
>
> - GET /_search
>
> - GET /my_index/_search
>
> - GET /my_index1,my_index2/_search
>
> - GET /my_*/_search 
>
> 其查询也有两种形式：
>
> - URI Search
> - Request Body Search
>   - 提供es的完备查询语法Query DSL(Domain Specific Language)

## URI Search

![image-20190217105012484](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190217105012484.png)

### Query String Syntax

- #### term 与 phrase

  - q = alfred way 等效于 alfred OR way(会匹配出所有包含alfred或way的doc)
  - q = "alfred way" 是词语查询，会限定前后顺序（会匹配出同时包含alfred与way而且alfred在way前面的doc）

- #### 泛查询

  - 在所有字段中匹配该term

- #### 指定字段

  - q = name:alfred

- #### Group 分组设定，使用括号指定匹配的规则

  - 指定优先的组
  - (quick OR brown) AND fox
  - status:(active OR pending) title:(full text search)

- #### 布尔操作符

  - AND(&&)，OR(||)，NOT(!)
    - name:(tom NOT lee)->有tom但不能有lee
    - 注意大写，不能小写
  - '+','-'分别对应must和must not
    - name:(tom +lee -alfred)->必须要有lee,可以有tom但不能有alfred
    - name:((lee&&!alfred)||(tom&&lee!alfred))
    - **+在url中会被解析为空格，所以写+的时候要换为%2B**

- #### 范围查询，支持数值和日期

  - 区间写法，闭区间用[]，开区间用{}
    - age:[1 TO 10]意为1<=age<=10
    - age:[1 TO 10}意为1<=age<10
    - age:[1 TO ]意为age>=1
    - age:[* TO 10]意为age<= 10
  - 算数符号写法
    - age:>=1
    - age:(>=1&&\<=10)或者age:(+\>=1+<=10)

- #### 通配符查询

  - ？代表1个字符，*代表0或多个字符
  - 通配符匹配执行效率低，且占用较多内存，不建议使用
  - 无特殊要求，不要将?/*放在最前面

- ![image-20190217212905092](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190217212905092.png)

## Request Body Search

> ![image-20190217213742425](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190217213742425.png)

基于JSON定义的查询语句，主要包含如下两种类型：

### 字段类查询

> 如term(针对词),match（针对全文检索）,range （针对范围）等，只针对某一个字段进行查询

- #### 全文匹配

  > 针对text类型的字段进行全文检索，会对查询语句先进行**分词处理**，如**match,match_phrase**等query类型

  ![image-20190217221226232](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190217221226232.png)

  ![image-20190217221355055](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190217221355055.png)

    ![image-20190217223931401](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190217223931401.png)

  ![image-20190217223945340](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190217223945340.png)

  #### 相关性算分：

  ![image-20190217225219508](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190217225219508.png)

  ##### TF-IDF是lucene的经典模型，其计算公式为:

  $$
  \operatorname { score } ( \mathrm { q } , \mathrm { d } ) = \operatorname { coord } ( \mathrm { q } , \mathrm { d } ) \cdot \text { queryNorm(q) } \cdot \sum _ { t \text { in } q }{\left( \mathrm { tf } ( \mathrm { t } \text { in } \mathrm { d } ) \cdot \mathrm { idf } ( \mathrm { t } ) ^ { 2 } \cdot \text { t.getBoost } () \cdot \operatorname { norm } ( \mathrm { t } , \mathrm { d } ) \right)}
  $$

  > 其中q为查询语句，d为匹配的文档，t为查询语句的分词，norm()是对Field-elngth Norm的计算，getBoost()是对某个分词的加权

  ![image-20190218085734685](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218085734685.png)

  ##### BM25模型（ES主要使用模型）

  ![image-20190218094842551](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218094842551.png)

  ![image-20190218094947347](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218094947347.png)


  - #### 单词匹配

    > 不会对查询语句做分词处理，直接去匹配字段的倒排索引，如**term,terms,range**等query类型

    ![image-20190218111959633](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218111959633.png)

    ![image-20190218112108240](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218112108240.png)

    > slop参数是控制匹配词与关键词有几个词的差异（如slop为1时 java ruby engineer也是可以被匹配到的

    ####  query_string![image-20190218112331389](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218112331389.png)

    ![image-20190218112415274](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218112415274.png)

    ![image-20190218112856240](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218112856240.png)

    #### Range_query:

    > 主要针对数值和日期类型

    ![image-20190218214118512](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218214118512.png)

    ![image-20190218214555821](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218214555821.png)

    ![image-20190218214621023](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218214621023.png)

    ![image-20190218214644156](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218214644156.png)

    ### 复合查询

    > 如bool查询等，包含一个或多个字段类查询或者复合查询语句

    ![image-20190218223508701](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218223508701.png)

    ![image-20190218223656619](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218223656619.png)

    #### Bool Query:

    ![image-20190218224050133](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218224050133.png)

    ![image-20190218224107538](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218224107538.png)

    ![image-20190218224310050](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218224310050.png)

    ![image-20190218224522112](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218224522112.png)

    ![image-20190218224825045](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218224825045.png)



![image-20190218224926133](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218224926133.png)

![image-20190218225115454](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218225115454.png)

![image-20190218225405944](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218225405944.png)



## Count API与Source Filtering API

![image-20190218225617378](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218225617378.png)

![image-20190218225646592](/Users/haohao/Documents/Haohao's-Book/ElasticStack/ElasticStack基础课程/assets/image-20190218225646592.png)

