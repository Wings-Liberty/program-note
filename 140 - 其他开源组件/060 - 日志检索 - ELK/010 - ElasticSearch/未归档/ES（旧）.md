#还没有复习 

## 前言

elasticsearch的基本使用在JavaEE的SpringBoot整合高级篇中有

这里描述的是elasticsearch的分词等使用



# 分词

> elasticsearch的分词对中文不友好，会将中文全部拆分为单个汉字。所以需要走下载中文分词器

---

## 下载使用中文分词器

[分词器的github地址](https://github.com/medcl/elasticsearch-analysis-ik)

下载慢的话可以将github仓库导入自己的gitee中，在gitee中下载快

下载后，对项目进行`mvn clean package`后拿到target/releases下的.zip压缩包解压到你es的plugins目录下并解压文件。==解压后的文件必须是一个文件夹A，文件夹A下是jar包和配置文件等。不允许文件夹A下有文件夹B，文件夹B下才是jar包和配置文件等（即不允许有多层文件夹包裹核心文件，至于文件夹A的命名，随意）==



==下载的分词器的版本一定要根据自己的es版本号选择==我的es是5.6.12但我使用的分词器是5.6.9

因为下载的5.6.12班的分词器，会变成5.6.9，官方对5.6.12版的提交信息进行的描述是“update to 5.6.9”反正就是下载的5.6.12到手后莫名变为5.6.9.尝试启动，报错，版本不对。尝试直接修改文件名中的版本号和一个配置文件中的版本号。运行成功.......



## 分词的简单实用

该分词器有两种模式

`ik_smart`简易分词

`ik_max_word`尽最大可能分词

```json
GET _analyze
{
  "analyzer": "ik_max_word",
  "text": "我是中国人"
}
```



# 集群

> 之前不管是啥集群，从来都没有成功做过。这次用docker有望成功启动集群。噢耶！

使用集群的关键是修改kibana和es的配置文件

前者可运行docker的时候使用-c 选项指定外部的配置文件

失败，跳过



# 将mysql数据导入es



## 设计阶段

设计数据库中那些表的数据需要导入es

表中的哪些字段需要导入es

哪些字段是用来展示的，哪些字段使用来做索引的



## es中的数据结构



### es中的默认数据类型

`boolean` `long` `doble` `date` `text/keyword`

前三个不做解释

`date` 举例 '2020-03-21'

`text`和`keyword`都是字符串，前者能分词，后者不能分词



### mappings定义数据结构

> 使用es的`mappings`功能给索引定义数据结构，就像mysql定义一张表的数据结构一样

es6中每个索引你只能有一个类型，每个索引也只能有一种数据结构

es5比6宽松，一个索引能有多个类型和多个数据结构

```json
PUT gmall
{
  "mappings": {
    "SkuInfo":{
      "properties": {
        "id":{
          "type": "keyword"
          , "index": false
        },
        "price":{
          "type": "double"
        },
         "skuName":{
          "type": "text",
          "analyzer": "ik_max_word"
        },
        "skuDesc":{
          "type": "text",
          "analyzer": "ik_smart"
        },
        "catalog3Id":{
          "type": "keyword"
        },
        "skuDefaultImg":{
          "type": "keyword",
          "index": false
        },
        "skuAttrValueList":{
          "properties": {
            "valueId":{
              "type":"keyword"
            }
          }
        }
      }
    }
  }
}
```



### 过滤+查询

> 搜索功能由过滤+查询组成，查询指根据输入的关键字，过滤指过滤条件
>
> 如：查询换位手机+过滤条件（4.4寸以下）

```json
// 格式
Query{
    Bool:{ // 先过滤，后查询
       Filter:{term,term} // 过滤
       must:{match} // 查询条件
    }
}

// 实例
GET gmall/PmsSkuInfo/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": { //符合要求的会被命中
          "FIELD": "VALUE"
        }
      }
      , "must": [
        {
          "match": {
            "FIELD": "TEXT"
          }
        }
      ]
    }
  }
}
```

**交集并集过滤**

```json
// 交集实例暂略，因为没有尝试过

// 并集实例
GET gmall/PmsSkuInfo/_search
{
  "query": {
    "bool": {
      "filter": {
        "terms": { // 并集过滤使用terms，所有符合要求的都会被命中
          "FIELD": ["VALUE1","VALUE2","VALUE3"] // 
        }
      }
      , "must": [
        {
          "match": {
            "FIELD": "TEXT"
          }
        }
      ]
    }
  }
}
```



# java操作es

---

## ElasticsearchRestTemplate

### 配置

```java
@Configuration
public class SearchClientConfig extends AbstractElasticsearchConfiguration {
    @Override
    @Bean
    public RestHighLevelClient elasticsearchClient() {
        ClientConfiguration configuration = ClientConfiguration.builder()
                .connectedTo("39.107.101.13:9200")
                .build();
        RestHighLevelClient client = RestClients.create(configuration).rest();
        return client;
    }
}
```

### 增删改查

> 对索引和类型的增删改，对数据的增删改查（主要是查）



**条件查询**

```java
@Autowired
private ElasticsearchRestTemplate template;

@Test
public void testTemplate() {
    String field = "age";
    int value = 14;

    NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
        .withQuery(QueryBuilders.termQuery(field, value))  // 设置查找条件
        .withHighlightFields(new HighlightBuilder.Field(field)).build(); // 设置高亮
    AggregatedPage<User> users = template.queryForPage(searchQuery, User.class);
    List<User> userList = users.getContent();
    for (User user : userList) {
        System.out.println(user);
    }
}
```

