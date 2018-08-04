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

- nohup hive --service metastore > metastore.log 2>&1 &

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
            - 就是不同的hdfs文件夹
            - 动态分区：根据某一个字段，自动的生成分区字段的内容
                - 不能通过LOAD加载数据，需要INSERT
                - 不能过多使用（10万以下）
            - show partitions table_name;
        - 桶表
            - 对数据表字段做hash散列，相当于提供了一个索引
            - 根据索引，分配到不同的桶中，将一个字段分成N个桶
            - 可以用数据采用的方法，只取桶里面的某一部分数据
            - 开启分桶开关
                - set hive.enforce.bucketing = true;
            - 设置reduce个数，与桶一致
                - set mapreduce.job.reduces = 4;
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

分区和桶的使用场景
- 用户日志数据，每天都产生，需要保存，用来产生报表
    - 需要做分区，不需要做桶
- 用户的明细数据，50万条，只用来做join
    - 不需要做分区
- 每一次都使用年龄范围来检索
    - 将年龄做成桶

***

<h4 id='4'>第四节 HiveHQL</h4>

1. 了解Hive HQL语法
2. 掌握Hive HQL编写方法
3. 学会使用HQL进行分析
4. 学习使用HQL DML语句

---

HQL语法
- SELECT
    - SELECT [ALL|DISTINCT] col1, col2, ... FROM table_name 
    [WHERE where_condition]
    [GROUP BY col_list [HAVING condition]]
    [CLUSTER BY col_list | [DISTRIBUTE BY col_list] [SORT BY| ORDR BY col_list]]
    [LIMIT number]
    - 简单的SELECT不会触发MapReduce，只是扫描了一下文件，使用order等操作会触发MapReduce
    - order是全局有序，sort只让reduce本身有序，当reduce>1时，sort不保证全部reduce内的数据有统一的排序
- JOIN
    - 只支持等值连接，即ON子句中使用等号连接
    - 先执行JOIN，再执行WHERE
    - 可以JOIN多个表
    - JOIN对MapReduce的开销比较大
    - left join
    - right join
    - full join
    - *left semi join*：相当于数据库的in操作
- IN
    - left semi join = in / exists
    - HIVE中，支持的子查询嵌套程度有限（3-7层）
    - NOT IN占用性能
- LIKE
    - 可以用LIKE创建表
        - CREATE TABLE new_tbl liek old_tbl;
    - 在map端实现，不触发MapReduce

DML语句
- LOAD：向数据表内加载文件
    - LOAD DATA [LOCAL] PATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (part_col=val, ...)]
        - 不校验数据是否存在，需要确认是否保留原表数据，使用into/overwrite
        - 不对数据做任何操作
        - 支持本地与HDFS文件/文件夹
- INSERT：将查询结果插入Hive表
    - 基本模式
        - INSERT OVERWRITE TABLE table1 [PARTITION (part_col=val, ...)] select_statement FROM from_statement
    - 多插入模式
        - FROM from_statement 
        INSERT OVERWRITE TABLE table1 [PARTITION (part_col=val, ...)] select_statement1 
        [INSERT OVERWRITE TABLE table2 [PARTITION (part_col=val, ...)] select_statement2] 
        ...
        - 只扫描数据表一次，可以将数据写入多个表，提升效率
        - 原理：MapReduce的context可以执行多次
    - 自动分区模式
        - INSERT OVERWRITE TABLE tablename PARTITION (part_col=[val], ...) select_statement FROM from_statement
    - insert into table tablename select ...
    - insert overwrite table tablename select ...
    - insert into table tablename values(val,...)
    - 注：如果包含分区，则select_statement的最后一个字段是针对分区的
- INSERT：将查询结果写入HDFS
    - INSERT OVERWRITE [LOCAL] DIRECTORY dir1 SELECT ... FROM ... FROM from_statement
    - 写入文件系统时，会进行文本序列化，每列用“^A”来区分，\n换行

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