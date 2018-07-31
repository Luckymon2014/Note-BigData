1. 了解Pig发展简史
2. 掌握Pig安装及配置
3. 掌握Pig基本使用方法
4. 了解Pig在生产环境中使用
5. 能够解决Pig开发中常见的问题

# 目录 #

- [第一节 Pig系统概述](#1)
- [第二节 Pig系统安装及配置](#2)
- [第三节 Pig Latin语法](#3)
- [第四节 Pig函数操作](#4)
- [第五节 Pig实现案例](#5)
- [第六节 Pig常见问题及优化方法](#6)

***

<h4 id='1'>第一节 Pig系统概述</h4>

1. 了解Pig发展简史
2. 掌握Pig主要版本
3. 了解Pig使用场景

---

Pig的起源和发展
- http://pig.apache.org/
- Pig是一个基于Hadoop或Spark的大规模数据分析平台
- 提供SQL-like语言：Pig Latin
    - 该语言的编译器会把类SQL的数据分析请求转换为一系列经过优化处理的MapReduce / Spark Core运算
- 2006年，Pig最早是雅虎公司基于Hadoop的并行处理架构
    - 目的：为了提升MapReduce的研发效率
- 2008年，雅虎将Pig捐献给Apache

Pig简化MapReduce——WordCount
```
--load文本的txt数据，并把每行作为一个文本
a = load '$in' as (f1:chararray);
--将每行数据，按指定分隔符分割，并转为扁平结构
b = foreach a generate flatten(TOKENIZE(f1, ' '));
--对单词分组
c = group b by $0;
--统计每个单词出现的次数
d = foreach c generate group, count($1);
--存储结果数据
store into '$out'
```

用Pig做什么
- 大规模ETL（数据清洗、数据排序、数据分析）
- 不能做大规模的SQL统计分析，如数据仓库
    - 使用Hive或Spark SQL来处理
- Pig优点
    - 减少Spark与Hadoop脚本的开发
    - 适合数据清洗、转换等类型ETL任务
- Pig缺点
    - 不适合做数据分析（SQL类型）
    - 权限控制不好做
        - 需要结合Yarn和Hadoop的安全特性来实施

Pig程序举例：查询被20到29岁网民访问的网址列表
```
USERS = load 'users' as (uid, age);
USERS_20s = filter USERS by age >= 20 and age <=29;
PVs = load 'pages' as (url, uid, timestamp);
PVs_20s = join USERS_20s by uid, PVs by uid;
```

***

<h4 id='2'>第二节 Pig系统安装及配置</h4>
略

***

<h4 id='3'>第三节 Pig Latin语法</h4>

1. 掌握Pig Latin基本语法
2. 能够使用Pig Latin开发程序
3. 掌握Pig Latin生产环境开发流程

---

Pig Latin介绍
- 高级数据处理语言：使用Pig分析Hadoop中的数据

Pig数据模型：完全嵌套
- Atom（原子）
    - 任何单个值，无论数据类型
    - 一条数据或一个简单的原子值，被称为字段
- Tuple（元祖）
    - 由有序字段集合形成的记录，可以是任意类型
    - 类似关系数据库中的行
- Bag（包）
    - 一组无序的元祖，即元祖的集合
    - 每个元祖可以有任意数量的字段（灵活模式）
    - 由“{}”表示，类似关系数据库中的表，但是不同在于，不需要每个元祖包含相同数量的字段，或者相同的位置（列）中的字段具有相同类型
    - 内包（inner bag）：整个包作为某个字段使用
- Map（映射）
    - 一组Key-Value对
        - Key：chararray类型，唯一
        - Value：任意类型
    - 由“[]”表示
- Relation（关系）
    - 一个关系是一个元祖的包
    - 是无序的，不能保证按任意特定顺序处理元祖

Pig语法基础——语句
- 使用关系relation，包扣表达式expression和模式schema
- 语句以分号结尾
- 除了LOAD和STORE，在执行所有其他操作时，采用关系作为输入，并产生另一个关系作为输出

Apache Pig执行
- 执行模式
    - Local模式
        - 本地文件，不需要Hadoop，用于测试
    - MapReduce模式
        - HDFS文件
    - Spark模式
        - 使用Apache Spark进行运行
- 脚本执行方式
    - 交互模式（Grunt Shell）
        - 输入Pig Latin语句并获取输出（使用Dump运算符）
    - 批处理模式（脚本）
        - 将Pig Latin脚本写入具有.pig扩展名的单个文件中，以批处理模式运行Apache Pig
    - 嵌入式模式（UDF）
        - Apache Pig允许在Java等编程语言中定义函数（UDF用户定义函数），并在脚本中使用它们

Apache Pig Grunt Shell
- Grunt Shell主要用于编写Pig Latin脚本
    - Local模式：```$ ./pig -x local```
    - MapReduce模式：```$ ./pig -x mapreduce```
- 执行脚本方法如下
    - Local模式：```$ pig -x local script.pig```
    - MapReduce模式：```$ pig -x mapreduce script.pig```

Pig数据类型
- int, long, float, double, chararray, bytearray
- map, tuple, bag
- 操作符：+, -, *, /, %, ==
- NULL

Pig字段定义-Schema
- 类似Table
- 可以指定relation为特定的结构，为字段指定名称和类型
-


***

<h4 id='4'>第四节 Pig函数操作</h4>

***

<h4 id='5'>第五节 Pig实现案例</h4>

***

<h4 id='6'>第六节 Pig常见问题及优化方法</h4>