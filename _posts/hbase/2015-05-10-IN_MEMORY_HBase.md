---
layout: post
title: HBase 使用IN_MEMORY=>True建表时遇到的坑
category: Hbase
tags: [IN_MEMORY， HBase]
---

---


## 遇到的问题
    线上的一个业务在后端用了HBase作为存储方案。在上线了一段时间后，业务前端反应读取数据库较以往慢很多，并添加了一些内部监控【查询各子阶段的延时监控】的同比，环比延时对比，发现确实比之前查询速度慢了很多。接下来就是定位问题了。
    
1. 在此故障期间，发现有一台regionserver上不断的在进行minor compacting 操作，并且写入很不均衡,即数据倾斜的历害。（这主要是因为有个别子业务的数据量较大，且为了考虑大多数业务前端的查询需求而导致的【各个业务的查询方式都是统一的，不区分数据量的问题】）因此猜测有可能是不断compacting影响了读的性能[后来发现并不是这个原因引起的，但写不均衡的问题一直存在]。
2. IN-MEMORY 表
    检查日志一直没有找到问题的源头【读性能下降的比较历害】，在接着检查storm topo[用来做断链分析]的时候发现有一个表的IN-Memory属性设置的是为true. 在当时主要是考虑到这个表较小【指标数据集数目相对固定】，且读较频繁。然而在实际的应用中，使用了表版本的特性进行了去重， 表最大版本属性设置：MAXVERSION => 1.[实际的操作过程中间涉及到了大量的写，不断的覆盖前一个版本的值。]，随着topo运行时间越长，表的不断写入，性能影响也越明显，kill 掉topo后，恢复正常。
    

---
Polish IN_MEMORY table behavior

## IN-MEMORY 属性

For a long time we've been able to support a mode of operation that keeps as much table data as possible in memory, so HBase can be used as an 'in-memory' DB with fully durable WAL and write-behind persistence of table data. However:
There are a set of relevant schema options (IN_MEMORY, CACHE_ON_WRITE, PREFETCH_BLOCKS_ON_OPEN, block encoding), so set up isn't simple. We should have a shortcut that sets all this up in one place. I'm thinking a utility class with static helpers that configure a table descriptor with all of the needed bits. (Other ideas?)
We don't have a safety valve. An in-memory table can become too large, where it falls out of blockcache and performs poorly without warning because it's become too big. Consider table quota support with an option for region size limits as % of total heap consumed by regions for a given table. Warn at soft limit. Refuse writes if over hard limit.
Follow on work can investigate options hooking up to offheap work. That's not in scope here.

```
https://issues.apache.org/jira/browse/HBASE-13472
https://issues.apache.org/jira/browse/HBASE-13408
```