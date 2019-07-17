---
layout:     post
title:      Elasticsearch(二)
subtitle:   Elasticsearch
date:       2019-07-17
author:     Mr.C
header-img: img/es2.jpg
catalog: true
tags:
    - Elasticsearch

---


## Elasticsearch的概念

> 分片、节点的概念更面向于运维人员 <br> 
索引、类型、文档、字段的概念更面向于发开人员

| 关系型数据库 |  Elasticsearch |
|--------------|----------------|
|数据库Database|索引Index，支持全文检索|
|表Table|类型Type|
|数据行Row|文档Document，但不需要固定结构，不同文档可以具有不同字段集合|
|数据列Column|字段Field|

#### 文档(Document)

- Elasticsearch是面向文档的，文档是所有可搜索数据的最小单位
- 文档会被序列化程 JSON 格式，保持在 Elasticsearch 中
    - JSON 对象由字段组成
        - 字段类型可以指定或者Elasticsearch自动推算
        - 支持数组/支持嵌套
    - 每个字段都有对应的字段类型(字符串/数值/布尔/日期/二进制/范围类型)
    
- 每个文档都有一个 Unique ID
- 元数据
    
    ~~~json
    {
        "_index": "name",
        "_type": "_doc",
        "_id": "1",
        "_score": 13.14520,
        "_source": {
            "year": 1994,
            "@version": "1",
            "genre": [
                "Adventure",
                "Comedy"
            ],
            "id": "1"
            "title": "Mr_c"
        }
    }
    ~~~

    > 用于标注文档的相关信息
    
    - _index：文档所属的索引名
    - _type：文档所属的类型名
    - _id：文档的唯一id
    - _source：文档的原始Json数据
    - _all：整合所有字段内容到该字段，Elasticsearch 7.X 之后被废除
    - _version：文档的版本信息
    - _score：相关性算分

#### 索引(Index)

- 索引是文档的容器，是一类文档的结合
    
    ~~~json
    {
        "name": {
            "settings": {
                "index": {
                    "creation_date": "1563367271",
                "number_of_shards": "2",
                "number_of_replicas": "0",
                "version": {
                    "created": "1"
                    },
                "provided_name": "name"
                }
            }
        }
    }

    ~~~

    - 每个索引都有自己的Mapping定义，用于定义包含的文档的字段名和字段类型
    - 索引中的数据分散在分片(Shard)上

- 索引上的 Mapping 与 Settings
    - Mapping：定义文档字段的类型
    - Setting：定义不同的数据分布

- 索引(index)的动名词
    - 名词：一个Elasticsearch集群中，可以创建很多个不同的索引
    - 动词：保持一个文档到Elasticsearch的过程也叫索引(indexing)
        - ES中，创建一个倒排索引的过程

#### 类型(Type)

- 在索引中，可以定义一个或多个类型。类型是索引的逻辑类别/分区
    - 在7.0之前，一个Index可以设置多个Types
    - 6.0开始，Type已经被Deprecated。7.0开始，一个索引只能创建一个Type-"_doc"

## 分布式系统的可用性与拓展性

- 高可用性
    - 服务可用性，允许有节点停止服务
    - 数据可用性，部分节点丢失，不会丢失数据

- 可扩展性
    - 请求量提升/数据的不断增长(将数据分布到所有节点上)

#### 分布式特性

- Elasticsearch的分布式架构的好处
    - 存储的水平扩容
    - 提高系统的可用性，部分节点停止服务，整个集群的服务不受影响

- Elasticsearch的分布式架构
    - 不同的集群通过不同的名字来分区，默认名字"elasticsearch"
    - 通过配置文件修改，或者在命令中 -E cluster.name=geekime 进行设定
    - 一个集群可以有一个或者多个节点

#### 节点(Node)

> 一个运行中的 Elasticsearch 实例称为一个 节点，而集群是由一个或者多个拥有相同 cluster.name 配置的节点组成， 它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。

- 每个节点启动后，默认就是一个Master eligible 节点
    - 可以设置 node.master:false 禁止

- Master-eligible 节点可以参加选主流程，成为 Master 节点

- 当第一节点启动的时候，它会将自己选举程Master节点
    - 集群状态(Cluster State)，维护了一个集群中，必要信息
        - 所有的节点信息
        - 所有的索引和其相关的Mapping 与 Setting 信息
        - 分片的路由信息
    - 任意节点都能修改信息会导致数据的不一致

- Data Node & Coordinating Node
    - Data Node —— 可以保持数据的节点，负责保存分片数据，在数据扩展上起到了至关重要
    - Coordinating Node —— 负责接受Client的请求，将请求分发到合适的节点，最后扒结果汇聚到一起并返回

