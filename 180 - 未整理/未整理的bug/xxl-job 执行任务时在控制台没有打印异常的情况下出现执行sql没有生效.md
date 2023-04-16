重庆项目在在执行网站漏洞检测任务时

`WebScanHandler.handlerWebScan` 执行任务时在控制台没有打印异常的情况下出现执行sql没有生效

```java
runTimerService.processExecuteWebJobs(executeJobList);
	processScanningItTask(engineService, executeJob, webEngineTaskId);
```

这个方法里调了很多 sql

```java
taskService.updateById(task);
runTimerMapper
processWebTaskScanData(engineService, executeJob, task.getEngineTaskId(), historyId);
```

获取到任务执行的最新进度和结果后，会调 `taskService` 和 `runTimerMapper` 结果 `processWebTaskScanData` 方法里出现了异常

但是异常被事务处理器吃掉，异常没有打印在控制台上

异常原因是，实体类上有些字段在数据库里没有，拼接 sql 语句时拼接上了数据库表里没有的字段

`processWebTaskScanData` 方法本身标注有事务注解，但其他方法没有（包括最外围的 processScanningItTask 或 processExecuteWebJobs 也没有）

调试结果表示，其他方法也被事务管理器管理了。`processWebTaskScanData` 抛异常后也会让其他方法的 sql 回滚

应该是 bladeX 配置了全局的事务管理器