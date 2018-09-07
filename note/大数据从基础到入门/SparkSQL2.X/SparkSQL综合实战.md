# 目录 #

- [第一节 使用数据源](#1)
- [第二节 SparkSQL编程案例](#2)
- [第三节 性能优化](#3)

***

<h4 id='1'>第一节 使用数据源</h4>

1. 掌握通用的Load/Save函数
2. 掌握Parquet文件
3. 理解JSON Datasets
4. 掌握使用JDBC
5. 掌握使用Hive Table

---

Load/Save函数
- load函数是用在Spark SQL中加载不同的数据源
    - 默认的数据源是Parquet文件
        - 修改默认数据源：spark.sql.sources.default
- save函数的存储模式
    - error（默认）：如果数据（目录）已存在，会报错
    - append：如果数据（目录）已存在，会追加
    - overwrite：如果数据（目录）已存在，会覆盖
```
// 加载parquet文件
val userDF = spark.read.load("/opt/software/spark-2.2.0-bin-hadoop2.6/examples/src/main/resources/users.parquet")

userDF.printSchema
userDF.show

userDF.select($"name",$"favorite_color").write.save("/tmp/result/")
userDF.select($"name",$"favorite_color").write.mode("overwrite").save("/tmp/result/")
userDF.select($"name",$"favorite_color").write.mode("append").save("/tmp/result/")

// 加载非parquet文件
val df = spark.read.format("json").load("/home/shanxiao/data/test_spark/emp.json")
val df = spark.read.json("/home/shanxiao/data/test_spark/emp.json")
val df = spark.read.csv...
```

Parquet
- 是列式存储文件，是SparkSQL默认的数据源格式
- 把其他类型的文件转换成Parquet文件
    ```
    val empJson = spark.read.json("/home/shanxiao/data/test_spark/emp.json")

    // 写成parquet文件
    empJson.write.mode("overwrite").parquet("/home/shanxiao/data/test_spark/parquet")

    // 重新读取
    val empParquet = spark.read.parquet("/home/shanxiao/data/test_spark/parquet")

    // 注册为视图
    empParquet.createOrReplaceTempView("empView")

    // 查询
    spark.sql("select * from empView where deptno=10 and sal>1500").show
    ```
- Parquet文件支持Schema的合并
    ```
    val df1 = sc.makeRDD(1 to 5).map(x=>(x,x*2)).toDF("single","double")
    val df2 = sc.makeRDD(6 to 10).map(x=>(x,x*3)).toDF("single","triple")

    df1.write.parquet("/home/shanxiao/data/test_spark/parquet/test_table/key=1")
    df2.write.parquet("/home/shanxiao/data/test_spark/parquet/test_table/key=2")

    val df3 = spark.read.option("mergeSchema", "true").parquet("/home/shanxiao/data/test_spark/parquet/test_table")
    --->df3.printSchema
            root
            |-- single: integer (nullable = true)
            |-- double: integer (nullable = true)
            |-- triple: integer (nullable = true)
            |-- key: integer (nullable = true)
    ```

JSON数据文件
- SparkSQL能自动解析JSON数据集的Schema
- 该JSON文件的每一行必须包含一个独立的、自满足有效的JSON对象
    - 如果用多行描述一个JSON对象，读取会出错

JDBC
- 启动Spark Shell时加载JDBC的Driver
    ```
    bin/spark-shell --master spark://hadoop001:7077 --jars ojdbc6.jar --driver-class-path ojdbc6.jar
    ```
- 从数据库中读取数据
    ```
    val oracleDF = spark.read.format("jdbc")
    .option("url","jdbc:oracle:thin:@localhost:1521/test")
    .option("dbtable","test.emp")
    .option("user","test")
    .option("password","test")
    .load
    ```
- 定义一个Properties类
    ```
    import java.util.Properties
    val oracleProp = new Properties()
    oracleProp.setProperty("user","test")
    oracleProp.setProperty("password","test")

    val oracleDF = spark.read.jdbc("jdbc:oracle:thin:@localhost:1521/test", "test.emp", oracleProp)
    ```

Hive
- Hive表对应HDFS的一个目录
- Hive表数据对应HDFS的一个文件
- 是一个数据分析引擎，支持SQL
    - 翻译器：SQL-->MapReduce
    - 从Hive2.x开始，推荐使用Spark作为Hive的执行引擎
        - 将HiveSQL转换成Spark任务提交到Spark集群上执行
- 配置集成SparkSQL和Hive
    - 将Hive和Hadoop的配置文件拷贝到$SPARK_HOME/conf
        - hive-site.xml
        - core-site.xml
        - hdfs-site.xml
    - 启动spark-shell，指定mysql的driver
        ```
        bin/spark-shell --master spark://hadoop001:7077 --jars $HIVE_HOME/lib/mysql-connector-java-5.1.40-bin.jar
        ```
    - 执行SQL
        ```
        spark.sql("show tables").show
        ```

***

<h4 id='2'>第二节 SparkSQL编程案例</h4>

1. 掌握指定Schema格式
2. 运用使用case class
3. 掌握将数据保存到数据库

---

- 使用指定Schema格式
```
import org.apache.spark.sql.{Row, SparkSession}
import org.apache.spark.sql.types.{IntegerType, StringType, StructField, StructType}

object SpecifyingSchema {

  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder().master("local").appName("SpecifyingSchema").getOrCreate()

    val studentRDD = spark.sparkContext.textFile("students").map(_.split(" "))

    val schema = StructType (
      List(
        StructField("id", IntegerType, false),
        StructField("name", StringType, true),
        StructField("age", IntegerType, true)
      )
    )

    val rowRDD = studentRDD.map(x => Row(
      x(0).toInt, x(1), x(2).toInt
    ))

    val studentDF = spark.createDataFrame(rowRDD, schema)

    studentDF.createOrReplaceTempView("students")

    spark.sql("select * from students").show()

    spark.stop()
  }

}
```

- 使用case class定义schema格式
```
import org.apache.spark.sql.SparkSession

case class Student(id: Int, name:String, age: Int)

object CaseClassSchema {

  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder().master("local").appName("CaseClassSchema").getOrCreate()

    val studentRDD = spark.sparkContext.textFile("students").map(_.split(" "))

    val rowRDD = studentRDD.map(x => Student(
      x(0).toInt, x(1), x(2).toInt
    ))

    import spark.sqlContext.implicits._
    val studentDF = rowRDD.toDF()

    studentDF.createOrReplaceTempView("studnets")

    spark.sql("select * from students").show()

    spark.stop()
  }

}
```

- 将查询结果保存到关系型数据库
```
// 结果保存如数据库
val result = spark.sql("select * from students")

// 定义JDBC
val props = new Properties()
props.setProperty("user", "test")
props.setProperty("password", "test")
// 通过JDBC写入数据库
result.write.jdbc("jdbc:oracle:thin:@localhost:1521/test", "test.tbl", props)
result.write.mode("append").jdbc("jdbc:oracle:thin:@localhost:1521/test", "test.tbl", props)
```

***

<h4 id='3'>第三节 性能优化</h4>

在内存中缓存数据
- spark.sqlContext.cacheTable("tblName")
- dataFrame.cache()
    - 标识表可以缓存，并不会实际进行缓存
    - 在第一次对表进行操作时，会对表进行缓存操作
    - 从第二次开始再进行操作时，操作性能会提高许多
- 清空缓存
    - spark.sqlContext.uncacheTable("tblName")
    - spark.sqlContext.clearCache