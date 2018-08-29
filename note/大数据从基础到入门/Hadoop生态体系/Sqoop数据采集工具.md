# 目录 #

- [第一节 Sqoop系统概述](#1)
- [第二节 Sqoop安装与配置](#2)
- [第三节 Sqoop数据导入](#3)
- [第四节 Sqoop数据导出](#4)
- [第五节 Sqoop高级导入导出](#5)
- [第六节 Sqoop生产环境优化](#6)

***

<h4 id='1'>第一节 Sqoop系统概述</h4>

1. 了解Sqoop发展历史
2. 了解Sqoop特点
3. 了解Sqoop使用案例

---

Sqoop介绍
- Sqoop项目在大数据中的流程
    - 数据库（MySQL等）中的数据与Hive、HDFS进行交换
    - 数据库与Web系统集成；前端业务对性能的要求更高
    - Sqoop支持HDFS、Hive、HBase等系统间的数据导入导出
        - Sqoop只支持这些系统与数据库之间的对接
```mermaid
graph LR
原始数据集 --> MapReduce清洗
MapReduce清洗 --> 存入HBase
存入HBase --> Hive统计分析
存入HBase --> Sqoop导出
Hive统计分析 --> 存入Hive表
存入Hive表 --> Sqoop导出
Sqoop导出 --> MySQL数据库
MySQL数据库 --> Web展示
```
- Sqoop分两个版本：Sqoop1、Sqoop2

Sqoop原理架构
- 通过JDBC连接数据库，在map中将数据导入
    - 某些数据库没有开源的JDBC连接或使用特殊的连接器，在使用Sqoop时需要二次开发
    - JDBC对数据库的压力比较大，通常使用备用库导数据
    - map数量=数据库某表的全部数据量/map能处理的数据量

***

<h4 id='2'>第二节 Sqoop安装与配置</h4>

略

***

<h4 id='3'>第三节 Sqoop数据导入</h4>

1. 了解Sqoop导入操作
2. 了解Sqoop导入策略

---

Sqoop导入：关系数据库 -> Hadoop(HDFS/Hive/HBase/...)
- 导入HDFS
    - 使用sqoop import命令
        - -connect jdbc:mysql://
        - -table tblname
            - 一次只能导入一张表
        - -username root
        - -password root
        - -target-dir /dir
            - HDFS存放目录，目录不能存在（非增量导入的时候）
        - -fields-terminated-by '\t'
        - -m 1
            - map数量
        - -null-string ''
            - 导入时对字段为空的字符进行替换
        - -incremental append
            - 增量导入
        - -check-column id
            - 增量导入时的参考列
        - -last-value num
            - 上一次导入的最后一个值
    - 默认导入 /user/用户名/表名 目录下 
```
sqoop import --connect jdbc:mysqlq://localhost:3306/default --username root --password root --query 'select * from users where id<60 and $CONDITIONS' --split-by id -m 1 --target-dir /output/query/
```
```
[root@hadoop001 ~]# sqoop import \
> --connect jdbc:mysql://hadoop001:3306/mysql \
> --username root \
> --password root \
> --table help_keyword \
> --target-dir /user/root/help_keyword \
> --fields-terminated-by '\t' \
> --m 1
```
- 导入Hive
    - 需要有Hive的服务
    - Sqoop需要有Hive的jar包
    - 默认导入 /user/hive/warehouse/表名 路径下
    - sqoop一般会和hive外部表搭配，指定location
    - 默认分隔符"\u0001"
```
sqoop create-hive-table ...
```
```
[root@hadoop001 ~]# sqoop import \
> --connect jdbc:mysql://hadoop001:3306/mysql \
> --username root \
> --password root \
> --table help_keyword \
> --fields-terminated-by '!' \
> --lines-terminated-by '\n' \
> --hive-import \
> --m 1
> --delete-target-dir
```

***

<h4 id='4'>第四节 Sqoop数据导出</h4>

1. 了解Sqoop导出操作
2. 了解Sqoop导出策略

---

HDFS->关系型数据库
- 全表导出
```
sqoop export 
--connect jdbc:
--username root
--password root
--table tblname//表必须存在
--export-dir /dir
--fields-terminated-by ','
```
- 部分导出
```
sqooop export
--connect jdbc:
--username root
--password root
--table tblname
--num-mappers 1
--input-fields-terminated-by "\t"
--export-dir /.../part-m-000000
```

***

<h4 id='5'>第五节 Sqoop高级导入导出</h4>

1. 了解Sqoop高级导入导出操作
2. 了解Sqoop高级导入导出策略

---

密码保护技术
- 使用-P参数代替--password

压缩与并行控制
- 使用--compress选项，使数据在HDFS压缩
- 并行控制：默认并行数量为4，可以使用--num-mappers(--m)指定mappers数量

query,columns,where条件导入
- query的where字句必须有$CONDITIONS，固定写法，用于提供MAP分区逻辑，不需要table参数
- --columns指定输入列，用“,”分隔
- --where指定输入table的查询条件

导出文件格式设置
- --as-parquetfile
- --as-sequencefile
- --as-avrodatafile

快速导入模式
- --direct
- 提高传输速度
- 只支持MySQL和PostgreSQL
- 使用数据库提供的本地工具进行数据传输
    - MySQL：使用mysqldump和mysqlimport
    - PostgreSQL：使用pg_dump工具
- 只支持--as-sequencefile,--as-avrodatafile

Sqoop Job使用流程
- 通过job来管理特定的同步任务，可以记录last-value等结果
    - -create <job-id>
    - -delete <job-id>
    - -exec <job-id>
    - -list
    - -meta-connect <jdbc-url>
    - -show <job-id>
    - -verbose
    - -help
- 创建作业
    - sqoop job --create your_job -- import --connect...
- 查看作业
    - sqoop job --show your_job
- 执行作业
    - sqoop job --exec your_job

***

<h4 id='6'>第六节 Sqoop生产环境优化</h4>

1. 了解Sqoop生产环境技巧
2. 了解Sqoop优化策略

---

Sqoop生产环境技巧
- 测试Sqoop语句时，一定要限制记录数量
    - --where "ID=3"
    - --query "..."
- Sqoop导入时删除string类型字段的特殊字符
    - 如果指定了\n为Sqoop导入的换行符，当某个记录的string字段中含有\n，则会导致Sqoop导入多一条记录
    - --hive-drop-import-delims 
        - Drops \n,\r, and \01 from string fields when importing to Hive
- datetime的值为0000-00-00 00:00:00的时候，sqoop import成功，但是hive中执行select查询时报错。原因是Hive的日期类型只到日，没有时分秒，与MySQL日期格式不对应。因此创建Hive表时应用string字段类型，或者检查时间值。
- Sqoop将数据导入到MySQL中出现错误，检查因为某条记录导入不到MySQL。原因可能是数据有乱码。(Hadoop环境中不检查数据格式，MySQL会检查)
- MySQL驱动版本不对，导致报错：Ensure that you have called .close() on any active streaming result sets before attempting more queries. ERROR tool.ImportTool: Encountered IOException running import job: java.io.IOException: No columns to generate for ClassWriter
- MySQL连接超时，导致报错：The driver has not received any packets from the server.
    - 修改mysql连接参数，新增autoReconnect，或修改mysql连接配置的超时时间：jdbc:mysql://localhost:3306//bireport?useUnicode=true&characterEncoding=UTF8&autoReconnect=true&failOverReadOnly=false