- Hot & Warm Node
    - 不同硬件配置的 Data Node，用来实现 Hot & Warm 架构，降低集群部署的成本

> 配置节点类型
>> - 开发环境中一个节点可以承担多种角色 <br> 
>> - 生产环境中，应该设置单一的角色的节点

|节点类型|配置参数|默认值|
|--------|--------|------|
|master eligible|node.master|true|
|data|node.data|true|
|ingest|node.ingest|true|
|coordinationg only|无|每个节点默认都是coordinating节点，设置其他类型全部为false|
|machine learning|node.ml|true(需 enable x-pack)|

#### 分片(Shard)

Elasticsearch 是利用分片将数据分发到集群内各处的。分片是数据的容器，文档保存在分片内，分片又被分配到集群内的各个节点里。 当你的集群规模扩大或者缩小时， Elasticsearch 会自动的在各节点中迁移分片，使得数据仍然均匀分布在集群里。

- 主分片（primary shard），用以解决水平扩展的问题，通过主分片，可以将数据分布到集群内所有节点之上
    - 一个分片是一个运行 Lucene的实例
    - 主分片数在创建时指定，后续不许修改，除非Reindex
    - 索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量。

- 复制分片（副分片 replica shard)，用以解决数据高可用
    - 一个副本分片只是一个主分片的拷贝。 副本分片作为硬件故障时保护数据不丢失的冗余备份，并为搜索和返回文档等读操作提供服务。
    - 副本分片数，可以动态调整
    - 增加副本数，还可以在一定程度上提高服务的可用性(读取的吞吐)

> 分片的设定

- 对于生产环境中分片的设定，需要提前做好容量规划
    - 分片数设置过小
        - 导致后续无法增加节点实现水平扩展
        - 单个分片的数据量太大，导致数据重新分配耗时
    
    - 分片设置过大，7.0 开始，默认主分片设置成1，解决了over-sharding的问题
        - 影响搜索结果的相关性打分，影响统计结果的准确性
        - 单个节点上过多的分片，会导致资源浪费，同时也会影响性能

~~~json
GET _cluster/health  # 查看集群的健康状况

·Gree —— 主分片与副本都正常
·Yellow —— 主分片全部正常分配，有副本分片未能正常分配
·Red —— 有主分片未能分配
~~~

#### 映射(Mapping)

> 映射(Mapping)相当于数据表的结构，ElasticSearch中的映射（Mapping）用来定义一个文档，可以定义所包含的字段以及字段的类型、分词器及属性等等
>> 动态映射：在关系数据库中，需要事先创建数据库，然后在该数据库实例下创建数据表，然后才能在该数据表中插入数据。而ElasticSearch中不需要事先定义映射（Mapping），文档写入ElasticSearch时，会根据文档字段自动识别类型，这种机制称之为动态映射 <br> 
静态映射：在ElasticSearch中也可以事先定义好映射，包含文档的各个字段及其类型等，这种方式称之为静态映射

#### 数据类型

- **字符串类型**
    
    - **string类型**：仅旧版使用，ES 5.X后不再支持
    - **text类型**：当一个字段是要被全文检索，test类型的字段不用于排序、很少用户聚合
    - **keyword类型**：适用与索引结构化的字段，如果字段需要进行过滤、排序、聚合，keyword类型的字段可以精确搜索到

- **整数类型**
    - **byte类型**：
    - **short类型**
    - **integer**
    - **long类型**

- **浮点数**
    - **float类型**：32位单精度
    - **double类型**：64位单精度

- **布尔型**
    - **boolean类型**：true和false

- **日期类型**
    - **date类型**

- **二进制类型**
    - **binary类型**：进制字段是用base64来表示索引中存储的二进制数据，该数据类型只存储不索引，二进制类型只支持index_name属性

