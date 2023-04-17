#还没有复习 

# 概述

flowable 实现了一个流程图

比如设置一个请假流程，流程包含 n 个步骤，每个步骤要求若干个人会签、或签

用 flowable 部署一套流程的完整步骤为

1. **定义**一套流程，输出一个 xml 文件
2. **部署**流程文件到数据库，形成一个运行时的**流程定义**
3. 根据流程定义**启动**一个流程实例

流程实例由多个节点和箭头组成

![Pasted image 20221119222256](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221119222256.png)


# 数据库

- 所有数据库表都以 `ACT_` 开头
- 第二部分是说明表用途的两字符标示符。服务 API 的命名也大略符合这个规则

- `ACT_RE_`：`RE` 代表 `repository`。表示“静态”信息，例如流程定义与流程资源（图片、规则等）。
- `ACT_RU_`：`RU` 代表 `runtime`。存储运行时信息，例如流程实例（process instance）、用户任务（user task）、变量（variable）、作业（job）等。Flowable 只在流程实例运行中保存运行时数据，并在流程实例结束时删除记录
- `ACT_HI_`：`HI` 代表 `history`。存储历史数据，例如已完成的流程实例、变量、任务等
- `ACT_GE_`：通用数据。在多处使用


具体的表结构的含义:

| **表分类**   | **表名**              | **解释**                                           |
| ------------ | --------------------- | -------------------------------------------------- |
| 一般数据     |                       |                                                    |
|              | [ACT_GE_BYTEARRAY]    | 通用的流程定义和流程资源                           |
|              | [ACT_GE_PROPERTY]     | 系统相关属性                                       |
| 流程历史记录 |                       |                                                    |
|              | [ACT_HI_ACTINST]      | 历史的流程实例                                     |
|              | [ACT_HI_ATTACHMENT]   | 历史的流程附件                                     |
|              | [ACT_HI_COMMENT]      | 历史的说明性信息                                   |
|              | [ACT_HI_DETAIL]       | 历史的流程运行中的细节信息                         |
|              | [ACT_HI_IDENTITYLINK] | 历史的流程运行过程中用户关系                       |
|              | [ACT_HI_PROCINST]     | 历史的流程实例                                     |
|              | [ACT_HI_TASKINST]     | 历史的任务实例                                     |
|              | [ACT_HI_VARINST]      | 历史的流程运行中的变量信息                         |
| 流程定义表   |                       |                                                    |
|              | [ACT_RE_DEPLOYMENT]   | 部署单元信息                                       |
|              | [ACT_RE_MODEL]        | 模型信息                                           |
|              | [ACT_RE_PROCDEF]      | 已部署的流程定义                                   |
| 运行实例表   |                       |                                                    |
|              | [ACT_RU_EVENT_SUBSCR] | 运行时事件                                         |
|              | [ACT_RU_EXECUTION]    | 运行时流程执行实例                                 |
|              | [ACT_RU_IDENTITYLINK] | 运行时用户关系信息，存储任务节点与参与者的相关信息 |
|              | [ACT_RU_JOB]          | 运行时作业                                         |
|              | [ACT_RU_TASK]         | 运行时任务                                         |
|              | [ACT_RU_VARIABLE]     | 运行时变量表                                       |
| 用户用户组表 |                       |                                                    |
|              | [ACT_ID_BYTEARRAY]    | 二进制数据表                                       |
|              | [ACT_ID_GROUP]        | 用户组信息表                                       |
|              | [ACT_ID_INFO]         | 用户信息详情表                                     |
|              | [ACT_ID_MEMBERSHIP]   | 人与组关系表                                       |
|              | [ACT_ID_PRIV]         | 权限表                                             |
|              | [ACT_ID_PRIV_MAPPING] | 用户或组权限关系表                                 |
|              | [ACT_ID_PROPERTY]     | 属性表                                             |
|              | [ACT_ID_TOKEN]        | 记录用户的token信息                                |
|              | [ACT_ID_USER]         | 用户表                                             |


# 执行流程简介


## 定义流程文件

流程文件是一个 xml 文件

![getting.started.bpmn.process](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/getting.started.bpmn.process.png)

