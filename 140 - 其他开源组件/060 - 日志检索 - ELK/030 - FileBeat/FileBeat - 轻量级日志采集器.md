#还没有复习 

参考：

- [Beats 和 FileBeat 轻量级日志采集器简介](http://t.zoukankan.com/songgj-p-10356987.html)
- [Logstash：结合 FileBeat 把 Apache 日志导入到 Elasticsearch](https://blog.csdn.net/UbuntuTouch/article/details/100727051)
- [Beats：Beats 入门教程 （一）](https://elasticstack.blog.csdn.net/article/details/104432643)
- [Beats：Beats 入门教程 （二）](https://elasticstack.blog.csdn.net/article/details/104473684)



Filebeat 是一个属于 Beats 系列的日志托运者 （一组安装在主机上的轻量级托运人），采集日志数据并可以输出到多种目的地，比如 logstash，es

![[../../../020 - 附件文件夹/Pasted image 20230402104406.png|500]]

FileBeat 的处理流程如下

![[../../../020 - 附件文件夹/Pasted image 20230402104416.png|500]]

Filebeat 由两个主要组件组成，prospector 和 harvester

harvester（收割者）：负责读取单个文件的内容。如果文件在读取时被删除或重命名，Filebeat 将继续读取文件

prospector（观察者）：负责管理 harvester 并找到所有要读取的文件来源，**主要监控文件是否变化**，如果输入类型为日志，则查找器将查找路径匹配的所有日志文件，并为每个文件启动一个 harvester（上图 **prospector1 就有两个 harvester**），如果文件有变化 harvester 就会收集新的日志





```
项目名称：态势感知主线3.2预研

项目经理：李瀚辰

项目角色：开发

所处阶段：预研阶段

task1：

task2：

项目名称：入职自动化办公项目

项目经理：常旭

项目组成员：

所处阶段：二期开发维护阶段
```

