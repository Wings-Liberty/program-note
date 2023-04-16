E：ElasticSearch
L：Logstash
K：Kibana

- ES 是提供 RestFul 接口的搜索引擎
- Logstash 能对日志进行收集、过滤、分析，支持大量的数据获取方法

常见方式是

Java 程序输出日志到本机的 Logstash 服务上，Logstash 对日志进行收集和过滤，筛掉不要的，把剩下的发给 ES 供以后检索

![21068378-c27870fdc01a3ba0](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/21068378-c27870fdc01a3ba0.webp)


