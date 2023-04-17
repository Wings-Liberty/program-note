#还没有复习 

# 简单 Demo  
  
不整合到 flowable  
  
## 待验证 Demo  
  
- [x] 策略执行顺序和执行条件验证 - TacticsExecutionConditionTest  
- [x] API 动作执行验证。动作执行授权请求，调用API以及动作的运行参数的赋值和输出参数的输出 - ApplicationAndActionExecutionTest  
- [x] 条件分支的条件判断执行验证 - TacticsConditionTest  
- [ ] 解决所有剩余的 todo 和 fixme  
  
## 汇总 todo 和 fixme  
  
- [ ] 条件分支获取左右值  
- [x] 从响应里提取关心的数据。用两个 json 对比，从里面提取出来东西 - GetResponseResultTest  
- [ ] 动作里获取左右值  
  
  
## 从动作里获取值  
  
左值：当前动作的运行参数  
  
右值：上一个节点的输入，全局变量，全局常量，手动输入的字面量  
  
action_variable_id, value_origin, value_id, value_constant  
  
## 从条件里获取值  
  
左值：上一个节点的输入，全局变量  
  
右值：上一个节点的输入，全局变量，全局常量，手动输入的字面量  
  
left_origin, left_id, right_origin, right_id, right_value  
  
  
输出结果  
  
如果是赋值，输出一个 map，内容例如：#(sIp):1.1.1.1, #(direct_visit):2  

# flowable 提供的工具

在这个包下有工具 org.flowable.engine.impl.util

# 整合 Demo 到 flowable  

最终结果，实现走通这个流程

![[../../../14 - 静态资源/Pasted image 20230211183421.png|250]]


- [ ] 用一个接口模拟自动处置触发策略执行条件判断的验证  



1. 直接调接口（不传参数）  
2. 模拟创建一个告警对象  
3. 根据位置信息，对策略排序’  
4. 选择第一个符合执行条件的策略，然后执行  
  
- [ ] API 动作执行验证。动作执行授权请求，调用API以及动作的运行参数的赋值和输出参数的输出
- [ ] 条件分支的条件判断执行验证
- [ ] 人工审批和发送通知

# flowable 的流程变量命名规范

不能有短横线，否则运行时报错。猜测命名规范和 Java 的变量名命名规范类似

# Demo 用的流程 json - UserTask + 监听器实现

