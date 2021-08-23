---
title: 1 - HBase源码分析-基本情况
date: 2017-10-28 11:21:05
tags:
    - HBase
    - Hbase源码分析系列
---

从本节起，将开启HBase的源码分析系列，分析的内容主要包括但不限于以下：
<!--more-->
> 
查询数据流程
插入数据流程
扫描数据流程
删除数据流程Zookeeper启动过程
HMaster代码结构
HRegionServer代码结构
HMaster启动过程
RegionServer启动过程
HMaster与RegionServer通信过程
Compact/Split过程
LSM数据模型
HFile格式
HLog格式
WAL
Lock (RowLock)
Filter实例，作用范围
Filter集合，对应类
以及一些设计模式或者其他亮点

分析的HBase源码为[github下载][1]，主要针对HBase client、server、common、protocol、replication等直接相关的代码进行分析，其他对第三方的集成支持的代码则先不考虑。


  [1]: https://github.com/apache/hbase/