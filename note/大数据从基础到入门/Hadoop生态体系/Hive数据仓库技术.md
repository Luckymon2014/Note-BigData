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
-[第十二节 Hive JDBC实践](#12)
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

1. 了解Hive函数使用
2. 掌握在HQL中使用函数

---

- 查询hive函数表：show functions;

Hive函数分类
- 内置函数
    - 简单函数-map
        - todate
        - tochar
    - 聚合函数-reduce
        - sum
    - 集合函数-map
        - join
    - 特殊函数
- 自定义函数
    - UDF-map
    - UDAF-reduce

Hive时间函数
- unix_timestamp()：返回当前时间的unix时间戳
- from_unixtime(bigint unixtime[, string format])：时间戳转日期函数
- unix_timestamp(string date)：返回指定日期格式的时间，date的形式必须为“yyyy-MM-dd HH:mm:ss”
- unix_timestamp(string date, string pattern)：返回指定日期格式的时间戳
- to_date(string date)：返回时间字段中的日期部分
- year(string date)：返回时间字段中的年
- month(string date)：返回时间字段中的月
- day(string date)：返回时间字段中的日

Hive数学函数
- round(a)：四舍五入的BIGINT值
- log([a,]b)：a默认是e
- sqrt(double/decimal a)
- abs(a)
- count/sum/avg/min/max
    - 聚合函数，会触发MapReduce

Hive字符串处理函数
- ltrim/rtrim(string A)：去掉空格
- concat(A, B, ...)：连接字符串
- substr/substring(A, int start, int end)
- upper/ucase/lower/lcase(A)

Hive Xpath函数
- Xpath是一种语言，专门解析XML的内容
- xpath：解析XML函数
- xpath_string：解析字符串类型XML函数
- xpath_Boolean：解析boolean类型XML函数
    - 判断是否存在标签
- xpath_short/int/long：解析整数类型XML函数
- xpath_float/double/number：解析小数类型XML函数
    - 不是数字，返回NaN

***

<h4 id='6'>第六节 Hive自定义函数</h4>

1. 了解Hive HUF、UDAF函数使用
2. 掌握在HQL中使用自定义函数

---

Hive UDF(User Define Function)
- 单输入、单输出（如substr等函数）
- 开发方式
    - 继承org.apache.hadoop.hive.ql.exec.UDF（已废除）
    - 继承org.apache.hadoop.hive.ql.udf.generic.GenericUDF
- 开发过程
    1. 按照GenericUDF的类，实现我们想要的功能
    2. 将jar包放入HDFS集群：hdfs dfs -put xxx.jar /
        - 本地也可以，但是一般都是在分布式集群上执行
    3. hive shell/client：add jar ...
    4. create tempory function func as 'com.udf.....Demo'
        - 临时函数，只对当前session有效
    5. select func(arguments) from ...;
    6. drop tempory function func;

```
public class Demo extends GenericUDF {
    
    // 计算函数
    // select func(arguments, arguments, ...) from table;
    abstract Object evaluate(GenericUDF.DeferredObject[] arguments);

    // 描述函数
    abstract String getDisplayString(String[] children);

    // 初始化
    abstract ObjectInspector initialize(ObjectInspector[] arguments);
}
```

```
public class GenderToNumber extends GenericUDF {
	
	// 转化HQL中的入参
	private ObjectInspectorConverters.Converter[] converters;

	@Override
	public Object evaluate(DeferredObject[] arguments) throws HiveException {
		if (null == arguments[0].get())
			return null;
		
		String gender = (String) converters[0].convert(arguments[0].get());
		return stringToNumber(gender);
	}
	
	/**
	 * 自定义方法
	 * @param gender
	 * @return
	 */
	protected String stringToNumber(String gender) {
		if (gender.equalsIgnoreCase("male"))
			return "1";
		else
			return "2";
	}

	@Override
	public String getDisplayString(String[] arg0) {
		return "function description";
	}

	// select function(gender);
	@Override
	public ObjectInspector initialize(ObjectInspector[] arguments) throws UDFArgumentException {
		// 校验入参长度
		if (arguments.length != 1)
			throw new UDFArgumentException("arguments is not right! \n "
					+ "e.q: select function(gender);");
		
		// 转化入参，将参数存入converters
		converters = new ObjectInspectorConverters.Converter[arguments.length];
		for (int i=0; i<arguments.length; i++) {
			// 初始化converters，将入参转化为Java字符串类型
			converters[i] = ObjectInspectorConverters.getConverter(arguments[i], PrimitiveObjectInspectorFactory.javaStringObjectInspector);
		}
		
		// 返回Java字符串类型的值
		return PrimitiveObjectInspectorFactory.getPrimitiveJavaObjectInspector(PrimitiveObjectInspector.PrimitiveCategory.STRING);
	}

}
```

Hive UDAF
- 自定义聚合函数，零到多行输入，单输出
- 开发方式
    - org.apache.hadoop.hive.ql.udf.generic.AbstractGenericUDAFResolver
    - org.apache.hadoop.hive.ql.udf.generic.GenericUDAFEvaluator
    - org.apache.hadoop.hive.ql.exec.NumericUDAF
```
public class UDAFSum extends NumericUDAF {
    public static class Evaluator implements UDAFEvaluator {
        // 初始化
        public void init() {}
        // map阶段聚合
        public boolean iterate (DoubleWritable 0) {}
        // 无参数，类似hadoop的combiner，为iterate函数轮转结束后，返回部分聚合数据的持久化
        public DoubleWritable terminatePartial()
        // reduce阶段聚合
        public boolean merge(DoubleWritable o) {}
        // 返回最终的聚集函数结果
        public DoubleWritable terminate() {}
    }
}
```

```
public class UDAFSum extends NumericUDAF {
	
	public static class Evaluator implements UDAFEvaluator {

		private boolean empty;
		private double sum;

		// 初始化参数
		public void init() {
			sum = 0;
			empty = true;
		}
		
		public Evaluator() {
			super();
			init();
		}
		
		// select sum(o);
		// mapper
		public boolean iterate(DoubleWritable o) {
			if (null != o) {
				sum += o.get();
				empty = false;
			}
			return true;
		}
		// combiner
		public DoubleWritable terminatePartial() {
			return empty? null: new DoubleWritable(sum);
		}
		// reducer
		public boolean merge(DoubleWritable o) {
			if (null != o) {
				sum += o.get();
				empty = false;
			}
			return true;			
		}
		// final combine
		public DoubleWritable terminate() {
			return empty? null: new DoubleWritable(sum);			
		}
	}

}
```

***

<h4 id='7'>第七节 Hive2.0存储过程实践</h4>

1. 掌握Hive2.0存储过程说明
2. 掌握Hive2.0存储过程使用

---

Hive2.0存储过程说明
- Hive从2.0开始支持存储过程HPL/SQL
- 支持Oracle PL/SQL、ANSI/ISO SQL/PSM(IBM DB2, MySQL, Teradata...)、PostgreSQL PL/pgSQL(Netezza)、Transact-SQL(Microsoft SQL Server and Sybase)

Hive2.0存储过程使用
```
CREATE FUNCTION test(context STRING) RETURNS STRING
BEGIN
    RETURN 'Hello,'||context||'!';
END;

FOR item IN (
    SELECT id,name FROM a limit 10
)
LOOP
    PRINT a.id||'|'||a.name||'|'||test(a.name);
END LOOP;
```
- ./hplsql -f text.sql
1. 安装Hive2.0以上版本
2. 安装MySQL
3. 配置Hive2.0
4. hplsql-site.xml
5. 初始化hive2.0
    - schematool -dbType mysql -initSchema

```
create procedure set_value(IN arg STRING)
begin
    set value = 'hello'
    print value || ',' || arg;
end;
```
- hplsql -f ~/hplpro.sql -main set_value

***

<h4 id='8'>第八节 Hive Index原理及使用</h4>

1. 了解Hive Index原理
2. 掌握在Hive中使用Index

---

Hive Index
- 索引：提高Hive表指定列的查询速度
- 增加索引，会消耗额外的资源去创建索引，需要更多的磁盘空间存储索引
- Hive0.7.0版本加入了索引，Hive0.8.0增加了bitmap索引
    - CompactIndexHandler(压缩索引)：通过将列中相同的值的字段进行压缩，从而减小存储和加快访问时间
    - Bitmap(位图索引)：如果索引列只有固定的几个值，可以采用位图索引来加速查询，可以方便的进行AND/OR/XOR等各类计算
        - 数字不适合用位图索引
        - 同时存在位图索引和压缩索引，优先选择位图索引

索引原理
- 在指定列上建立索引，会产生一张索引表（Hive的一张物理表），里面的字段包扣：索引列的值、该值对应的HDFS文件路径、该值在文件中的偏移量
- 在执行索引字段查询的时候，首先额外生成一个MR job，根据对索引列的过滤条件，从索引表中过滤出索引列的值对应的HDFS文件路径及偏移量，输出到HDFS上的一个文件中，然后根据这些文件中的HDFS路径和偏移量，筛选原始input文件，生成新的split，作为整个job的split，这样就达到不用全表扫描的目的

Hive Index使用
1. 创建压缩索引
    ```
    create index index_name on table table_name(key) 
    as 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' 
    with deferred rebuild;
    ```
    ```
    create index index_name on table table_name(key) 
    as 'BITMAP' with deferred rebuild;
    ```
2. 生成索引数据
    ```
    alter index index_name on table_name rebuild;
    ```
    - 如果表数据更新了，需要重建索引
3. 查看索引
    ```
    show index on table_name;
    ```
4. 设置查询时索引自动生效
    ```
    set hive.input.format = org.apache.hadoop.hive.ql.io.HiveImputFormat;
    set hive.optimize.index.filter = true;
    set hive.optimize.index.filter.compact.minsize = 0;
    ```

***

<h4 id='9'>第九节 Hive Update,Delete操作说明</h4>

1. 了解Hive Update,Delete原理
2. 掌握在Hive中使用Update,Delete

---

Hive Update,Delete配置
- hive.support.concurrency = true
- hive.enforce.bucketing = true
- hive.exec.dynamic.partition.mode = nonstrict
- hive.txn.manager = org.apache.hadoop.hive.ql.lockmgr.DbTxnManager
- hive.compactor.innitiator.on = true
- hive.compactor.worker.threads = 1

- 输出必须是AcidOutputFormat，必须分桶
- 只有ORCFileformat支持AcidOutputFormat
- 建表时必须指定参数('transactional'='true')

```
create table table_name(value type, value type...) clustered by (key) into 2 buckets stored as orc TBLPROPERTIES('transactional'='true');
```

- map的个数与桶的个数一致

***

<h4 id='10'>第十节 Hive ORCFile,Parquet 文件格式实践</h4>

1. 了解Hive ORCFile、Parquet File原理
2. 掌握在Hive中使用ORCFile、ParquetFile

---

Hive文件存储格式
- Hive文件存储格式主要用来增加查询效率
- 分类：TextFile、ORCFile、Parquet
- 生产环境中，综合使用这几种文件格式

Hive ORCFile
- hive/spark都支持这种存储格式，存储的方式：数据按照行分块，每个块按照列存储，其中每个块都存储有一个索引
    - 按行水平切割，再按列垂直切割
    - 针对不同列，采用特定的编码特点
        - 编码格式：bit、增量、字典、游程
            - 如：String类型，会用字典编码
            - 编码格式是可选的
    - 最后对编码以后的数据进行压缩
        - 压缩格式：zlib、snappy、lzo等
- 特点：数据压缩率非常高
- 对每一列，采用Stream存储
    - String列，如果字典编码，会有4个stream：present stream、data stream、dictionary stream、length stream
- 索引：包含min、max、avg、count等信息
- Bloomfilter(粗过滤)
- ORC表不能通过LOAD加载数据，需要从另一张表导入

```
create table orc_table (val type, ...) 
stored as orc 
tblproperties('orc.compress'='SNAPPY');
```

Hive Parquet
- Parquet是一种行式存储，同时具有很好的压缩性能，同时可以减少大量的表扫描和反序列化的时间
- 由多个Row Group组成
- 在Row Group中先按列来分段
- 然后按列的数据（行）继续切割
- 然后再对数据进行编码
- 参考自Google的Dremel技术，数据结构复杂，支持深度嵌套

```
create table parquet_table (val type, ...) 
stored as parquet;
```

- ORC、Parquet建表时不需要指定分隔符，适合没有分隔符的文件
- ORC、Parquet都是提高查询性能的技术，也能减少文件大小
- ORC相对比较流行，Parquet相对更耗CPU，需要在上线前进行测试：文件大小、性能

***

<h4 id='11'>第十一节 Hive 数据压缩及解决数据倾斜</h4>

1. 了解Hive数据倾斜原理
2. 掌握在Hive中使用数据压缩技术
3. 掌握Hive数据倾斜技术及调优

---

Hive压缩技术
- 主要目的是：提升IO，降低存储空间消耗
- 开启压缩
    - map输出压缩
        ```
        set hive.exec.compress.intermediate=true
        set mapreduce.map.output.compress=true
        set mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.SnappyCodec
        ```
        - 推荐在sql中添加，不要写在配置文件中，除非需要全局生效
    - reduce压缩
        ```
        set hive.exec.compress.output=true
        set mapreduce.output.fileoutputformat.compress=true
        set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.il.compress.SnappyCodec
        ```

Hive数据倾斜
- 数据倾斜：进行分布式计算过程汇总，数据的分散度不够，导致计算集中在某些服务器上，导致计算资源利用度不够
- 现象：某些机器上的任务非常慢，某些很快执行完毕
- 产生原因：
    - Shuffle阶段，Map的结果中同一个Key分配到同一个Reduce上，Key分布不均衡导致Reduce的任务不均衡
    - 产生数据倾斜的操作

        关键词|情形|后果
        -----|----|----
        Join|其中一个表较小，但是key集中|分发到某一个或几个Reduce上的数据远高于平均值
        Join|大表与大表，但是分桶的判断字段0值或空值过多|这些空值都是由一个Reduce处理，非常慢
        Group by|group by维度过小，某值的数量过多|处理某值的Reduce非常耗时
        Count Distinct|某特殊值过多|处理此特殊值的Reduce耗时

Hive数据倾斜优化
- 参数调优
    - Join操作：set hive.optimize.skewjoin.compiletime=true
    - GroupBy操作：set hive.groupby.skewindata=true
    - Join的键对应的记录条数超过这个值，则进行优化：set hive.skewjoin.key=100000
    - Map Join：set hive.map.aggr=true
        - select /*+ MAPJOIN(b)*/ a.key,a.value from a join b on a.key=b.key;
    
    - reduce处理数据量（默认1G=1000000000）：hive.exec.reducers.bytes.per.reducer
    - 最大reduce个数（默认1009）：hive.exec.reducers.max
    - 设置reduce个数=min(参数2(1009),map端输出数量总量/参数1(1G))：mapreduce.job.reduces
- 小表与大表关联
    - 将小表刷入内存：set hive.auto.convert.join=true;
    - 刷入内存表的字节数：set hive.mapjoin.smalltable.filesize=350000;
- 转换数据类型
    ```
    select * from test a left outer join logs b on a.id = cast(b.id as string)
    ```
- sum() group by替换count(distinct)
    ```
    select x, count(distinct y) as c from test group by x;
    -- 改写为
    select x, count(*) as c from (select distinct x, y from test) group by x;
    ```
- 空值数据倾斜：Join时，尽量避免null参与关联
    ```
    ... where val is not null
    ```

***

<h4 id='12'>第十二节 Hive JDBC实践</h4>

1. 了解Hive JDBC实现方式
2. 掌握在Java中调用JDBC

---

Hive JDBC配置
- Java程序Client → Thrift服务 → JDBC
- Port number of HiveServer2 Thrift interface：hive.server2.thrift.port=10000
- Bind host on which to run the HiveServer2 Thrift interface：hive.server2.thrift.bind.host=0.0.0.0
- 启动JDBC：nohup ./bin/hive --service hiveserver2 &

Hive JDBC JAVA
1. 启动metastore服务
2. 启动HiveServer2
- !connect jdbc:hive2://hadoop001:10000/default


