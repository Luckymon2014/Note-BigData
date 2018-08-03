# 目录 #

-[第一节 Hive系统概述](#1)
-[第二节 Hive系统安装及配置](#2)
-[第三节 Hive数据模型](#3)
-[第四节 HiveHQL](#4)
-[第五节 Hive常见函数](#5)
-[第六节 Hive自定义函数](#6)
-[第七节 Hive2.0存储过程实践](#7)
-[第八节 Hive Index原理及使用](#8)
-[第九节 Hive Update,Delete操作说明](#9)
-[第十节 Hive ORCFile,Parquet 文件格式实践](#10)
-[第十一节 Hive 数据压缩及解决数据倾斜](#11)
-[第十二节 Hive JDBC实践V1](#12)
Hive微博分析大数据平台数据仓库设计
Hive实现KOL分析
Hive实现声量分析
Hive实现微博情感分析
Hive实现用户微博热度分析
Hive微博发帖终端分析

***

<h4 id='1'>第一节 Hive系统概述</h4>

1. 了解Hive发展简史
2. 掌握Hive主要版本
3. 了解Hive使用场景
4. 了解Hive2.0特性

---

Hive起源和发展
- 起源于Facebook，为分析师（熟悉SQL）开发的类SQL语言，用于操作Hadoop的MapReduce
- Hive是数据仓库
- Hive是构建在Hadoop之上的，所有数据都是存储在HDFS中
- Hive的执行延迟高，不适合做实时的数据分析，适合做离线分析
- Hive Apache最新版本为2.3.0

Hive应用场景与定位
- 离线分析，高延迟
- 数据仓库

Hive功能架构
- Hive
    - Client
        - CLI（命令行）/JDBC → 访问Driver
        - HiveServer2：处理jdbc请求
        - Driver：SQL解析器 → 转换HQL语言给MR执行
            - SQL Parser
            - Query Optimize：优化
            - Physical Plan：生成物理执行计划
            - Execution
    - Meta Store：存储Hive的元数据（MySql等第三方数据库）
- MapReduce
- HDFS

```mermaid
graph LR

subgraph 
    CLI
    JDBC/ODBC
    WebUI
end

USER-->|Hive SQL|CLI
USER-->JDBC/ODBC
USER-->WebUI

subgraph Hive
    HiveServer2
    Hive
    Compiler
    Optimizer
    Executor
end

CLI-->Hive
JDBC/ODBC-->HiveServer2
HiveServer2-->Hive
Hive-->Compiler
Compiler-->Optimizer
Optimizer-->Executor

subgraph Hadoop
    NameNode
    DataNode
    JobTracker
    TaskTracker
end

Executor-->|MapReduce|JobTracker
```

Hive与关系型数据库的差别
- 不支持实时数据的写入与复杂事务
- 不支持触发器
- 不支持主外键
- 使用Hadoop与Spark作为计算引擎

Hive2.0特性
- 存储过程HPLSQL
- 低延迟分析处理Low Latency Analytical Processing(LLAP)
    - 也可称作Live Long and Process
    - 是一个常驻内存的进程，可以将Hive的计算代入内存，即时执行SQL
    - 能有效的提升Hive的分析能力
- HBase MetaStore
    - Hive的数据存储在MetaStore中
    - 当前技术不够成熟，不建议使用
- Hive On Spark Improvement
    - Spark：Tez、MapReduce
    - Hive2.0开始支持Spark执行引擎，解决MapReduce问题
    - set hive.execution.engine=spark;
    - 版本对应关系

    Hive Version|Spark Version
    ------------|-------------
    master|2.2.0
    2.3.x|2.0.0
    2.2.x|1.6.0
    2.0.x|1.5.0
    1.2.x|1.3.1
    1.1.x|1.2.0

    - CDH Hadoop，Cloudera Hadoop的Hive默认使用Spark作为引擎，Apache Hadoop需要进行配置

Hive版本
- CDH：hive-1.1.0+cdh5.14.0+1330
- Apache Hive 2.2.0

***

<h4 id='2'>第二节 Hive系统安装及配置</h4>

1. 掌握Hive安装
2. 掌握Hive安装注意事项

---

Hive安装过程
1. Hive、MySQL软件下载
2. 安装环境监测：Hadoop存在
3. MySQL安装与监测
4. 配置环境变量
5. Hive命令测试及开发环境伪分布式测试

***

<h4 id='3'>第三节 Hive数据模型</h4>

1. 掌握Hive数据模型使用
2. 掌握Hive数据模型使用场景

---

Hive字段类型
- 原始类型
    - 布尔型：BOOLEAN
    - 整型：TINYINT、SMALLINT、INT、BIGINT
    - 浮点型：FLOAT、DOUBLE、DECIMAL
    - 字符串：STRING、VARCHAR、CHAR
    - 字节：BINARY
    - 日期：TIMESTAMP、DATE(只精确到日)
- 复杂类型
    - ARRAY：有序的同类型集合
    - MAP：键值对，key必须为原始类型，value可以是任意类型
    - STRUCT：字段集合，类型可以不同
        - 利用Map、Struct类型可以避免外键的使用，使一个字段可以存储多个值
        - 弊端是，更新数据复杂，存在数据冗余
    - UNION：有限取值范围内的一个值

Hive数据模型
- Database
    - 与数据库概念一致
    - Schema = Database
    - CREATE (Database|Schema) [IF NOT EXISTS] database_name [COMMENT database_comment] [LOCATION hdfs_path] [WITH DBPROPERTIES (property_name=value, ...)];
    - DROP (Database|Schema) [IF EXISTS] database_name [RESTRICT|CASECADE];
    - USE database_name; USE DEFAULT;
        - 提供默认数据库，表默认放在DEFAULT数据库中
- Table
    - 与数据库Table概念一致
    - 主要分类
        - 内部表：删除时，仅删除数据，保留索引
        - 外部表：删除时，仅删除索引，保留数据
            - 生产通常使用外部表，因为数据得保留在HDFS上
            - 内、外部表都有分区模式与桶模式
        - 分区表
            - 作用：减少数据被扫描的量
            - 特点：不能使用建表的字段
        - 桶表
        - 临时表：只对当前session有效
    - CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name [(col_name data_type [COMMENT col_comment], ... [constraint_specification])] [COMMENT table_comment] 
    [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
    [CLUSTERED BY (col_name, ...)]
    [SORTED BY (col_name [ASC|DESC], ...)]
    [SKEWED BY (col_name, ...) 
        ON ((col_value, ...), (col_value, ...), ...) [STORED AS DIRECTORIES]]
    [
        [ROW FORMAT row_format] //分隔符
        [STORED AS file_format] | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]
    ]
    [LOCATION hdfs_path]
    [TBLPRPERTIES (property_name=value, ...)]
- View
    - 与数据库View概念一致
    - CREATE VIEW [IF NOT EXISTS] [db_name.]view_name [(column_name [COMMENT column_comment], ...)] [COMMENT view_comment] [TBLPROPERTIES (property_name=value, ...)] AS SELECT ...;
    - DROP VIEW [IF EXISTS] [db_name.]view_name;




***

<h4 id='4'>第四节 HiveHQL</h4>

***

<h4 id='5'>第五节 Hive常见函数</h4>

***

<h4 id='6'>第六节 Hive自定义函数</h4>

***

<h4 id='7'>第七节 Hive2.0存储过程实践</h4>

***

<h4 id='8'>第八节 Hive Index原理及使用</h4>

***

<h4 id='9'>第九节 Hive Update,Delete操作说明</h4>

***

<h4 id='10'>第十节 Hive ORCFile,Parquet 文件格式实践</h4>

***

<h4 id='11'>第十一节 Hive 数据压缩及解决数据倾斜</h4>

***

<h4 id='12'>第十二节 Hive JDBC实践V1</h4>