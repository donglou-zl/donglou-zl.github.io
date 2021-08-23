---
title: 2 - HBase源码分析-数据查询与Get
date: 2017-10-31 21:42:58
tags:
    - HBase
    - Hbase源码分析系列
---

使用这种方法用于从hbase中获取单行数据：
　　

    Result get(Get get) throws IOException

首先我们来看下Result的结构，
　　

    Result {
       Cell[] cells;
       Boolean exist;//if the query was just to check existence
       ...
    }
Result用于存储Get或者Scan操作后返回的表的单行值，使用Result可以直接获取值或者各种Map结构（Key-Value对），例如：<!--more-->

    /**
    * Method for retrieving(检索) the row key that corresponds to
    * the row from which this Result was created.
    **/
    public byte[] getRow(){
        ...
    }
    
    /**
    * Map of qualifiers to values.
    * form: <qualifier, value>
    * 指定的列族内部的列与值的映射
    **/
    public NavigableMap<byte[], byte[]> getFamilyMap(byte [] family){
        ...
    }

接着，我们简单看下Result内部最重要的Cell数组。Cell是HBase中最基本的存储单元，包含很多的信息，比如：row、column family、type、mvcc version等。其唯一性由：坐标四元素<行键、列族名、列限定符、时间戳>+type 共计5者组合而定。前四者大家都清楚，**type**是指（写）操作类型，比如“put”，“delete”，读操作则没有。

    interface Cell{
        byte[] getRowArray();
        int getRowOffset();
        int getRowLength();
        ...
    }
另外几个不常见的信息，比如：
**mvcc version**，多版本并发控制，其目的是在保证数据一致性的前提下，提供一种高并发的访问性能。
**tag**，非必需的，可能一个cell中有很多tags。
放一张Table-Cell的基本结构图（图片来自网络）

![HBase Table结构图][1]

单行获取数据时，每次RPC请求都会发送一个Get对象。因为Get对象初始化需要输入行键，因此可以理解为一个Get对象就代表一行数据，包含多个列族或者多个列的信息。

    Get extends Query implements Row, Comparable<Row>{
        /**
        * create a Get operation for the specified row
        * 如果没有其他操作设置，则默认为获取所有列数据的最新版本
        **/
        public Get(byte[] row) {...}
    }

当然，用户可以有很多种方式来筛选目标数据，精确地获取某个单元格的数据：

    Get addFamily(byte[] family);//取得指定列族的数据
    Get addColumn(byte[] column, byte[] qualifier);//取得指定那一列的数据
    ...

一个完整的单行数据获取过程：

    Configuration conf = HBaseConfiguration.create();//参数配置
    HTable table = new HTable(conf, "test");//初始化一个新的表引用
    Get get = new Get(Bytes.toBytes("row1"));//使用指定行键“row1”构建一个Get实例
    get.addColumn(Bytes.toBytes("colfam1"), Bytes.toBytes("qua1"));//向Get实例中添加一个列的限定
    Result rst = table.get(get);//从HBase中返回数据
    byte[] val = result.getValue(Bytes.toBytes("colfam1"), Bytes.toBytes("qua1"));//从返回的结果中获取对应列的数据（默认获取最新版本）
    System.out.println("value: " + val);

  [1]: http://oybpgm6jn.bkt.clouddn.com/image/hexoBlog/HBase%20Table%E7%BB%93%E6%9E%84%E5%9B%BE.jpg