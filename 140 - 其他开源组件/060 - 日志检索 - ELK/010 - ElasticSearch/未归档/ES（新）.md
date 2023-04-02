

ELK 能够安全可靠地获取任何来源、任何格式的数据，然后实时地对数据进行搜索、分析和可视化

ES 是一个开源的高扩展的分布式全文搜索引擎



- 搜索的对象是非结构化的文本数据
- 文件记录量达到数十万或数百万个甚至更多
- 支持大量基于交互式文本的查询
- 需求非常灵活的全文搜索查询
- 对高度相关的搜索结果的有特殊需求，但是没有可用的关系数据库可以满足



# ES 的数据格式

ES 是面向文档型数据库，一条数据在这里就是一个文档。 将 Elasticsearch 里存储文档数据和关系型数据库 MySQL 存储数据的概念进行一个类比

![[../../../../020 - 附件文件夹/Pasted image 20230402104151.png]]


ES 7.X 中, Type 的概念已经被删除了

> 公司开发，测试环境用的 ES 版本是 7.2


ES 中一个文档（也就是一条数据）用 json 格式保存，如一条用户的信息

```json
{
    "name" : "John",
    "sex" : "Male",
    "age" : 25,
    "birthDate": "1990/05/01",
    "about" : "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```



# ES 客户端



ES 对外暴露 http 接口供外部客户端操作 ES



客户端向 ES 发送 http 请求完成对数据的 curd



## 索引操作

索引的 curd

重复创建同名索引会报错（返回 400 bad request）



## 文档操作

文档/数据 的 curd



文档的修改有：全覆盖式修改，局部修改两种方式

文档的删除是逻辑删除

删除也有两种：指定文档 id 删除，条件删除



## 映射操作



ES 的映射相当于 mysql 中的表结构

创建数据库表需要设置字段名称，类型，长度，约束等；索引库也一样，需要知道这个类型下有哪些字段，每个字段有哪些约束信息，这就叫做映射  



使用映射的目的在于约束指定字段的数据类型

在没有映射时，向索引库 student 插入一个文档，学生成绩为 60。ES 默认认为学生成绩是 long 类型

又插入一个文档，学生成绩是 60.5，ES 默认认为学生成绩是 double 类型



如果此时对接 Java，查询某些学生的成绩，Java 实体类型封装 ES 返回的文档就存在精度损失或类型不匹配

所以映射约束了字段的数据类型，能便于 ES 对接客户端的实体类



映射的 curd

索引映射关联和创建映射两个接口好像是一样的



## 高级查询

ES 提供了基于 JSON 提供完整的查询 DSL 来定义查询  

- 根据唯一 id 查询一个文档

- 全库查询

  ```json
  {
      "query": {
          "match_all": {}
      }
  }
  ```

- 匹配查询

  ```json
  {
      "query": {
          "match": {
              "name":"zhangsan"
          }
      }
  }
  ```

- 字段匹配查询。可以在多个字段中查询

  ```json
  {
      "query": {
          "multi_match": {
              "query": "zhangsan",
              "fields": ["name","nickname"]
          }
      }
  }
  ```

- 关键字精确查询。精确的关键词匹配查询，不对查询条件进行分词。  

  ```json
  {
      "query": {
          "term": {
              "name": {
             		"value": "zhangsan"
              }
          }
      }
  }
  ```

- 多关键字精确查询。允许指定多值进行匹配

  ```json
  {
      "query": {
          "terms": {
              "name": [
                  "zhangsan",
                  "lisi"
              ]
          }
      }
  }
  ```

- 指定查询字段。只返回指定字段，不返回所有字段。类似`select *`和`select select_list`

  ```json
  {
      "_source": [
          "name",
          "nickname"
      ],
      "query": {
          "terms": {
              "nickname": [
                  "zhangsan"
              ]
          }
      }
  }
  ```

- 过滤字段，和指定查询字段类似。用 includes：来指定想要显示的字段；excludes：来指定不想要显示的字段

  ```json
  {
      "_source": {
          "includes": [
              "name",
              "nickname"
          ]
      },
      "query": {
          "terms": {
              "nickname": [
                  "zhangsan"
              ]
          }
      }
  }
  ```

- 组合查询。设置查询条件必须满足 field1=aa，必须不满足 field2=bb，应该满足 field3=cc

  ```json
  {
      "query": {
          "bool": {
              "must": [
                  {
                      "match": {
                          "name": "zhangsan"
                      }
                  }
              ],
              "must_not": [
                  {
                      "match": {
                          "age": "40"
                      }
                  }
              ],
              "should": [
                  {
                      "match": {
                          "sex": "男"
                      }
                  }
              ]
          }
      }
  }
  ```

- 范围查询

  ```json
  {
      "query": {
          "range": {
              "age": {
                  "gte": 30,
                  "lte": 35
              }
          }
      }
  }
  ```

- 模糊查询。实现距离是**编辑距离**，一个单词转为另一个单词需要执行的转换次数。转换行为包含，修改，删除，添加一个字符，交换两个字符位置

  具体实现方式见学习文档

  ```json
  {
      "query": {
          "fuzzy": {
              "title": {
                  "value": "zhangsan"
              }
          }
      }
  }
  ```

- 单字段排序，多字段排序

  ```json
  {
      "query": {
          "match": {
              "name": "zhangsan"
          }
      },
      "sort": [
          {
              "age": {
                  "order": "desc"
              }
          }
      ]
  }
  ```

  ```json
  {
      "query": {
          "match_all": {}
      },
      "sort": [
          {
              "age": {
                  "order": "desc"
              }
          },
          {
              "_score": {
                  "order": "desc"
              }
          }
      ]
  }
  ```

  首先按照年龄排序，然后按照相关性得分排序

- 高亮显示。要求查询到的文档中，高亮显示匹配到的关键字。实现方式是用 h5 样式标签包裹查询到的关键字

  ```json
  {
      "query": {
          "match": {
              "name": "zhangsan"
          }
      },
      "highlight": {
          "pre_tags": "<font color='red'>",
          "post_tags": "</font>",
          "fields": {
              "name": {}
          }
      }
  }
  ```

- 分页查询

  ```json
  {
      "query": {
          "match_all": {}
      },
      "from": 0,
      "size": 2
  }
  ```

- **聚合查询**。类似 MySQL 的 group by。例如计数、取最大最小值、平均值，求和等

  ```json
  {
      "aggs": {
          "sum_age": {
              "sum": {
                  "field": "age"
              }
          }
      },
      "size": 0
  }
  ```

- 桶聚合查询。相当于 sql 中的 group by 语句。能实现聚合查询，还能实现分组下再进行聚合

  ```json
  {
      "aggs": {
          "age_groupby": {
              "terms": {
                  "field": "age"
              }
          }
      },
      "size": 0
  }
  ```

  terms 下能再进行聚合

  ```json
  {
      "aggs": {
          "age_groupby": {
              "terms": {
                  "field": "age",
                  "aggs": {
                      "sum": {
                          "field": "age"
                      }
                  }
              }
          }
      },
      "size": 0
  }
  ```

匹配查询和模糊查询的区别？





# Java Client API

用 Spring 的 ElasticsearchTemplate



# ES 环境



- 单机，集群



# ES 核心概念

- 分片，副本，分配的概念
- mget 和 bulk 批量请求



集群管理：状态查询，故障转移，水平扩容，应对故障，路由计算分片控制

数据的读写，更新流程