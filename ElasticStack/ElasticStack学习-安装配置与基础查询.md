# ElasticStack学习-安装配置与基础操作

## 1. ElasticSearch与Kibana的安装及配置

## 2. ElasticSearch的基础操作语句

>首先强调一下Restful Api对资源的操作
>
>PUT：对资源进行更新
>
>POST：对资源进行新建
>
>GET：获取资源
>
>DELETE：删除资源 

1. 创建/更新文档(index)命令:

   - 简单创建/更新文档：

     ```bash
     PUT /index_name/type_name/id
     {
     "username": "alfred",
     "age": 1
     }
     ```

     > 其中type_name一般设为doc
     >
     > id为自定义的Index id，如果不设定，系统会自动分配一个id

     这样就会创建一个index

     ```bash
     POST /index_name/doc
     {
         "username": "alfred",
         "age":1
     }
     ```

     自动分配id需要使用POST方法

   - 批量操作文档：

     ```bash
     POST _bulk
     {"index":{
         "_index":"index_name",
         "_type":"doc",
         "_id": "3"
     }}
     {"username":"alfred","age":10}
     {"delete":{
         "_index":"index_name",
         "_type":"doc",
         "_id":"1"
     }}
     {"update":{
         "_index":"index_name",
         "_type":"doc",
         "_id":"1"
     }}
     {"doc":{"age":"20"}}
     ```

     > 批量操作的规则主要是
     >
     > 1. 先指定操作方式（action_type)与指定要操作的文档。
     > 2. 之后是要操作的内容（delete等不需要操作具体内容的操作不需要这一项

   **action_type:**

   - index：查询，若没有则添加这一项
   - update：更新某index的某一项的值
   - create：创建
   - delete：某index的某一项

   使用_bulk方式批量操作时，如果使用curl命令则需要将请求的头Content-Type设置为application/json

   ```bash
   curl -s -H "Content-Type:application/x-ndjson" -XPOST localhost:9200/_bulk --data-binary "@data.json"
   ```

   

2. 查询文档命令：

   - 简单查询（id查询）
   - 条件查询

3. 删除文档命令