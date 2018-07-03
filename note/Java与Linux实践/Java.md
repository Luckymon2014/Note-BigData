## 1. 掌握Java基本语法 ##
## 2. 熟练使用Java集合框架 ##
## 3. 掌握I/O基本原理及使用 ##
## 4. 掌握多线程开发技术 ##
## 5. 掌握数据库的原理及使用 ##
## 6. 熟练使用JDBC技术完成数据库编程 ##
## 7. 完成Java开发技术的入门 ##

********

# 目录 #

[第一章 JavaSE基础](#c1)

- [第一节 开发环境搭建](#c1s1)
- [第二节 Java基本语法](#c1s2)
- [第三节 Java异常处理](#c1s3)
- [第四节 Java集合框架](#c1s3)

[第二章 JavaSE高级应用](#c2)

- [第一节 JavaIO流与文件操作](#c2s1)
- [第二节 字节流的包装和链接](#c2s2)
- [第三节 字符流的包装和链接](#c2s3)
- [第四节 对象的序列化](#c2s4)
- [第五节 线程的创建、运行和结束](#c2s5)
- [第六节 线程控制](#c2s6)

[第三章 JAVA操作数据库](#c3)

- [第一节 Linux系统下安装MySQL数据库](#c2s1)
- [第二节 MySQL数据库操作](#c2s2)
- [第三节 事务与隔离级别](#c2s3)
- [第四节 JDBC操作MySQL数据库](#c2s4)

********

<h1 id="c1">第一章 JavaSE基础</h1>

1. 掌握Java开发环境搭建
2. 掌握Java的基本语法
3. 掌握Java程序的异常处理
4. 掌握Java的集合框架

********

<h2 id="c1s1">第一节 开发环境搭建</h2>

1. 掌握[JDK](https://www.oracle.com/technetwork/java/javase/downloads/index.html)的下载安装
2. 配置JDK环境变量
3. 安装与使用[Eclipse](https://www.eclipse.org/downloads/)

----------

安装JDK

- 开发工具
    - JDK中的核心功能模块，包括一系列可执行程序，如javac.exe、java.exe，还包含了一个专用的JRE环境
- 源代码
    - Java提供公共API类的源代码
- 公共JRE
    - Java程序的运行环境。由于开发工具中已包含了一个JRE，此项可以不作选择

配置JDK

- JAVA_HOME：JDK的安装目录
- PATH：JDK工作的路径 //%JAVA_HOME%\bin

JDK安装目录结构

- ./bin：JDK工具目录
- ./jre：JDK运行环境目录
- ./lib：JDK的Java与C支持库
- ./include：JDK中C库的头文件目录
- ./db：Apache Derby嵌入式数据库引擎
- ./src.zip：JDK API库的源代码

********

<h2 id="c1s2">第二节 Java基本语法</h2>

1. 掌握Java数据类型
2. 掌握各种运算符的使用语法
3. 掌握控制语句的语法与执行流程
4. 掌握数组的使用语法
5. 掌握面向对象思想
6. 掌握类与对象的使用
7. 掌握方法的定义与调用语法
8. 掌握继承的使用

----------

关键字

- 是变成语言里事先定义好并赋予了特殊含义的单词，也称作保留字，约51个

分隔符

- 具有特殊分割作用的符号
- “;”：Java语句的分隔符
- “{}”：定义了一个代码块
- “[]”：访问数据，紧跟数据变量名
- “()”：形参声明、有限计算、强制类型转换
- “空格”：任意地方，除变量名中间
- “.”：调用某个类或某个实例的指定成员

标识符

- 用于给程序中变量、类、方法命名的符号，Java中必须以字母、下划线、$开头
    - 可以使用数字但不允许开头，不能包含空格，不能包好除“$”外的其他任何特殊字符

注释

- 单行注释： //
- 多行注释： /* ... */
- 文档注释： /** ... */

----------

数据类型

- 基本数据类型
    - 一般直接存储在分配的内存空间中，数据值长度固定
    - 逻辑类型
        - boolean 1字节
    - 字符类型
        - char 2字节
    - 整数类型
        - byte 1字节
        - short 2字节
        - int 4字节
        - long 8字节
    - 小数类型
        - float 4字节
        - double 8字节
- 引用类型
    - 类、接口、数组类型、null类型
    - 注：字符串是一个类，属于引用类型数据

常量

- 是在程序中固定不变的值，是不能改变的数据
- 定义：final 类型 常量名[=初始值]
- 注：只能赋值一次，不一定在定义的时候初始化

变量

- 定义变量的主要目的是存放数据
- 定义：数据类型 变量名[=值]
- 使用方式
    - 用来给另外一个变量赋值
    - 在表达式中使用
    - 直接用在函数中

赋值与类型转换

- 自动类型转换
    - 表数范围小的->表数范围大的
- 强制类型转换
    - 表数范围大的->表数范围小的
    - 例：int a = 123; byte b= (byte) a;
- 不同类型的数据不能互相转换

字符串

- 定义：String 字符串变量=["字符串"]
- 字符串长度：[字符串].length()
- 连接字符串：[字符串1] + [字符串2] + ...

----------

运算符

- 算数运算
    - 单目：一个变量参与运算
        - “+”、“-”、“++”、“--”
    - 双目：两个变量参与运算
        - “+”、“-”、“*”、“/”、“%”
- 比较运算
    - “==”、“!=”、“>”、“<”、“>=”、“<=”
- 逻辑运算
    - “&&”、“||”、“!”
- 复合运算
    - 与赋值运算符号“=”复合的运算，也称赋值运算符
        - “=”、“+=”、“-=”、“*=”、“/=”、“%=”、“&=”、“^=”、“|=”、“<<=”、“>>=”
    - 特殊的三目运算
        - 逻辑表达式? 表达式1 : 表达式2

运算优先级

- “[]”、“()”
- “++”、“--”
- “!”、“~”
- “instance of”
- “+”、“-”、“*”、“/”、“%”
- “<<”、“>>”、“>>>”
- “<>”、“<”、“=”、“>”
- “==”、“!=”
- “&”、“^”
- “&&”、“||”
- “?”、“:”
- “=”、“op=”

----------

流程控制

- if条件块
- switch条件块
    - 表达式类型必须是char、byte、short、int类型
    - JavaSE7.0以后增加了String类型
- while循环块
- for循环块
- break
- continue

----------

数组

- 数组是一次定义多个变量的另外一种方式，不同的变量类型都是一样的，变量名也一样，用下标的方式来区别
- 定义：数据类型[] 数组名={值1,值2,值3...}
    - 数据类型可以是基本类型，也可以是用户自定义类型
- 初始化数组
    - 空间分配
    - 对数组中的每个变量赋初值
- 数组的长度：数组名.length

----------

面向对象

- 在程序中使用对象来映射现实中的事物，使用对象的关系来描述事物之间的联系，这种思想就是面向对象
- 特点：封装性、继承性、多态性
- 面向对象的思想引入了两个概念，即类和对象
    - 类是对某一类事物的抽象描述，而对象用于表示现实中该类事物的个体
    - 类是对象的抽象，它用于描述一组对象的共同特征和行为

----------

类

- 类中可以定义成员变量和成员方法
    - 成员变量：描述对象的特征，也称作“属性”
    - 成员方法：描述对象的行为，也称作“方法”
- 修饰符
    - “public”、“final”、“abstract”
- 变量
    - 成员变量 //类的对象释放空间
    - 局部变量 //方法运行完毕，自动释放空间
- 成员方法
    - 构造方法
        - 没有返回值
        - 名称和类名相同
    - 普通方法
        - 有返回值（void）
        - 自定义名称
        - 有入参
    - main方法
        - 固定的运行入口
- this
    - 本类对象 //“自己”
- static
    - 静态，共享使用
    - 内存区域分配
        - 栈：局部变量
        - 堆：对象
        - 方法区：静态变量、String字符串
- 包
    - 管理类

继承

- 类的继承
    - class Child extends Parent
    - 单继承
    - Object类
        - Java中所有类都继承自Object类
- 实现接口
    - class Child implements interface1, interface2...
    - 多继承
- 多态
    - 父类可以接受子类的对象，并调用方法

********

<h2 id="c1s3">第三节 Java异常处理</h2>

1. 掌握异常语法
2. 掌握异常的开发使用方法

----------

异常

- 程序运行过程中，有时会发生非正常的状况
- Java引入了异常，以异常类的形式对这些非正常情况进行封装、处理
- 当异常发生时，异常后面的代码不会执行，只有处理之后的代码会被执行
- catch 处理异常，为了保证代码正常结束
- finally 必定会执行的代码，一般用于关闭资源

JDK异常类型设计

- Throwable
    - Error：错误类，表示Java运行时产生的系统内部错误，是比较严重的错误
        - 例：VirtualMachineError
    - Exception：异常类，表示程序本身可以处理的错误，在开发Java程序中需要进行捕获异常处理
        - 运行时异常：RuntimeException
            - 可以不处理，Jvm会处理（打印异常）
        - 编译时异常：其他Exception //必须解决的异常
            - 抛出异常，调用者必须处理/抛出;
                - 如果没人处理，Jvm会处理（打印异常）
            - 处理异常

********

<h2 id="c1s4">第四节 Java集合框架</h2>

1. 掌握List集合的使用技术
2. 掌握Set集合的使用技术
3. 掌握Map集合的使用技术
4. 掌握泛型的语法与使用

----------

集合

- 数组可以用来保存多个对象，但某些情况下，无法确定到底需要保存多少个对象，此时数组不再适用，因为数组的长度不可变
- JDK中提供了一系列特殊的类，这些类可以存储任意类型的对象，并且长度可变，统称为集合
- 按照存储结构分类
    - 单列集合Collection
        - List
            - ArrayList / LinkedList / Vector
        - Set
            - HashSet(LinkedHashSet) / TreeSet
    - 双列集合Map
        - Hashtable(Properties) / HashMap(LinkedHashMap) / TreeMap

Iterator接口
- 与Collection、Map接口并列为顶层接口
    - Collection、Map：存储元素
    - Iterator：迭代访问（即遍历）
- Iterator对象也被称为迭代器

----------

Collection接口
- 所有单列集合的父接口
- 定义了单列集合（List和Set）通用的一些方法
    - add(Object o)：增加元素
    - contains(Object o)：是否包含指定元素
    - iterator()：返回Iterator对象，用于遍历集合中的元素
    - remove(Object o)：移除元素
    - size()：返回元素个数
    - toArray()：转换为数组
- List接口
    - 继承自Collection接口
    - 特点是有序，允许元素重复，每个元素都有对应的顺序索引
        - 默认按元素的添加顺序设置元素的索引
        - 添加了一些根据索引来操作集合元素的方法
    - 相关操作
        - add(int index, Object o)：在指定位置插入元素
        - get(int index)：取得指定位置的元素
        - indexOf(Object o)：返回对象在集合中第一次出现的位置
        - remove(int index)：删除并返回指定位置的元素
        - set(int index, Object o)：替换指定位置的元素
    - 常用实现类
        - ArrayList：数组队列，相当于动态数组，非线性安全
        - LinkedList：继承了AbstractSequence，是一个双向链表。可以当做堆栈、队列或者双向队列使用，支持序列化
- Set接口
    - 继承自Collection接口
    - 特点是无序，会以某种规则保证存入的元素不出现重复
    - 常用实现类
        - HashSet：根据对象的哈希值来确定元素在集合中存储的位置，具有良好的存取和查找性能
            - 不能保证元素的排列顺序
            - 不是线性安全的
            - 集合元素可以是null
            - 元素不重复
                - 两个对象的hashCode()方法返回值相等
                - 两个对象通过equals()方法返回true
            - 存入元素时，HashSet会调用该对象的hashCode()方法来决定该对象在HashSet中的存储位置
        - TreeSet：以二叉树的方法来存储元素，它可以实现对集合中的元素进行排序
            - 元素不重复
            - 可以对元素进行排序
                - 自然排序（默认）
                    - 添加TreeSet元素时，该对象的类必须实现Comparable接口
                    - 实现Comparable的类必须实现compareTo(Object obj)方法
                    - Java基础类基本都已实现了compareTo方法
                - 定制排序
                    - new TreeSet<>(new Comparator<T>{...})
                    - 创建TreeSet集合对象时，提供一个Comparator接口的实现类对象。由该Comparator对象负责集合元素的排序逻辑

----------

Map接口

- 存储具有一一对应关系的数据
- 是一种双列集合，每个元素都包含一个键对象Key和值对象Value，键和值之间存在一种对应关系，称为映射
- 访问元素时，只要指定了Key，就能找到对应的Value
- 最常用的实现类：HashMap
    - for (Entry<K, V> e : hashMap.entrySet) { entry.getValue(); entry.getValue(); }

----------

泛型

- 把一个对象存入集合后，集合会“忘记”这个对象的类型，将该对象从集合中取出时，这个对象的编译类型就变成了Object类型
- 在取出元素时，如果进行强制类型转换，很容易出错
- 为了解决这个问题，Java引入了“参数化类型（parameterized type）”概念，即泛型
- 泛型可以限定方法操作的数据类型，在定义集合时，可以使用“<参数化类型>”的方式指定该类中方法操作的数据类型

********

<h1 id="c2">第二章 JavaSE高级应用</h1>

1. 掌握IO技术原理及使用
2. 掌握对象序列化技术
3. 掌握多线程运行流程及控制

********

<h2 id="c2s1">第一节 JavaIO流与文件操作</h2>

1. 熟练掌握文件管理中的术语
2. 熟练掌握文件操作技术

----------

IO流

- 大多数应用程序都需要实现与设备之间的数据传输，这种传输可以归纳为输入/输出数据
- Java中，将这种输入/输出数据的传输抽象表述为“流”
- Java中的“流”都位于java.io包中，所以称为IO流
- IO流分类
    - IO流（java.io）
        - 字节流
            - 字节输入流（java.io.InputStream）
            - 字节输出流（java.io.OutputStream）
        - 字符流
            - 字符输入流（java.io.Reader）
            - 字符输出流（java.io.Writer）
    - 字节流可以操作字符，字符流不可以操作字节

文件管理中的术语

- 文件与路径
    - 文件：目录或者文件对象
    - 路径：由目录名
- 抽象路径
    - Unix/Linux系统使用“/”作为开始
    - Windows系统使用“\”（UNC）或者盘符“:”作为开始

文件类
- File类
    - 用于封装一个路径（绝对路径/相对路径）
    - 封装的路径指向一个文件/目录，操作文件/目录之前，首先得创建一个File对象
- 构造文件对象
    - File(File parent, String child)
    - File(String pathname)
    - File(String parent, String child) //parent:路径，child：文件名
    - File(URI uri)
        - 解决跨平台问题：使用相对路径 & 使用静态变量
- 静态变量
    - static String pathSeparator //系统相关的路径分隔符
    - static char pathSeparatorChar
    - static String separator //系统相关的分隔符
    - static char separatorChar
- 文件属性与类型判定
    - boolean exists()
    - boolean isDirecotry() //用于目录的递归
    - boolean isFile()
- 文件基本操作
    - boolean delete()
    - boolean createNewFile()
    - boolean mkdir()
    - boolean mkdirs() //创建多层路径
    - String getName()
    - String getAbsolutePath() //绝对路径 
    - String getPath() //相对路径
- 文件遍历
    - String[] list()

********

<h2 id="c2s2">第二节 字节流的包装和链接</h2>

1. 了解字节流技术的基本原理
2. 熟练掌握字节流读、写数据
3. 熟练掌握包装流技术

----------

字节流

- 所有文件都是以二进制（字节）的形式存在
- IO流中针对字节的输入输出提供的一系列的流，统称为字节流
- JDK中提供了两个抽象类：InputStream和OutputStream，它们是字节流的顶级父类，所有的字节输入流继承自InputStream，所有的字节输出流继承自OutputStream
- 针对文件的读写，JDK专门提供了两个类：FileInputStream和FileOutputStream，是IS和OS的子类

字节流读取数据

- InputStream提供的基本操作
    - void close()
    - int read(byte[] b, int off, int len) //返回-1表示读取到文件末尾
1. 创建文件对象
    - File file = new File(filepath);
2. 关联文件对象和字节流
    - FileInputStream inputStream = new FileInputStream(file);
3. 读取数据
    - 读单个字节
        - (char) inputStream.read();
    - 读多个字节
        - byte[] buffer = new byte[5];
        - inputStream.read(buffer);
        - syso(new String(buffer, 0, buffer.length));
    - 读完整文件
        - byte[] buffer = new byte[5]; // 一次读取5个字节
        - int length = 0;
        - while((length = inputStream.read(buffer)) != -1) {
            syso(new String(buffer, 0, buffer.length));
        }
4. 提高效率
    - BufferedInputStream bufferedInputStream = new BufferedInputStream(inputStream);
    - ...while((length = bufferedInputStream.read(buffer)) != -1)...
5. 关闭资源
    - finally
        - bufferedInputStream.close();
        - inputStream.close();

字节流写数据

- OutputStream提供的基本操作
    - void close()
    - void flush()
    - void write(byte[] b, int off, int len)
1. 创建文件对象
    - File file = new File(filepath);
2. 关联文件对象和字节流
    - FileOutputStream outputStream = new FileOutputStream(file);
3. 写数据
    - outputStream.write("hello world".getBytes());
    - outputStream.flush(); //强制刷新，将数据从内存写入磁盘，否则数据将在流关闭时自动写入磁盘
4. 提高效率
    - BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(outputStream);
    - bufferedOutputStream.write("hello world".getBytes());
5. 关闭资源
    - finally
        - bufferedOutputStream.close();
        - outputStream.close();

缓冲区提高效率读写字节

- 自定义缓冲区BufferedInputStream、BufferedOutputStream

********

<h2 id="c2s3">第三节 字符流的包装和链接</h2>

1. 熟练掌握字符流读、写文本数据
2. 掌握字符集编、解码技术
3. 熟练掌握字符流的包装流技术

----------

字符流

- 在程序开发中，经常需要对文本文件的内容进行读写操作，为了操作方便，Java专门为文本的操作定义了相关API，这就是字符流
- 字符输入流FileReader是Reader的一个子类，字符输出流FileWriter是Writer的一个子类

字符流读数据

- FileReader reader = new FileReader(filepath);
- char[] buffer = new char[1024]; //一次读1024个字符
- reader.read(buffer);
- syso(new String(buffer, 0, buffer.length));
- *BufferedReader*
    - FileReader reader = new FileReader(filepath);
    - BufferedReader br = new BufferedReader(reader);
    - String buffer = null;
    - while((buffer = br.readLine()) != null) {
        syso(buffer);
    }

字符流写数据

- FileWriter writer = new FileWriter(filepath);
- writer.write("hello world"); //直接写字符串
- writer.flush();
- writer.close();

----------

字符码表

- 是一种可以方便计算机识别的特定字符集，它是将每一个字符和一个唯一的数字对应而形成的一张表
- 常见的码表有ASCII（美国，单字节）、ISO8859-1（欧洲）、GBK（中国，双字节）、unicode（统一，双字节）、UTF-8（目前最常用）
- 编码：把字符串转换成计算机能够识别的字节序列
- 解码：把字节序列转换成普通人能看懂的明文字符串
- 乱码是由于编码和解码方式不一致导致的

********

<h2 id="c2s4">第四节 对象的序列化</h2>

1. 掌握序列化的含义与意义
2. 掌握序列化读、写对象

----------

序列化接口

- 序列化（Serialization）是将对象的状态信息转换为可以存储或传输的形式的过程
- 主要目的：数据持久化，即数据的物理存储，比如保存到磁盘
- 反序列化：把持久化的数据恢复成原来的格式
- Java对象的序列化，必须实现序列化接口Serializable
    - 该接口没有任何接口方法，由JDK在底层实现
    - 序列化接口表明一种类型，JavaVM对序列化类型的对象进行额外的处理
    - 没有序列化的对象，在Java编程中保存的时候会抛出异常

----------

写对象

- 被写对象实现序列化接口
    - public class SerializedObject implements Serializable { ... }
- 确定对象写往的目的地，通常是文件或网络。然后采用过滤方式构造ObjectOutputStream，通过writeObject方法写对象
    - ObjectOutputStream os = new ObjectOutputStream(fileOutputStream);
    - os.writeObject(new SerializedObject())

读对象

- 确定读取对象的源，然后使用流过滤，得到ObjectInputStream对象，通过ObjectInputStream对象的readObject方法可以直接读取对象
    - ObjectInputStream is = new ObjectInputStream(fileInputStream);
    - SerializedObject o = (SerializedObject) is.readObject();

禁止某些字段序列化（节省空间）

- 使用transient修饰符号

----------

序列化版本管理

- JDK为了序列化的兼容性，引入了序列化版本管理的概念
- 读取比较旧的对象时，新版本必须定义与旧类一样的版本ID：serialVersionID
    - private static final long serialVersionUID = 1L;
    - 如果类中定义了serialVersionID，JDK的序列化系统会自动读取序列化的不同版本
    - 如果类中没有定义serialVersionID，需要使用工具查看版本，并手工处理在新的类中的定义与原来的版本相同

********

<h2 id="c2s5">第五节 线程的创建、运行和结束</h2>

1. 了解线程的含义与意义
2. 掌握线程的实现方式
3. 掌握线程的创建、运行和结束流程

----------

进程

- 在一个操作系统中，每个独立执行的程序都可称之为一个进程，也就是“正在运行的程序”
- 目前大部分计算机上安装的都是多任务操作系统，即能够同时执行多个应用程序

线程

- 每个运行的程序都是一个进程，在一个进程中可以有多个执行单元同时运行，这些执行单元可以看做程序执行的一条条线索，被称为线程
- 操作系统中的每一个进程中都至少存在一个线程，当一个Java程序启动时，就会产生一个进程，该进程默认创建一个线程，在线程上运行main()方法中的代码
- 单线程程序：代码按照调用顺序依次往下执行
- 多线程程序：多段程序代码交替运行，每个线程之间都是独立的，可以并发执行

----------

实现线程

- 两种方式
    - 继承Thread类，重写Thread中的run()方法
    - 实现Runnable接口，重写Runnable接口中的run()方法
- Thread也实现了Runnable接口，所以两种方式本质是一样的
    - 继承Thread类：线程实现方法与Thread类集成在一起
    - 实现Runnable接口：线程实现方式与Thread类分离开
- 推荐使用实现Runnable接口方式，可以避免由于Java的单继承带来的局限性

----------

创建线程

- Thread构造器
    - Thread() //对象*继承Thread类*时，创建对象即创建Thread对象
    - Thread(Runnable target) //target为*实现Runnable接口*的对象
    - Thread(Runnable target, String name)
- 继承Thread类
    - public class Object extends Thread {
        @Override //重写run()方法
        public void run() {
            ...
        }
    }
    - 调用线程
        - Object o1 = new Object(); //创建一个对象，该对象同时已经继承了Thread类，因此也是一个线程
        - Object o2 = new Object();
        - o1.start(); //默认执行run方法，多线程并发
        - o2.start();
- 实现Runnable接口
    - public class Object implements Runnable {
        @Override //重写run()方法
        public void run() {
            ...
        }
    }
    - 调用线程
        - Object o = new Object(); //创建Runnalbe接口的对象
        - Thread thread1 = new Thread(o); //创建Thread类，即一个线程
        - Thread thread2 = new Thread(o);
        - thread1.start(); //启动线程
        - thread2.start();

启动线程

- 在Thread类中，提供了一个start()方法用于启动线程
- 新线程启动后，系统会自动调用run()方法，如果子类重写了该方法，便会执行子类中的方法

结束线程

- 线程的run方法执行结束，就意味着线程结束
- 在run方法中使用return也可以结束线程 //本质上也是run方法执行结束了

********

<h2 id="c2s6">第六节 线程控制</h2>

1. 熟练掌握线程的生命周期、状态
2. 熟练掌握线程状态转换的开发
3. 掌握多线程安全问题初步处理方法

----------

线程的生命周期

- 五个阶段状态：新建状态（New）、就绪状态（Runnable）、运行状态（Running）、阻塞状态（Blocked）、死亡状态（Terminated）
- *新建状态* --start()--> *就绪状态* <--CPU使用权--> *运行状态* --run方法执行完/异常--> *死亡状态*
- *运行状态* --等待同步锁/调用IO阻塞方法/调用wait()/调用join()/调用sleep()--> *阻塞状态* --获得同步锁/阻塞IO返回/调用notify()/调用join()的线程终止/sleep()时间到--> *就绪状态*
- 新建状态（New）
    - 创建线程对象后，处于新建状态，此时不能运行，和其它Java对象一样，仅仅由JavaVM为其分配了内存，没有任何线程的动态特征
- 就绪状态（Runnable）
    - 当线程对象调用了start()方法后，线程进入就绪状态。处于就绪状态的线程位于可运行池中，此时它只是具备了运行的条件，能否获得CPU的使用权开始运行，需要等待系统的调度
- 运行状态（Running）
    - 如果处于就绪状态的线程获得了CPU的使用权，开始执行run()方法中的线程执行体，则该线程处于运行状态。一般来说当使用完系统分配的时间后，系统会剥夺线程占用的CPU资源，让其它线程活动执行的机会
- 阻塞状态（Blocked）
    - 一个正在执行的线程在某些特殊情况下，会放弃CPU的使用权，进入阻塞状态。线程进入阻塞状态后，就不能进入排队队列。只有当引起阻塞的原因被消除后，线程才可以转入就绪状态
        1. 当线程试图获取某个对象的同步锁时，如果该锁被其它线程所持有，则当前线程会进入阻塞状态
        2. 当线程调用了一个阻塞式的IO方法时
        3. 当线程调用了某个对象的wait()方法时
            - 调用notify()方法唤醒该线程
        4. 当线程调用了Thread的sleep(long millis)方法时
            - 线程睡眠时间到了以后，自动进入就绪状态
        5. 当一个线程中调用了另一个线程的join()方法时 //线程插队
            - 等待新加入的线程运行结束后，进入就绪状态
- 死亡状态（Terminated）
    - run方法中的代码执行完毕，线程进入死亡状态

----------

线程安全

- 由多线程操作共享资源时，可能会导致线程安全的问题
- 同步代码块解决多线程问题
    - 同步：线程一个一个的执行，对于共享的资源，只能有一个线程操作，其它的线程必须等待
    - 锁：如果想同步，需要一个同步锁，不同的线程对于锁要一致
    - synchronized(object) {
        ... //对于共享的数据进行锁定
    }

********

<h1 id="c3">第三章 JAVA操作数据库</h1>

********

<h2 id="c3s1">第一节 Linux系统下安装MySQL数据库</h2>

********

<h2 id="c3s2">第二节 MySQL数据库操作</h2>

********

<h2 id="c3s3">第三节 事务与隔离级别</h2>

********

<h2 id="c3s4">第四节 JDBC操作MySQL数据库</h2>