```json
{
    "modelId": "90e4b5a9-ab7e-11ed-910c-005056c00008",
    "bounds": {
        "lowerRight": {
            "x": 1200,
            "y": 1050
        },
        "upperLeft": {
            "x": 0,
            "y": 0
        }
    },
    "properties": {
        "process_id": "DemoTest",
        "name": "Demo验证",
        "documentation": "",
        "process_author": "",
        "process_version": "",
        "process_namespace": "http://www.flowable.org/processdef",
        "process_historylevel": "",
        "isexecutable": true,
        "dataproperties": "",
        "executionlisteners": "",
        "eventlisteners": "",
        "signaldefinitions": "",
        "messagedefinitions": "",
        "escalationdefinitions": "",
        "process_potentialstarteruser": "",
        "process_potentialstartergroup": "",
        "iseagerexecutionfetch": "false"
    },
    "childShapes": [
        {
            "resourceId": "startEvent1",
            "properties": {
                "overrideid": "",
                "name": "",
                "documentation": "",
                "executionlisteners": "",
                "initiator": "",
                "formkeydefinition": "",
                "formreference": "",
                "formfieldvalidation": true,
                "formproperties": ""
            },
            "stencil": {
                "id": "StartNoneEvent"
            },
            "childShapes": [],
            "outgoing": [
                {
                    "resourceId": "sid-B429C6BF-8E8E-4177-B467-ED56C573CADD"
                }
            ],
            "bounds": {
                "lowerRight": {
                    "x": 57.75,
                    "y": 196
                },
                "upperLeft": {
                    "x": 27.75,
                    "y": 166
                }
            },
            "dockers": []
        },
        {
            "resourceId": "sid-C966AFC1-22AB-4C87-91C0-27990443A3A8",
            "properties": {
                "overrideid": "",
                "name": "API动作测试",
                "documentation": "",
                "asynchronousdefinition": "false",
                "exclusivedefinition": "false",
                "executionlisteners": "",
                "multiinstance_type": "None",
                "multiinstance_variableaggregations": "",
                "multiinstance_cardinality": "",
                "multiinstance_collection": "",
                "multiinstance_variable": "",
                "multiinstance_condition": "",
                "isforcompensation": "false",
                "usertaskassignment": "",
                "formkeydefinition": "",
                "formreference": "",
                "formfieldvalidation": true,
                "duedatedefinition": "",
                "prioritydefinition": "",
                "formproperties": "",
                "tasklisteners": {
                    "taskListeners": [
                        {
                            "event": "create",
                            "implementation": "${actionCallListener}",
                            "className": "com.cdos.soar.flow.listener.ActionCallListener",
                            "expression": "",
                            "delegateExpression": "",
                            "$$hashKey": "uiGrid-0019",
                            "fields": []
                        }
                    ]
                },
                "skipexpression": "",
                "categorydefinition": "",
                "taskidvariablename": ""
            },
            "stencil": {
                "id": "UserTask"
            },
            "childShapes": [],
            "outgoing": [
                {
                    "resourceId": "sid-419EA6E4-F97F-4049-AD92-4A4CA7A76552"
                }
            ],
            "bounds": {
                "lowerRight": {
                    "x": 235,
                    "y": 221
                },
                "upperLeft": {
                    "x": 135,
                    "y": 141
                }
            },
            "dockers": []
        },
        {
            "resourceId": "sid-B429C6BF-8E8E-4177-B467-ED56C573CADD",
            "properties": {
                "overrideid": "",
                "name": "",
                "documentation": "",
                "conditionsequenceflow": "",
                "executionlisteners": "",
                "defaultflow": "false",
                "skipexpression": "",
                "showdiamondmarker": false
            },
            "stencil": {
                "id": "SequenceFlow"
            },
            "childShapes": [],
            "outgoing": [
                {
                    "resourceId": "sid-C966AFC1-22AB-4C87-91C0-27990443A3A8"
                }
            ],
            "bounds": {
                "lowerRight": {
                    "x": 134.5458984375,
                    "y": 181
                },
                "upperLeft": {
                    "x": 58.197265625,
                    "y": 181
                }
            },
            "dockers": [
                {
                    "x": 15,
                    "y": 15
                },
                {
                    "x": 50,
                    "y": 40
                }
            ],
            "target": {
                "resourceId": "sid-C966AFC1-22AB-4C87-91C0-27990443A3A8"
            }
        },
        {
            "resourceId": "sid-6531EF45-1CE5-430B-A63D-882CB655D320",
            "properties": {
                "overrideid": "",
                "name": "",
                "documentation": "",
                "asynchronousdefinition": "false",
                "exclusivedefinition": "false",
                "sequencefloworder": ""
            },
            "stencil": {
                "id": "InclusiveGateway"
            },
            "childShapes": [],
            "outgoing": [
                {
                    "resourceId": "sid-A8F54C85-279C-4CEF-BCA7-732D926ACE70"
                },
                {
                    "resourceId": "sid-01703820-455A-4809-BF67-7B43829D21D3"
                }
            ],
            "bounds": {
                "lowerRight": {
                    "x": 490,
                    "y": 199
                },
                "upperLeft": {
                    "x": 450,
                    "y": 159
                }
            },
            "dockers": []
        },
        {
            "resourceId": "sid-CA71B34F-71D2-4333-BB54-385FDA6B1A09",
            "properties": {
                "overrideid": "",
                "name": "脚本动作测试",
                "documentation": "",
                "asynchronousdefinition": "false",
                "exclusivedefinition": "false",
                "executionlisteners": "",
                "multiinstance_type": "None",
                "multiinstance_variableaggregations": "",
                "multiinstance_cardinality": "",
                "multiinstance_collection": "",
                "multiinstance_variable": "",
                "multiinstance_condition": "",
                "isforcompensation": "false",
                "usertaskassignment": "",
                "formkeydefinition": "",
                "formreference": "",
                "formfieldvalidation": true,
                "duedatedefinition": "",
                "prioritydefinition": "",
                "formproperties": "",
                "tasklisteners": {
                    "taskListeners": [
                        {
                            "event": "create",
                            "implementation": "${actionCallListener}",
                            "className": "com.cdos.soar.flow.listener.ActionCallListener",
                            "expression": "",
                            "delegateExpression": "",
                            "$$hashKey": "uiGrid-0033",
                            "fields": []
                        }
                    ]
                },
                "skipexpression": "",
                "categorydefinition": "",
                "taskidvariablename": ""
            },
            "stencil": {
                "id": "UserTask"
            },
            "childShapes": [],
            "outgoing": [
                {
                    "resourceId": "sid-EA7FDE84-A27E-4F29-A549-F13B431F86F6"
                }
            ],
            "bounds": {
                "lowerRight": {
                    "x": 699,
                    "y": 170
                },
                "upperLeft": {
                    "x": 599,
                    "y": 90
                }
            },
            "dockers": []
        },
        {
            "resourceId": "sid-A8F54C85-279C-4CEF-BCA7-732D926ACE70",
            "properties": {
                "overrideid": "",
                "name": "",
                "documentation": "",
                "conditionsequenceflow": {
                    "expression": {
                        "type": "static",
                        "staticValue": "${condition_result_sid_A8F54C85_279C_4CEF_BCA7_732D926ACE70}"
                    }
                },
                "executionlisteners": "",
                "defaultflow": "false",
                "skipexpression": "",
                "showdiamondmarker": true
            },
            "stencil": {
                "id": "SequenceFlow"
            },
            "childShapes": [],
            "outgoing": [
                {
                    "resourceId": "sid-CA71B34F-71D2-4333-BB54-385FDA6B1A09"
                }
            ],
            "bounds": {
                "lowerRight": {
                    "x": 598.5305067920871,
                    "y": 174.97886756104097
                },
                "upperLeft": {
                    "x": 486.80347758291293,
                    "y": 143.99574181395903
                }
            },
            "dockers": [
                {
                    "x": 20.5,
                    "y": 20.5
                },
                {
                    "x": 50,
                    "y": 40
                }
            ],
            "target": {
                "resourceId": "sid-CA71B34F-71D2-4333-BB54-385FDA6B1A09"
            }
        },
        {
            "resourceId": "sid-29059462-A089-49C5-9A58-639430ABCA93",
            "properties": {
                "overrideid": "",
                "name": "人工审批测试",
                "documentation": "",
                "asynchronousdefinition": "false",
                "exclusivedefinition": "false",
                "executionlisteners": "",
                "multiinstance_type": "Sequential",
                "multiinstance_variableaggregations": "",
                "multiinstance_cardinality": "",
                "multiinstance_collection": "",
                "multiinstance_variable": "",
                "multiinstance_condition": "",
                "usertaskassignment": {
                    "assignment": {
                        "type": "static",
                        "candidateUsers": [
                            {
                                "id": "1353264636949987330",
                                "name": "管理员",
                                "realName": "管理员",
                                "value": "1353264636949987330"
                            },
                            {
                                "id": "1483721810339041280",
                                "name": "操作员",
                                "realName": "操作员",
                                "value": "1483721810339041280"
                            }
                        ]
                    }
                },
                "isforcompensation": "false",
                "formkeydefinition": "",
                "formreference": "",
                "formfieldvalidation": true,
                "duedatedefinition": "",
                "prioritydefinition": "",
                "formproperties": "",
                "tasklisteners": {
                    "taskListeners": [
                        {
                            "event": "create",
                            "implementation": "${userApprovalListener}",
                            "className": "com.cdos.soar.flow.listener.UserApprovalListener",
                            "expression": "",
                            "delegateExpression": "",
                            "$$hashKey": "uiGrid-002H",
                            "fields": []
                        }
                    ]
                },
                "skipexpression": "",
                "categorydefinition": "",
                "taskidvariablename": ""
            },
            "stencil": {
                "id": "UserTask"
            },
            "childShapes": [],
            "outgoing": [
                {
                    "resourceId": "sid-8FF333E4-DDA3-4561-9D3A-3D3AE6F2AB15"
                }
            ],
            "bounds": {
                "lowerRight": {
                    "x": 699,
                    "y": 290
                },
                "upperLeft": {
                    "x": 599,
                    "y": 210
                }
            },
            "dockers": []
        },
        {
            "resourceId": "sid-01703820-455A-4809-BF67-7B43829D21D3",
            "properties": {
                "overrideid": "",
                "name": "",
                "documentation": "",
                "conditionsequenceflow": {
                    "expression": {
                        "type": "static",
                        "staticValue": "${condition_result_sid_01703820_455A_4809_BF67_7B43829D21D3}"
                    }
                },
                "executionlisteners": "",
                "defaultflow": "false",
                "skipexpression": "",
                "showdiamondmarker": true
            },
            "stencil": {
                "id": "SequenceFlow"
            },
            "childShapes": [],
            "outgoing": [
                {
                    "resourceId": "sid-29059462-A089-49C5-9A58-639430ABCA93"
                }
            ],
            "bounds": {
                "lowerRight": {
                    "x": 598.5640556649091,
                    "y": 230.07992114496412
                },
                "upperLeft": {
                    "x": 485.3753974600908,
                    "y": 185.37515698003588
                }
            },
            "dockers": [
                {
                    "x": 20.5,
                    "y": 20.5
                },
                {
                    "x": 50,
                    "y": 40
                }
            ],
            "target": {
                "resourceId": "sid-29059462-A089-49C5-9A58-639430ABCA93"
            }
        },
        {
            "resourceId": "sid-4E5FB9A6-1005-40EB-95E9-D827D984332B",
            "properties": {
                "overrideid": "",
                "name": "",
                "documentation": "",
                "asynchronousdefinition": "false",
                "exclusivedefinition": "false",
                "sequencefloworder": ""
            },
            "stencil": {
                "id": "InclusiveGateway"
            },
            "childShapes": [],
            "outgoing": [
                {
                    "resourceId": "sid-55168CC9-CBEB-48A7-BE28-D0D4E35FF3A9"
                }
            ],
            "bounds": {
                "lowerRight": {
                    "x": 813,
                    "y": 205
                },
                "upperLeft": {
                    "x": 773,
                    "y": 165
                }
            },
            "dockers": []
        },
        {
            "resourceId": "sid-EA7FDE84-A27E-4F29-A549-F13B431F86F6",
            "properties": {
                "overrideid": "",
                "name": "",
                "documentation": "",
                "conditionsequenceflow": "",
                "executionlisteners": "",
                "defaultflow": "false",
                "skipexpression": "",
                "showdiamondmarker": false
            },
            "stencil": {
                "id": "SequenceFlow"
            },
            "childShapes": [],
            "outgoing": [
                {
                    "resourceId": "sid-4E5FB9A6-1005-40EB-95E9-D827D984332B"
                }
            ],
            "bounds": {
                "lowerRight": {
                    "x": 778.5658209254499,
                    "y": 179.48694549235933
                },
                "upperLeft": {
                    "x": 699.4341790745501,
                    "y": 149.26305450764067
                }
            },
            "dockers": [
                {
                    "x": 50,
                    "y": 40
                },
                {
                    "x": 20,
                    "y": 20
                }
            ],
            "target": {
                "resourceId": "sid-4E5FB9A6-1005-40EB-95E9-D827D984332B"
            }
        },
        {
            "resourceId": "sid-8FF333E4-DDA3-4561-9D3A-3D3AE6F2AB15",
            "properties": {
                "overrideid": "",
                "name": "",
                "documentation": "",
                "conditionsequenceflow": "",
                "executionlisteners": "",
                "defaultflow": "false",
                "skipexpression": "",
                "showdiamondmarker": false
            },
            "stencil": {
                "id": "SequenceFlow"
            },
            "childShapes": [],
            "outgoing": [
                {
                    "resourceId": "sid-4E5FB9A6-1005-40EB-95E9-D827D984332B"
                }
            ],
            "bounds": {
                "lowerRight": {
                    "x": 778.5885528275983,
                    "y": 227.2448328735687
                },
                "upperLeft": {
                    "x": 699.4114471724017,
                    "y": 191.5051671264313
                }
            },
            "dockers": [
                {
                    "x": 50,
                    "y": 40
                },
                {
                    "x": 20,
                    "y": 20
                }
            ],
            "target": {
                "resourceId": "sid-4E5FB9A6-1005-40EB-95E9-D827D984332B"
            }
        },
        {
            "resourceId": "end",
            "properties": {
                "overrideid": "",
                "name": "",
                "documentation": "",
                "executionlisteners": ""
            },
            "stencil": {
                "id": "EndNoneEvent"
            },
            "childShapes": [],
            "outgoing": [],
            "bounds": {
                "lowerRight": {
                    "x": 916.5,
                    "y": 199
                },
                "upperLeft": {
                    "x": 888.5,
                    "y": 171
                }
            },
            "dockers": []
        },
        {
            "resourceId": "sid-55168CC9-CBEB-48A7-BE28-D0D4E35FF3A9",
            "properties": {
                "overrideid": "",
                "name": "",
                "documentation": "",
                "conditionsequenceflow": "",
                "executionlisteners": "",
                "defaultflow": "false",
                "skipexpression": "",
                "showdiamondmarker": false
            },
            "stencil": {
                "id": "SequenceFlow"
            },
            "childShapes": [],
            "outgoing": [
                {
                    "resourceId": "end"
                }
            ],
            "bounds": {
                "lowerRight": {
                    "x": 887.8750105208338,
                    "y": 185.4094753922974
                },
                "upperLeft": {
                    "x": 813.2343644791662,
                    "y": 185.0670871077026
                }
            },
            "dockers": [
                {
                    "x": 20.5,
                    "y": 20.5
                },
                {
                    "x": 14,
                    "y": 14
                }
            ],
            "target": {
                "resourceId": "end"
            }
        },
        {
            "resourceId": "sid-AD74E13A-56F8-4264-9715-A94890A90C98",
            "properties": {
                "overrideid": "",
                "name": "条件判断虚拟节点",
                "documentation": "",
                "asynchronousdefinition": "false",
                "exclusivedefinition": "false",
                "executionlisteners": "",
                "multiinstance_type": "None",
                "multiinstance_variableaggregations": "",
                "multiinstance_cardinality": "",
                "multiinstance_collection": "",
                "multiinstance_variable": "",
                "multiinstance_condition": "",
                "isforcompensation": "false",
                "usertaskassignment": "",
                "formkeydefinition": "",
                "formreference": "",
                "formfieldvalidation": true,
                "duedatedefinition": "",
                "prioritydefinition": "",
                "formproperties": "",
                "tasklisteners": {
                    "taskListeners": [
                        {
                            "event": "create",
                            "implementation": "${conditionCallListener}",
                            "className": "com.cdos.soar.flow.listener.ConditionCallListener",
                            "expression": "",
                            "delegateExpression": "",
                            "$$hashKey": "uiGrid-001V",
                            "fields": []
                        }
                    ]
                },
                "skipexpression": "",
                "categorydefinition": "",
                "taskidvariablename": ""
            },
            "stencil": {
                "id": "UserTask"
            },
            "childShapes": [],
            "outgoing": [
                {
                    "resourceId": "sid-5D71C8F7-A6DB-4AC5-81F3-85E9D7FB3F71"
                }
            ],
            "bounds": {
                "lowerRight": {
                    "x": 400,
                    "y": 225
                },
                "upperLeft": {
                    "x": 300,
                    "y": 145
                }
            },
            "dockers": []
        },
        {
            "resourceId": "sid-419EA6E4-F97F-4049-AD92-4A4CA7A76552",
            "properties": {
                "overrideid": "",
                "name": "",
                "documentation": "",
                "conditionsequenceflow": "",
                "executionlisteners": "",
                "defaultflow": "false",
                "skipexpression": "",
                "showdiamondmarker": false
            },
            "stencil": {
                "id": "SequenceFlow"
            },
            "childShapes": [],
            "outgoing": [
                {
                    "resourceId": "sid-AD74E13A-56F8-4264-9715-A94890A90C98"
                }
            ],
            "bounds": {
                "lowerRight": {
                    "x": 299.3713874681104,
                    "y": 183.77263969619662
                },
                "upperLeft": {
                    "x": 235.62861253188962,
                    "y": 182.22736030380338
                }
            },
            "dockers": [
                {
                    "x": 50,
                    "y": 40
                },
                {
                    "x": 50,
                    "y": 40
                }
            ],
            "target": {
                "resourceId": "sid-AD74E13A-56F8-4264-9715-A94890A90C98"
            }
        },
        {
            "resourceId": "sid-5D71C8F7-A6DB-4AC5-81F3-85E9D7FB3F71",
            "properties": {
                "overrideid": "",
                "name": "",
                "documentation": "",
                "conditionsequenceflow": "",
                "executionlisteners": "",
                "defaultflow": "false",
                "skipexpression": "",
                "showdiamondmarker": false
            },
            "stencil": {
                "id": "SequenceFlow"
            },
            "childShapes": [],
            "outgoing": [
                {
                    "resourceId": "sid-6531EF45-1CE5-430B-A63D-882CB655D320"
                }
            ],
            "bounds": {
                "lowerRight": {
                    "x": 450.25124766112214,
                    "y": 182.46568738305612
                },
                "upperLeft": {
                    "x": 400.68625233887786,
                    "y": 179.98743761694388
                }
            },
            "dockers": [
                {
                    "x": 50,
                    "y": 40
                },
                {
                    "x": 20,
                    "y": 20
                }
            ],
            "target": {
                "resourceId": "sid-6531EF45-1CE5-430B-A63D-882CB655D320"
            }
        }
    ],
    "stencil": {
        "id": "BPMNDiagram"
    },
    "stencilset": {
        "namespace": "http://b3mn.org/stencilset/bpmn2.0#",
        "url": "../editor/stencilsets/bpmn2.0/bpmn2.0.json"
    }
}
```