如果把上述流程转为 xml 文件 - `_holiday-request.bpmn20.xml`，就是下面的结果

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:xsd="http://www.w3.org/2001/XMLSchema"
             xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI"
             xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC"
             xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI"
             xmlns:flowable="http://flowable.org/bpmn"
             typeLanguage="http://www.w3.org/2001/XMLSchema"
             expressionLanguage="http://www.w3.org/1999/XPath"
             targetNamespace="http://www.flowable.org/processdef">
    <process id="holidayRequest" name="Holiday Request" isExecutable="true">
        <startEvent id="startEvent"/>
        <sequenceFlow sourceRef="startEvent" targetRef="approveTask"/>
        <userTask id="approveTask" name="Approve or reject request"/>
        <sequenceFlow sourceRef="approveTask" targetRef="decision"/>
        <exclusiveGateway id="decision"/>
        <sequenceFlow sourceRef="decision" targetRef="externalSystemCall">
            <conditionExpression xsi:type="tFormalExpression">
                <![CDATA[
          ${approved}
        ]]>
            </conditionExpression>
        </sequenceFlow>
        <sequenceFlow  sourceRef="decision" targetRef="sendRejectionMail">
            <conditionExpression xsi:type="tFormalExpression">
                <![CDATA[
          ${!approved}
        ]]>
            </conditionExpression>
        </sequenceFlow>
        <serviceTask id="externalSystemCall" name="Enter holidays in external system"
                     flowable:class="org.flowable.CallExternalSystemDelegate"/>
        <sequenceFlow sourceRef="externalSystemCall" targetRef="holidayApprovedTask"/>
        <userTask id="holidayApprovedTask" name="Holiday approved"/>
        <sequenceFlow sourceRef="holidayApprovedTask" targetRef="approveEnd"/>
        <serviceTask id="sendRejectionMail" name="Send out rejection email"
                     flowable:class="org.flowable.SendRejectionMail"/>
        <sequenceFlow sourceRef="sendRejectionMail" targetRef="rejectEnd"/>
        <endEvent id="approveEnd"/>
        <endEvent id="rejectEnd"/>
    </process>
</definitions>
```

如果把 XML 保存在 `src/main/resources`，启动引擎时就能自动部署（如果没有部署过）

## 部署流程定义

部署一个流程定义意味着：

-  流程引擎会将XML**文件存储在数据库中**，这样可以在需要的时候获取它
    
-  流程定义转换为内部的、可执行的对象模型，这样使**用它就可以启动_流程实例**

```java
ProcessEngine processEngine = cfg.buildProcessEngine();
RepositoryService repositoryService = processEngine.getRepositoryService();
Deployment deployment = repositoryService.createDeployment()
		.addClasspathResource("holiday-request.bpmn20.xml")
		.name("请求流程")
		.deploy();
