# 目录 #

- [第一节 SparkSQL基础](#1)
- [第二节 使用DataSet](#2)

***

<h4 id='1'>第一节 SparkSQL基础</h4>

1. 了解SparkSQL简介
2. 掌握基本概念：DataFrames
3. 测试数据
4. 掌握创建DataFrames
5. 掌握DataFrame操作
6. 运用Global Temporary View

---

Spark SQL
- Spark SQL is Apache Spark's module for working with structured data.
- Spark SQL是Spark用于处理结构化数据的模块，类似Hadoop的Hive
- 特点
    - 容易集成（整合）
    - 统一的数据访问方式：JSON、CSV、JDBC、Parquet
    - 兼容Hive
    - 支持标准的数据库连接：JDBC、ODBC

DataFrame
- 组织成命名列的数据集
    - 底层是RDD
- 类似关系型数据库中的表
    - 表Table = 表结构 + 数据
- DataFreame（表） = schema（结构） + Data（数据）
- 数据源：结构化的数据文件（CSV）、Hive中的数据、外部数据库、现有的RDD
- API支持语言：Scala、Java、Python、R
- 创建DataFrame
    - 通过case class来定义schema
        ```
        // 定义表结构
        case class Emp(
            empno:Int,
            ename:String,
            job:String,
            mgr:String,
            hiredate:String,
            sal:Int,
            comm:String,
            deptno:Int
        )

        // 导入数据文件
        val lines = sc.textFile("hdfs://hadoop001:9000/data_test/emp.csv").map(_.split(","))

        // 生成DataFrame
        val empDF = lines.map(x=>Emp(
            x(0).toInt,
            x(1),
            x(2),
            x(3),
            x(4),
            x(5).toInt,
            x(6),
            x(7).toInt)
        ).toDF

        // 操作DataFrame
        empDF.show
        empDF.printSchema
        ```
    - 使用SparkSession
        - SparkSession是Spark新的访问接口（统一的访问方式），来访问Spark的各个模块，不必用不同的SparkContext来访问
        ```
        import org.apache.spark.sql._
        import org.apache.spark.sql.types._

        val mySchema = StructType(
            List(
                StructField("empno",DataTypes.IntegerType),
                StructField("ename",DataTypes.StringType),
                StructField("job",DataTypes.StringType),
                StructField("mgr",DataTypes.StringType),
                StructField("hiredate",DataTypes.StringType),
                StructField("sal",DataTypes.IntegerType),
                StructField("comm",DataTypes.StringType),
                StructField("deptno",DataTypes.IntegerType)
            )
        )

        // 将数据中的每一行映射成一个Row
        val rowRDD = lines.map(x=>Row(
            x(0).toInt,
            x(1),
            x(2),
            x(3),
            x(4),
            x(5).toInt,
            x(6),
            x(7).toInt)
        )
        // 使用SparkSession创建DataFrame
        val df = spark.createDataFrame(rowRDD, mySchema)

        df.show
        df.printSchema
        ```
    - 使用带有格式的数据文件（JSON）
        ```
        val empDF = spark.read.json("/home/shanxiao/data/test_spark/emp.json")

        empDF.show
        empDF.printSchema
        ```
- 操作DataFrame
    - DSL语句（不常用）
        - 查询所有列
            - df.show
        - 查询单列
            - df.select("ename").show
            - df.select($"ename").show
        - 查询时运算
            - df.select($"ename",$"sal",$"sal"+100).show
        - 条件查询
            - df.filter($"sal" > 2000).show
            - 底层是RDD算子
        - 分组
            - df.groupBy($"deptno").count.show
    - SQL语句
        - 需要将DataFrame注册成一个试图（或真正的表）才能执行SQL操作
            - df.createOrReplaceTempView("emp")
        ```
        spark.sql("select * from emp").show
        spark.sql("select * from emp where deptno = 10").show
        spark.sql("select job, avg(sal) from emp group by job").show
        ```

视图（View）
- 是一个虚表
    - 所有的操作和表是一样的
    - 本身不存储数据，操作的是视图对应的表
- 类型
    - 局部视图
        - 只在当前会话中有效
        - df.createOrReplaceTempView("emp")
    - 全局视图
        - GlobalTemporaryView
        - 可以在不同Session中访问
            - 相当于SparkSQL维护了一个“全局数据库”global_temp
        - df.createOrReplaceGlobalTempView("empG")
        ```
        spark.sql("select * from global_temp.empG").show
        spark.newSession.sql("select * from global_temp.empG").show
        ```

***

<h4 id='2'>第二节 使用DataSet</h4>

1. 了解什么是DataSet
2. 掌握创建和使用DataSet

---

DataSet
- DataFrame可以让Spark更好的处理结构化的数据的计算
- 问题：缺乏编译时类型安全
- 为了解决这个问题，Spark采用新的DataSet API
    - DataFrame API的扩展
    - Datasets = DataFrames + RDDs
- 创建和使用DataSet
    - 序列
        ```
        case class MyData(a:Int, b:String)
        val ds = Seq(MyData(1,"Tom"),MyData(2,"Mary")).toDS

        ds.show
        ds.pringSchema

        ds.collect
        ```
    - 读取带格式的数据文件（JSON）
        ```
        // 从json文件创建DataFrame
        val df = spark.read.json("/home/shanxiao/data/test_spark/emp.json")

        // 定义Schema结构
        case class Emp(
            empno:Long,
            ename:String,
            job:String,
            mgr:String,
            hiredate:String,
            sal:Long,
            comm:String,
            deptno:Long
        )

        // 将DataFrame转换成DataSet来操作
        df.as[Emp].show
        df.as[Emp].collect
        ```
    - 读取HDFS的数据文件
        ```
        // 读取HDFS文件，直接创建DataSet
        val lineDS = spark.read.text("hdfs://hadoop001:9000/test.txt").as[String]

        // 分词操作，查询长度大于3的单词
        val words = lineDS.flatMap(_.split(" ")).filter(_.length > 3)

        words.show
        words.collect
        ```