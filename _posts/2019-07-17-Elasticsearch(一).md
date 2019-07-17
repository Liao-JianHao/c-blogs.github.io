---
layout:     post
title:      Elasticsearch(一)
subtitle:   ELK技术栈
date:       2019-07-17
author:     Mr.C
header-img: img/es1.jpg
catalog: true
tags:
    - Elasticsearch
    - Logstash
    - Kibana

---


![es生态圈](http://www.c-blogs.cn/img/es生态圈.png)

## ELK技术栈

> ELK是由Elasticsearch、Logstash和Kibana三个技术的组合，是一种很典型的MVC思想，模型持久层、视图层和控制层。Logstash担任控制层的角色，负责搜集和过滤数据；Elasticsearch担任数据持久层的角色，负责存储数据；Kibana担任视图层角色，拥有各种维度的查询和分析，并使用图形化的界面展示存放在Elasticsearch中的数据

#### Logstash
 
> 其实Logstash的作用就是数据收集器，将各种渠道、各式各样的数据数据通过它收集解析之后格式化输出到Elasticsearch
>> Logstash只做三件事： <br> 
数据输入 <br> 
数据加工 <br> 
数据输出

- 开源的服务器端数据处理管道，支持不同来源采集数据，转换数据，并将数据发送到不同存储库中

> Logstash特性

- 实时解析和转换数据
    - 从IP地址破译出地理座标
    - 将PII数据匿名化，完全排除敏感字段
- 可扩展
    - 200多个插件(日志/数据库/Arcsigh/Netflow)
- 可靠性安全性
    - Logstash 会通过持久化队列来保证至少将运行中的事件送达一次
    - 数据传输加密
- 监控

#### Kibana

> 基于Logstash的工具，数据可视化工具，帮助用户揭开对数据的任何疑问
>> Kibana 提供了 Console的功能，在Kibana Console中练习ES的API

#### Elasticsearch

> ES是一款基于(Lucene)的开源、分布式、RESTful接口的全文搜索引擎，从海量数据中快速找到相应的内容
>> 主要功能：分布式收搜引擎，大数据近实时分析引擎(近实时：从写入数据到数据可以被搜索有一个小延迟，大概1秒)

###### 版本比较

新特性 5.X

- lucene 6.X，性能提升，默认打分机制从 TF-IDF 改为 BM 25
- 支持 Ingest 节点 / Painless Scripting / Completion suggested 支持 / 原生的 Java REST 客户端
- Type 标记程 deprecated，支持 Keyword 的类型
- 性能优化
    - 内部引擎移除了避免同一文档并发更新的竞争锁，带来 15% —— 20% 的性能提升
    - instant aggregation，支持分片上聚合的缓存
    - 新增 Profile API

新特性 6.X

- Lucene 7.X
- 新功能
    - 跨集群复制(CCR)
    - 索引生命周期管理
    - SQL的支持
- 更友好的升级及数据迁移
    - 在主要版本之间迁移更为简化，体验升级
    - 全新的基于操作的数据复制框架，可加快恢复数据
- 性能优化
    - 有效存储稀疏字段的新方法，降低了存储成本
    - 在索引时进行排序，可加快排序的查询性能

新特性 7.X

- Lucene 8.0
- 重大改进 - 正式废除单个索引下多 Type 的支持
- 7.1 开始，Security 功能免费使用
- ECK - Elasticsearch Operator on Kubernetes
- 新功能
    - New Cluster coordination
    - Feature-Complete High Level REST Client
    - Script Score Query
- 性能优化
    - 默认的Primary Shard 数从5改为1，避免Over Sharding
    - 性能优化，更快的Top K

###### Elasticsearch 的文件目录结构

|目录|配置文件|描述|
|----|--------|----|
|bin||脚本文件，包括启动ES，安装插件。运行统计数据等|
|config|elasticsearch.yml|集群配置文件，user，role based 相关配置|
|JDK||Java运行环境|
|data|path.data|数据文件|
|lib||Java类库|
|logs|path.log|日志文件|
|modules||包含所以ES模块|
|pluginx||包括所以已安装插件|

###### Plugin组件

-  analysis-ice 国际化分词插件
    
    > icu分词插件,是Elasticsearch的分析器插件，使用国际化组件Unicode(ICU)提供丰富的处理 Unicode编码的工具。该查件对处理亚洲语言特别有用，还有大量对除英语外其他语言进行正确匹配和排序所必须的分词过滤器。

- analysis-ik 中文分词插件
    
    > 是一款基于词典和规则的中文分词器


**注：原创文章，转载请注明出处**