System.out.println("deployment.getId() = " + deployment.getId());
System.out.println("deployment.getName() = " + deployment.getName());
```


> [!NOTE] 数据库记录
> - `act_re_deployment`：流程定义部署表，每部署一次就增加一条记录（包含 `deployment_id`）
> 
> - `act_re_procdef`：流程定义表，每部署一个新的流程定义都会在这加一条记录（包含 `deployment_id` ，`processDefinitionId`和 `processDefinitionKey`
> 
> - `act_ge_bytearray`：流程资源表，流程部署的 bpmn 文件和 png 图片会保存在这
> 
> - `ACT_DE_MODEL` 也会加一条，它保存的是 json 格式的流程定义信息


`act_re_deployment` 和 `act_re_procdef` 是一对多的关系，一个 xml 文件可以包含多个流程

也就是说，一次部署可以部署多个流程定义

## 用流程定义启动流程实例

用 `processDefinitionId` 或 `processDefinitionKey` 可以启动一个流程实例

启动流程实例后就会向 `act_ru` 和 `act_hi` 表里加数据


> [!NOTE] 数据库记录
> 
> 运行时数据
> 
> - `act_ru_execution`：流程执行信息 - 每条数据对应每个正在执行的流程的节点（所有类型的节点）
> - `act_ru_identitylink`：流程的参与用户信息
> - `act_ru_task`：任务信息 - 还正在执行的流程，每条数据对应每个正在执行的流程的 task 节点（所有类型的节点）
> 
> 历史数据
> 
>  - `act_hi_actinst`：流程实例执行历史 - 每条对应一个流程实例执行过的节点（和 act_hi_taskinst 的区别好像在于，这里会记录所有类型的节点，act_hi_taskinst只会记录 task 节点）
>  - `act_hi_identitylink`：流程的参与用户的历史信息
>  - `act_hi_procinst`：流程实例历史信息 - 创建一个流程实例，就有一条数据
>  - `act_hi_taskinst`：流程任务历史信息 - 每条对应一个流程实例执行过的 task 节点

启动一个流程实例后，流程实例的标识符是 `processInstanceId`

启动一个流程后，会在 `ACT_RU_` 对应的表结构中操作，运行时实例会涉及的表结构共 10 张：

-   ACT_RU_DEADLETTER_JOB 正在运行的任务表
-   ACT_RU_EVENT_SUBSCR 运行时事件
-   ACT_RU_EXECUTION 运行时流程执行实例
-   ACT_RU_HISTORY_JOB 历史作业表
-   ACT_RU_IDENTITYLINK 运行时用户关系信息
-   ACT_RU_JOB 运行时作业表
-   ACT_RU_SUSPENDED_JOB 暂停作业表
-   ACT_RU_TASK 运行时任务表
-   ACT_RU_TIMER_JOB 定时作业表
-   ACT_RU_VARIABLE 运行时变量表

**启动一个流程实例的时候涉及到的表有**

-   ACT_RU_EXECUTION 运行时流程执行实例。表示当前正在执行的任务节点，如果是并行网关，一个流程实例就可能有多个执行实例
    
-   ACT_RU_IDENTITYLINK 运行时用户关系信息
    
-   ACT_RU_TASK 运行时任务表。每走到一个任务节点就创建一条数据
    
-   ACT_RU_VARIABLE 运行时变量表


> [!NOTE] 流程实例和执行实例的关系
> 根据流程定义能启动多个流程实例，一个流程实例执行时正在执行的任务节点可以有多个，比如用了并行网关后就可能存在多个任务同时处于等待被完成的状态
> 
> `ACT_RU_EXECUTION` 就是执行实例表





# 执行工作流时的数据库变化


## 任务分配和流程变量

任务可以被分配给指定的人，这个人可以是一个变量

变量用 `ACT_RU_VARIABLE` 表

任务分配给了谁可以在 `ACT_RU_TASK` 里看

## 用户和组管理

用户数据在 `ACT_ID_USER`

用户组数据在 `ACT_ID_GROUP`

用户和用户组的中间表是 `ACT_ID_MEMBERSHIP`

## 多人会签

如果一个任务节点是多人会签，走到这个节点后会在 `ACT_RU_TASK` 里给每个人都创建一个任务

# 使用方式

用 SpringBoot 整合 flowable 方式开发

## 初始化数据库

启动即可。但启动后缺少部分和 ui 相关的表。如果是用 flowbale-ui 项目初始化的数据库，就会补上缺少的几张表

```java
@SpringBootTest(classes = {Application.class})  
@RunWith(SpringRunner.class)  
public class Init {  
    @Autowired  
    private ProcessEngine processEngine;  
  
    @Test  
    public void initDB() {  
        System.out.println(processEngine);  
    }  
}
```

## 部署流程定义

部署流程的方式有

- 把 xml 文件放到指定目录下，启动项目自动部署
- 把 xml 文件放到资源目录下，手动指定 xml 地址部署
- 把 xml 字符串或字节数组或模型对象或输入流传给 API 进行部署
- 用 flowable-ui 手动点击【发布】按钮

### 自动部署 - SpringBoot 整合 flowable

- `processes` 目录下的任何 BPMN 2.0 流程定义都会被自动部署
- `cases` 目录下的任何 CMMN 1.1 事例都会被自动部署
- `forms` 目录下的任何 Form 定义都会被自动部署。

在 `processes` 下放一个 xml，启动 spring 后部署上一个流程定义，但 flowable-ui 上查不到

### 手动指定 xml 位置部署流程定义

```java
Deployment deployment = repositoryService.createDeployment()  
        .addClasspathResource("holiday-request-new.bpmn20.xml")  
        .name("新请求流程")  
        .deploy();
