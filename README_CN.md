Name
====

openwaf_anti_cc是[OpwnWAF](https://github.com/titansec/openwaf)项目的CC（HTTP FLOOD）防护模块

Table of Contents
=================
* [Name](#name)
* [Synopsis](#synopsis)
* [Modules Configuration Directives](#modules-configuration-directives)

Synopsis
========

[Back to TOC](#table-of-contents)

Modules Configuration Directives
================================

```txt
{
    "twaf_limit_conn": {
        "state":false,                                       -- CC防护模块开关
        "log_state":true,                                    -- CC日志开关
        "trigger_state":true,                                -- 触发开关
        "clean_state":true,                                  -- 清洗开关
        "trigger_thr":{                                      -- 触发阈值（关系为“或”）
            "req_flow_max":1073741824,                       -- 每秒请求流量，单位B
            "req_count_max":10000                            -- 每秒请求数
        },
        "clean_thr":{                                        -- 清洗阈值
            "new_conn_max":40,                               -- 单一源ip每秒新建连接数
            "conn_max":100,                                  -- 单一源ip防护期间内连接总数
            "req_max":50,                                    -- 单一源ip每秒请求总数
            "uri_frequency_max":3000                         -- 单一路径每秒请求总数
        },
        "event_id":"710002",                                 -- 事件ID
        "event_severity":"high",                             -- 事件严重等级
        "timer_flush_expired":10,                            -- 清理shared_dict过期数据的时间间隔
        "interval":10,                                       -- 进入CC防护后发送日志的时间间隔，单位秒
        "shared_dict_name":"twaf_limit_conn",                -- 存放其他信息的shared_dict
        "shared_dict_key": "remote_addr",                    -- shared_dict的键值
        "action":"DENY",                                     -- 触发CC防护执行的动作
        "action_meta":403,
        "timeout":30                                         -- 清洗时长（当再次触发清洗值时，重置）
    }
}
```

###rules
**syntax:** *"state": true|false*

**default:** *false*

**context:** *twaf_limit_conn*

boolean类型，CC防护模块总开关，默认关闭

###log_state
**syntax:** *"log_state": true|false*

**default:** *true*

**context:** *twaf_limit_conn*

boolean类型，CC防护模块日志开关，默认开启

###trigger_state
**syntax:** *"trigger_state": true|false*

**default:** *true*

**context:** *twaf_limit_conn*

boolean类型，CC防护模块的触发开关，默认开启

若关闭，则触发机制关闭，时刻进入CC清洗状态

###clean_state
**syntax:** *"clean_state": true|false*

**default:** *true*

**context:** *twaf_limit_conn*

boolean类型，CC防护模块总开关，默认开启

若关闭（仅用于测试），则清洗机制关闭，CC模块将无法拦截请求

###trigger_thr
**syntax:** *"trigger_thr": table*

**default:** *{"req_flow_max":1073741824,"req_count_max":10000}*

**context:** *twaf_limit_conn*

table类型，触发阈值

当达到其中一个触发阈值，进入CC清洗状态

当前有两个触发阈值  
```txt
    "trigger_thr":{                                      -- 触发阈值（关系为“或”）
        "req_flow_max":1073741824,                       -- 每秒请求流量，单位B，默认1GB/s
        "req_count_max":10000                            -- 每秒请求数，默认10000个/秒
    }
```

###clean_thr
**syntax:** *"clean_thr": table*

**default:** *{"new_conn_max":40,"conn_max":100,"req_max":50,"uri_frequency_max":3000}*

**context:** *twaf_limit_conn*

table类型，清洗阈值

当进入CC清洗状态，达到其中一个清洗阈值，则执行相应动作

当前有四个清洗阈值  
```txt
    "clean_thr":{                                        -- 清洗阈值（关系为“或”）
        "new_conn_max":40,                               -- 单一源ip每秒新建连接数，默认40个/秒
        "conn_max":100,                                  -- 单一源ip防护期间内连接总数，默认100个
        "req_max":50,                                    -- 单一源ip每秒请求总数，默认50个/秒
        "uri_frequency_max":3000                         -- 单一路径每秒请求总数，默认3000个/秒
    }
```

###timer_flush_expired
**syntax:** *"timer_flush_expired": number*

**default:** *10*

**context:** *twaf_limit_conn*

number类型，清理shared_dict过期数据的时间间隔，默认10秒

###interval
**syntax:** *"interval": number*

**default:** *10*

**context:** *twaf_limit_conn*

number类型，进入CC防护后发送日志的时间间隔，默认10秒

###shared_dict_name
**syntax:** *"shared_dict_name": string*

**default:** *"twaf_limit_conn"*

**context:** *twaf_limit_conn*

string类型，存放当前CC防护信息的shared_dict名称

###shared_dict_key
**syntax:** *"shared_dict_key": string|table*

**default:** *"remote_addr"*

**context:** *twaf_limit_conn*

string或table类型，shared_dict的键值，支持nginx变量

支持字符串类型和数组类型
```
    "shared_dict_key": "remote_addr"
    
    "shared_dict_key": ["remote_addr", "http_user_agent"]
```

###action
**syntax:** *"action": string*

**default:** *"DENY"*

**context:** *twaf_limit_conn*

string类型，触发CC防护执行的动作，默认"DENY"

###action_meta
**syntax:** *"action_meta": number|string*

**default:** *403*

**context:** *twaf_limit_conn*

string或number类型，执行动作的附属信息，默认403

###timeout
**syntax:** *"timeout": number*

**default:** *30*

**context:** *twaf_limit_conn*

number类型，清洗时长，N秒内不再达到触发阈值，则退出CC清洗状态

在清洗过程中，再次达到触发阈值，则时间重置为30秒