- **数组类型**
    - **字符数组**：[ “one”, “two” ]
    - **整数数组**：productid:[ 1, 2 ]
    - **对象数组**：“user”:[ { “name”: “Mr_c”, “age”: 18 }

- **object类型**
    - **JSON对象**，文档会包含嵌套对象

- **IP类型**
    - **ip类型**的字段用于存储IPv4和IPv6的地址

#### 类型映射

- enabled：仅存储、不做搜索和聚合分析
- index：是否构建倒排索引（即是否分词，设置false，字段将不会被索引）
- index_option：存储倒排索引的哪些信息
- norms：是否归一化相关参数、如果字段仅用于过滤和聚合分析、可关闭
分词字段默认配置，不分词字段：默认{“enable”: false}，存储长度因子和索引时boost，建议对需要参加评分字段使用，会额外增加内存消耗
- doc_value：是否开启doc_value，用户聚合和排序分析
- fielddata：是否为text类型启动fielddata，实现排序和聚合分析
- store：是否单独设置此字段的是否存储而从_source字段中分离，只能搜索，不能获取值
- coerce：是否开启自动数据类型转换功能，比如：字符串转数字，浮点转整型
- multifields：灵活使用多字段解决多样的业务需求
- dynamic：控制mapping的自动更新
- data_detection：是否自动识别日期类型
- analyzer：指定分词器，默认分词器为standard analyzer
- boost：字段级别的分数加权，默认值是1.0
- fields：可以对一个字段提供多种索引模式，同一个字段的值，一个分词，一个不分词
- ignore_above：超过100个字符的文本，将会被忽略，不被索引
- include_in_all：设置是否此字段包含在_all字段中，默认时true，除非index设置成no
- null_value：设置一些缺失字段的初始化，只有string可以使用，分词字段的null值也会被分词
- position_increament_gap：影响距离查询或近似查询，可以设置在多值字段的数据上或分词字段上，查询时可以指定slop间隔，默认值时100
- search_analyzer：设置搜索时的分词器，默认跟analyzer是一致的，比如index时用standard+ngram，搜索时用standard用来完成自动提示功能
- similarity：默认时TF/IDF算法，指定一个字段评分策略，仅仅对字符串型和分词类型有效
- trem_vector：默认不存储向量信息，支持参数yes（term存储），with_positions（term+位置），with_offsets（term+偏移量），with_positions_offsets（term+位置+偏移量）对快速高亮fast vector highlighter能提升性能，但开启又会加大索引体积，不适合大数据量用

## ES的RESTful api

> HTTP动词+端点+请求主体JSON”的格式

- URL的增删改查
    
    > 增删改查：GET、POST、PUT、DELETE请求方式等

- URL的格式
    
    ~~~python
    http://IP:9200/<index>/<type>/[<id>]
    ~~~

    - 其中index、type是必须提供的。
    - id是可选的，不提供es会自动生成。
    - index、type将信息进行分层，利于管理。
    - index可以理解为数据库；type理解为数据表；id相当于数据库表中记录的主键，是唯一的。

- **pretty参数**
    
    > 通过 ?pretty 可以将结果以json格式化的方式输出

- **q参数**

    > 查询条件（q）参数用于指定返回的文档必须匹配的查询条件
    
- **sort参数**
    
    > 排序（sort）参数，用于对结果进行排序，使ElasticSearch按照指定的字段对结果进行排序

- **default_operator参数**
    
    > 查询条件之间的逻辑关系由默认操作符（default_operator）参数指定，默认值是or，该属性可以设置为and 或 or

#### 查询端点

- _search端点
    
    > 允许执行收搜查询，返回查询结果

- _query端点
    
    > 只用于将查询的结果删除

#### 分析端点

- _analyze
    
    > 用于对查询参数进行分析，并返回分析的结果

#### 计数端点

- _count
    
    > 执行查询，获取满足查询条件的文档数量

#### 解释端点

- _explain
    
    > 用于验证指定的文档是否满足查询条件，格式是index/type/_id/_explain

## ES的curl操作

> curl 命令是利用 URL 语法在命令行方式下工作的开源文件传输工具，使用 curl 可以模拟请求。简单的认为是可以在命令行下访问url的一个工具。
>> elasticsearch 提供了一个分布式、支持多用户的全文检索引擎，可以使用 RESTful API 通过端口 9200 和 ES 进行通信。 <br>  <br> 
elasticsearch rest api 遵循的格式为: <br> 
**curl -X REST Verb> <Node>:<Port>/<Index>/<Type>/<ID>** <br> 
 <br> 

 
 ~~~python
 curl -X GET 127.0.0.1:9200/_analyze?pretty -H 'Content-Type: application/json' -d ' {
  "analyzer": "standard",
  "text": "我是中国人"
}'


参数说明：
 -X 指定http的请求方法 有HEAD GET POST PUT DELETE
 -d 指定要传输的数据
 -H 指定http请求头信息
~~~

- 检查 ES 版本信息
    ~~~python
    curl IP:9200
    ~~~

- 查看集群是否健康
    ~~~python
    curl http://IP:9200/_cat/health
    ~~~

- 查看节点列表
    ~~~python
    curl http://IP:9200/_cat/nodes
    ~~~

- 查看索引
    ~~~python
    curl 127.0.0.1:9200/_cat/indices
    ~~~

**注：原创文章，转载请注明出处**