# 包容网关报错问题


## 问题描述

![[../../../14 - 静态资源/Pasted image 20230214145943.png]]

demo 用这个流程

当上述 4 个节点都是用 create 的监听器，并在监听器里调用 taskService.complete 时，执行到最后一个包容网关会报错，说是包容网关的方法里，在获取一个执行实例的 activeId 并和其他对象做 equals 时，获取到 null，报空指针异常

报错内容如下


```
Caused by: org.flowable.common.engine.api.FlowableException: Exception while invoking TaskListener: Exception while invoking TaskListener: Exception while invoking TaskListener: Exception during command execution
...
Caused by: org.flowable.common.engine.api.FlowableException: Exception while invoking TaskListener: Exception while invoking TaskListener: Exception during command execution
	at com.cdos.soar.flow.listener.ActionCallListener.notify(ActionCallListener.java:76)
Caused by: org.flowable.common.engine.api.FlowableException: Exception while invoking TaskListener: Exception during command execution
...
Caused by: org.flowable.common.engine.api.FlowableException: Exception during command execution
	at org.flowable.common.engine.impl.interceptor.CommandContextInterceptor.execute(CommandContextInterceptor.java:105)
	at org.flowable.common.spring.SpringTransactionInterceptor.execute(SpringTransactionInterceptor.java:51)
	at org.flowable.common.engine.impl.interceptor.LogInterceptor.execute(LogInterceptor.java:30)
	at org.flowable.common.engine.impl.cfg.CommandExecutorImpl.execute(CommandExecutorImpl.java:56)
	at org.flowable.common.engine.impl.cfg.CommandExecutorImpl.execute(CommandExecutorImpl.java:51)
	at org.flowable.engine.impl.TaskServiceImpl.complete(TaskServiceImpl.java:208)
	at com.cdos.soar.flow.listener.ConditionCallListener.notify(ConditionCallListener.java:35)
Caused by: java.lang.NullPointerException: null
	at org.flowable.engine.impl.bpmn.behavior.InclusiveGatewayActivityBehavior.executeInclusiveGatewayLogic(InclusiveGatewayActivityBehavior.java:72)
```

