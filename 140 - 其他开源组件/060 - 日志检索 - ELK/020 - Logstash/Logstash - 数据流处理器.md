#还没有复习 

参考

- [Logstash 简单介绍](https://www.cnblogs.com/WeiKing/p/13448973.html)

- [Logstash 入门教程（一）- LogStash 是什么](https://blog.csdn.net/ubuntutouch/article/details/105973985)

  三大组件都有哪些插件可用可以直接参考[官网](https://www.elastic.co/guide/en/logstash/current/plugins-filters-mutate.html)

  Elastic 中文社区也提供了一些插件的使用方式介绍，[Gork filter 入门](https://elasticstack.blog.csdn.net/article/details/105922198)，[mutate 过滤器](https://elasticstack.blog.csdn.net/article/details/106466873)

- [Logstash 入门教程 （二）- 如何安装和运行一个 Logstash 以及如何在配置文件中配置三大组件](https://blog.csdn.net/UbuntuTouch/article/details/105979677)

- [Logstash：结合 FileBeat 把 Apache 日志导入到 Elasticsearch](https://blog.csdn.net/UbuntuTouch/article/details/100727051)

  - [Beats 和 FileBeat 轻量级日志采集器简介](http://t.zoukankan.com/songgj-p-10356987.html)



# 简单介绍

Logstash 是一个 input | decode | filter | encode | output 的数据流处理器

接收指定数据源数据，解码后对原有数据进行筛选，加工，再次编码后输出到 target

![[../../../020 - 附件文件夹/Pasted image 20230402104642.png]]

logstash 处理数据的流程是固定的，但这些核心组件都是插件化的，都能通过添加或更换插件修改处理数据的细节

logstash 收到数据后将其转为事件，并添加一些额外信息帮助 logstash 和插件处理

对于 开发者来说只需要关注 logstash 的配置文件即可


1. Logstash 中的数据类型：
   1. bool：use_column_value => true
   2. string：jdbc_driver_class => "com.mysql.jdbc.Driver"
   3. number：jdbc_page_size => 50000
   4. array：hosts => ["192.168.57.100:9200","192.168.57.101:9200","192.168.57.102:9200"]
   5. hash：options =>{key1 =>value1,key2 =>value2}
2. logastah 中的逻辑运算符：
   1. 相等关系：==、!=、<、>、<=、>=
   2. 正则关系：=~（匹配正则）、!~(不匹配正则）
   3. 包含关系：in、not in
   4. 布尔操作：and(与）、or（或）、nand（非与）、xor（非或）
   5. 一元运算符：!（取反）、()（复合表达式）、!() （对复合表达式结果取反）


这些都能写在配置文件里


Logstash **不是集群组件**，无法感知其他 Logstash 实例。 通过跨实例负载平衡数据，可以使用多个 Logstash 实例来满足高可用性和扩展需求

Logstash 通过运行一个或多个 Logstash 管道作为 Logstash 实例的一部分来处理 ETL 工作负载

![[../../../020 - 附件文件夹/Pasted image 20230402104657.png]]

# 如何配置 Logstash pipeline

Logstash 管道有两个必需元素，输入（inputs）和输出（ouputs），以及一个可选元素 filters。 输入插件使用来自源的数据，过滤器插件在你指定时修改数据，输出插件将数据写入目标。

![[../../../020 - 附件文件夹/Pasted image 20230402104705.png]]

![[../../../020 - 附件文件夹/Pasted image 20230402104710.png]]

