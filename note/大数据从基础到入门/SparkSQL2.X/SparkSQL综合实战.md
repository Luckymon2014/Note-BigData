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

    empJson.write.mode("overwrite").parquet("/home/shanxiao/data/test_spark/parquet")
    ```








***

<h4 id='2'>第二节 SparkSQL编程案例</h4>

***

<h4 id='3'>第三节 性能优化</h4>
