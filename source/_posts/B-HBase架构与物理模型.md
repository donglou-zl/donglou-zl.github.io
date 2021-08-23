---
title: B - HBase架构与物理模型
date: 2017-10-24 19:24:06
tags:
	- HBase
---
HBase可在大规模廉价的PC机器上搭建集群，其利用Hadoop的HDFS作为底层文件存储系统，利用MapReduce进行海量数据的分析，并使用ZooKeeper作为协同服务。<!--more-->

![HBase架构图][1]

1、HBase整体架构

Client：客户端，是HBase的入口，使用HBase的RPC机制与HMaster和HRegionServer进行通信。与HMaster进行通信，负责管理类的操作；与HRegionServer进行通信，负责数据读写类操作。

ZooKeeper：分布式应用程序协调服务组件，负责管理多个HMaster的选举、存储HBase元数据信息（HMaster启动时将系统表-ROOT-加载到zk）、实时监控HRegionServer状态信息、存储所有Region的寻址入口等。

HMaster：主节点，可以在启动HBase时设置为“启动多个”，但会通过选举机制保证只有一个用于提供服务。其主要负责管理Table的CRUD操作，管理RegionServer的负载均衡，Region的重分配及迁移等。

HRegionServer：主要负责用户的I/O请求，向HDFS（Hadoop的分布式文件系统）中读写数据（HBase底层数据存储依靠于HDFS）。其包含一个HLog（实现Write ahead log，用户每次写入操作前，会写一份数据到HLog文件中）和多个HRegion。当前，每个HRegionServer可以管理大约100~1000个HRegion，每个HRegion大小为1~20GB。

HRegion：存储的是实际数据，由一个或者多个HStore组成。每个table最开始只有一个HRegion，随着HRegion不断增大，当增大到一个阈值时，分割为两个新的HRegion。最后一个table会按行分割为多个HRegion，每个HRegion分散在不同的HregionServer中。HRegion是分布式存储和负载的最小单元（注意，不是存储的最小单元），即一个HRegion不会拆分到多个Server上。

HStore：每个HStore保存了Table中的一个ColumnFamily（列族，即将经常同时访问的字段放在一起）的存储。每个HStore包含了一个MemStore和至少一个StoreFile。MemStore是Sorted Memory Buffer，用户写入数据时会首先放入MemStore中，当它满了以后，会缓冲（flush）成一个StoreFile。

StoreFile：包含一个或者多个HFile，存在着Compact（合并）与Split（分裂）操作。

HFile：结构分为6部分，主要包括，
> [a] Data Block: 存储表中的数据，该部分数据可以被压缩。Data block是由多个block组成，每个block的组成形式为“块头+key长度+value长度+key+value”。块的大小可以设置，默认是64KB。如果设置较大，利于scan；设置较小，利于随机查询。
[b] Meta Block（optional）：元数据块，用于保存元数据是kv类型的值。（注意，该数据块中只保存value值，key的值保存在第五项）。
[c] FileInfo：HFile的元信息，保存的数据是以key值排序的kv类型的值。
[d] DataIndex：Data block的索引，保存的是每个数据块在HFile中的位置、大小以及每个块的第一条key，即被索引的block中第一条记录的key。
[e] MetaBlock Index：Meta block的索引，保存的是每个元数据在HFile中的位置、大小及每个元数据的key值。
[f] Trailer Block：保存了HFile的一些基本信息，比如FileInfo的偏移、DataIndex块的个数等。

2、HBase表物理模型

![HBase表物理模型][2]

HBase表的特征之一：多维有序映射。

> RowKey -> 某一行数据;
Family -> 某行数据中的某个列族；
Qualifier -> 某行数据中某个列族的某一列；
Timestamp -> 某行数据中某个列族的某一列（单元格数据）的某一个版本



  [1]: http://oybpgm6jn.bkt.clouddn.com/image/hexoBlog/HBase%E6%9E%B6%E6%9E%84%E5%9B%BE.png
  [2]: http://oybpgm6jn.bkt.clouddn.com/HBase%E7%89%A9%E7%90%86%E6%A8%A1%E5%9E%8B.jpg