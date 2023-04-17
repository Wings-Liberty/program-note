#还没有复习 

# 草稿



计划讲的内容

- 微服务架构
- 基本使用：服务端，客户端的搭建和配置
- nacos 和 opfeign 的整合 - rabbion
- restful 接口访问形式
- 读写分离
- raft 一致性协议
- 横向对比：zk（dubbo 在用，老版 kafka 在用），eureka（老版 Spring Cloud Netflix 在用）



计划用时：30 min



# 微服务架构





# Nacos Quick Start



注册中心常用架构

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220828223542135.png" alt="image-20220828223542135" style="zoom:80%;" />

nacos 的架构

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220828223710587.png" alt="image-20220828223710587" style="zoom:80%;" />



服务提供者配置服务名，需要向外暴露的接口后就和 nacos 通信，向 nacos 注册自己的服务。并向 nacos 进行心跳探活



消费者主动获取 nacos 的服务列表，根据服务名和负载均衡结果选择提供服务的一台主机，并进行远程调用



上图的 Nacos Server 中包含了  OpenAPI 模块，Nacos 把注册中心对外提供 http api 供客户端调用



<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220828225130632.png" alt="image-20220828225130632" style="zoom:80%;" /><img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220828225207054.png" alt="image-20220828225207054" style="zoom:80%;" />



# Nacos 和 OpenFeign 的整合



服务注册于发现架构下，消费者的客户端常会**缓存一份服务列表并定时更新**，列表保存服务和提供服务的主机地址



调用服务时会在服务列表中根据服务名和负载均衡结果选择一台主机调用



Nacos 默认集成 Rabbion 实现上述行为，Nacos 会把服务列表放在 Rabbion 的 ServerList

OpenFeign 也是通过继承 Rabbion 实现的负载均衡



所以两者能无缝整合依赖于 Rabbion，OpenFeign 调用服务时获取到的 ServerList 就是 Nacos 提供的



# 读写分离



分析客户端发请求到服务端收请求后整个注册流程



到 nacos 真正要注册的时候有三大部分需要分析

```java
// ConsistencyService.class
consistencyService.put(key, instances);
```

- 把服务注册到本地的服务列表上（读写分离实现）
- 即使通知消费者更新列表（spring 事件发布模型实现）
- 把注册到 nacos 的服务同步到 nacos 集群



服务注册也是事件发布模型，把服务注册事件发布后，让服务注册任务异步执行，并让主线程直接继续向下执行尽快给客户端返回响应，很多开源框架都是采用这种方式提高响应速度



以下是读写分离方式更新服务列表

```java
public void updateIps(List<Instance> ips, boolean ephemeral) {
        
    Set<Instance> toUpdateInstances = ephemeral ? ephemeralInstances : persistentInstances;

    HashMap<String, Instance> oldIpMap = new HashMap<>(toUpdateInstances.size());
	// 1. 把旧列表复制一份
    for (Instance ip : toUpdateInstances) {
        oldIpMap.put(ip.getDatumKey(), ip);
    }
	// 2. 把新注册的服务和旧列表上的服务合并起来
    List<Instance> updatedIPs = updatedIps(ips, oldIpMap.values());
    
    toUpdateInstances = new HashSet<>(ips);

    // 3. 替换掉 nacos 当前对外提供的服务列表
    if (ephemeral) {
        ephemeralInstances = toUpdateInstances;
    } else {
        persistentInstances = toUpdateInstances;
    }
}
```



# 通知消费者更新服务列表



```java
public void serviceChanged(Service service) {
    // 通知
    this.applicationContext.publishEvent(new ServiceChangeEvent(this, service));
}
```



接收事件

```java
public void onApplicationEvent(ServiceChangeEvent event) {
    Service service = event.getService();
    String serviceName = service.getName();
    String namespaceId = service.getNamespaceId();

    Future future = GlobalExecutor.scheduleUdpSender(() -> {
            Map<String, Object> cache = new HashMap<>(16);
            long lastRefTime = System.nanoTime();
            for (PushClient client : clients.values()) {

                Receiver.AckEntry ackEntry;
              
                String key = getPushCacheKey(serviceName, client.getIp(), client.getAgent());
                byte[] compressData = null;
                Map<String, Object> data = null;

                if (compressData != null) {
                    ackEntry = prepareAckEntry(client, compressData, data, lastRefTime);
                } else {
                    ackEntry = prepareAckEntry(client, prepareHostsData(client), lastRefTime);
                    if (ackEntry != null) {
                        cache.put(key, new org.javatuples.Pair<>(ackEntry.origin.getData(), ackEntry.data));
                    }
                }
				// 以 udp 方式发通知。没有长连接，节省资源；超时重传；保证了实时性和提高性能瓶颈
                udpPush(ackEntry);
            }
    }, 1000, TimeUnit.MILLISECONDS);

}
```



# Raft 算法



- [ ] [JavaGuide Raft](https://snailclimb.gitee.io/javaguide/#/docs/distributed-system/theorem&algorithm&protocol/raft-algorithm)
- [ ] [Raft算法 - 个人博客](https://tanxinyu.work/raft/)
  - [ ] [raft 官网](https://raft.github.io/)
  - [ ] [raft 动画教程](http://thesecretlivesofdata.com/raft/)
  - [ ] [raft 会议论文](https://raft.github.io/raft.pdf)
  - [ ] [raft 博士论文](https://web.stanford.edu/~ouster/cgi-bin/papers/OngaroPhD.pdf)
  - [ ] [raft 论文笔记](https://blog.laisky.com/p/raft/#集群节点变更-ohtdR)
  - [ ] [6.824 Raft 讲义 1](http://nil.csail.mit.edu/6.824/2020/notes/l-raft.txt)
  - [ ] [6.824 Raft 讲义 2](http://nil.csail.mit.edu/6.824/2020/notes/l-raft2.txt)
- [ ] https://github.com/OneSizeFitsQuorum/raft-thesis-zh_cn/blob/master/raft-thesis-zh_cn.md
- [ ] https://github.com/ongardie/dissertation/blob/master/stanford.pdf
- [ ] https://knowledge-sharing.gitbooks.io/raft/content/chapter5.html



[Raft 算法在 Nacos 中的实现](https://www.jianshu.com/p/76d61de06db9)

[Nacos 数据一致性](https://blog.csdn.net/liyanan21/article/details/89320872)



PPT 里的代码截图可以在 nacos 里找，简化为伪代码后展示；Raft 截图可以在 [raft 动画教程](http://thesecretlivesofdata.com/raft/) 里截



最好在来一个 eureka 和 zk 的对比。说完 zk 后，提一句新版 kafka 自己实现了分布式一致性协调