情况一：如果把两个网关都换成并行网关，就没有问题
情况二：如果把最后一个包容网关去掉，也没问题
情况三：如果让 “人工审批测试” 节点的监听器不执行 taskService.complete，而是让人员审批时，模拟调用审批接口时再执行 taskService.complete，也不会报错

## 猜想和验证

猜想一：

看完包容网关代码实现后，发现会不会是这样的原因才报错的

执行实例1，执行实例2并行执行两个动作，执行实例1先执行完动作

执行实例1到达包容网关后，先获取到所有的执行实例（1和2），在执行实例1检查其他执行实例所在节点能否也能到达包容网关前等待

执行实例2执行完动作后，执行实例1检查其他执行实例能不能到达网关（或执行实例2去检查）。结果有一个执行实例已经停了，处于 inactive，并且没有 activityId，导致上述异常

猜测报错原因是因为用 debug方式运行代码。本应该并行执行并几乎同时完成动作的两个执行实例，因为 debug，两个执行实例同时执行监听器时，coder 同一时刻只能在debug界面执行一个动作并完成任务

验证方案一：

出现上述报错原因是有一个执行实例的 activityId 为 null



## 定位问题



## 解决方案


# Demo 用的流程 - JavaDelegate + 监听器实现

https://blog.csdn.net/u012702547/article/details/127513360

# 补充

在定位[[24 - Demo 功能验证#包容网关报错问题|包容网关报错]]问题时，看到在 `InclusiveGatewayActivityBehavior` 里遇到空指针后，跳转到 `ContinueProcessOperation` ，这里有一个

```java
try {  
    activityBehavior.execute(execution);  
} catch (RuntimeException e) {  
    if (LogMDC.isMDCEnabled()) {  
        LogMDC.putMDCExecution(execution);  
    }  
    throw e;  
}
```

MDC