```

### 传入 xml 的字符串，字节数组，火星对象或输入流给 API 进行部署

```java
Deployment deployment = repositoryService.createDeployment()  
        .addBytes(resourceName, data) // 字节数组
        // .addString(resourceName, xmlStr) // 字符串和输入流
		// .addInputStream(resourceName, classPathResource.getInputStream())
        .name("deployname") // 是 deployment 的名字，不是流程定义的名字
        .deploy();
```


> [!NOTE] `resourceName` 一定要以 `bpmn20.xml、bpmn` 为结尾
> 不然部署结束后不会插入ACT_RE_PROCDEF 流程定义表


### 数据库变化

执行一次部署，`act_re_deployment` 就会加一条部署记录

一个 xml 文件里有几个流程定义就在 `act_re_procdef` 里加几条流程定义数据

deployment 有自己的 id，name

processDefinition 有自己的 id，key 和它所归属的 deployment_id

流程定义的 id = key:version:deploymentId

重复部署 key 相同的流程会被认为是更新版本，`act_re_procdef` 里会新增一条流程定义数据，版本号递增

## 启动流程实例

根据流程定义的 id 或 key 定位到一个流程定义后就可以启动流程实例了

```java
ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("process1", variables);
// startProcessInstanceById
```

如果一个流程定义在 `act_re_procdef` 有多个版本，会用最新版本的流程定义创建流程实例

创建流程实例后，每个流程实例有一个 processInstanceId，也叫 `PROC_INST_ID`

- `ACT_RU_EXECUTION` 添加数据，表示当前流程实例正在执行哪些任务节点。忽略一条启动事件的数据，如果走到并行网站，一个流程实例就可能有多个**执行实例**

每个执行实例都有 一个 id，并记录自己所属的流程实例 id，流程定义 id，act_id（任务节点的静态 task_def_key）

- `ACT_RU_TASK` 添加数据。执行实例每走到一个任务节点就创建一条数据

每条数据都有一个自己的 id 和 task_def_key。此外还记录自己所属的流程实例 id，流程定义 id，执行实例 id，任务被分配给了谁去执行等

- `ACT_RU_VARIABLE` 里添加变量信息

每个变量就是一条数据，每条数据除记录自己的 id，数据类型，值外也记录自己所属的执行实例 id，流程实例 id


- 向历史表里插数据

>  - `act_hi_actinst`：流程实例执行历史 - 每条对应一个流程实例执行过的节点，节点包括事件，箭头，网关，任务节点等所有类型的节点
>  - `act_hi_identitylink`：流程的参与用户的历史信息
>  - `act_hi_procinst`：流程实例历史信息 - 创建一个流程实例，就有一条数据
>  - `act_hi_taskinst`：流程任务历史信息 - 每条对应一个流程实例执行过的 task 节点

## 查看任务

```java
taskService.createTaskQuery();
```

提供大量 API，支持各种条件查询

## 完成任务

完成任务后，执行实例走到下一个节点。此时查看运行时数据表

- 执行实例还是只有一条数据（此时除了启动事件那条数据，且没有并行网关）
- 任务信息表，被完成的任务被删除，新任务被添加进来


## 查看历史信息

`HistoryService` 提供大量查询 Builder 和条件

## 删除流程

删除方法有一个布尔类型的参数。表示是否级联删除

- 如果为 `false`，删除时，如果流程定义启动有实例，删除会失败
- 如果为 `true`，删除时会删除所有数据。部署记录，流程定义，流程实例，执行实例，变量等运行时数据和所有历史数据

```java
repositoryService.deleteDeployment(deploymentId, true);
```

## 任务分配和流程变量



## 候选人和候选人组

## 网关

## 会签

## 或签

## 任务回退

## 外置表单



## 其他

### 流程的挂起和激活

# 其他类型的任务节点


