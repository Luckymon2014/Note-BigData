# 目录 #

- [第一节 Scala语言入门](#1)
- [第二节 Scala编程基础](#2)
- [第三节 Scala面向对象基础](#3)
- [第四节 Scala面向对象高级编程](#4)

***

<h4 id='1'>第一节 Scala语言入门</h4>

Scala编程语言
- 是一种多范式的编程语言，设计的初衷是要集成面向对象编程和函数式编程的各种特性，支持多种模式的编程
    - 面向对象
    - 函数式编程：Scala最大特点
- 基于JVM，兼容现有的Java程序

Scala开发环境搭建
- 安装JDK
- 安装Scala
    - 解压
    - 环境变量：SCALA_HOME、PATH

Scala开发工具
- 命令行
    - REPL
        - 进入：scala
        - 退出：:quit
- IDE
    - Scala IDE：基于Eclipse
    - IDEA

Scala的基本数据类型
- 任何数据都是对象
    - 例：数字1，在Scala中是一个对象，具有方法（函数）
    - 1.toString
- 可以不用指定数据类型，Scala会自动进行类型的推倒
    - 例：val a:Int = 10可以写作val b = 10
- 数值类型
    - Byte：8位有符号的数字，-128 ~ 127
    - Short：16位有符号的数字，-32768 ~ 32767
    - Int：32位有符号的数字
    - Long：64位有符号的数字
    - Float
    - Double
- 字符/字符串类型
    - Char
    - String
        - 插入操作
        ```
        scala> s"My name is Tom and ${s1}"
        res1: String = My name is Tom and Hello World
        ```
- Unit类型
    - 相当于Java的void
    ```
    scala> val f=()    -->相当于定义了函数f，返回void
    f: Unit = ()
    ```
- Nothing类型
    - 一般表示在执行过程中，产生了Exception
    ```
    scala> def myFunction=throw new Exception("Error!")
    myFunction: Nothing
    ```

变量和常量
- val：常量
- var：变量

函数
- scala是函数式编程，因此函数至关重要
- 内置函数
    ```
    scala> import scala.math._    -->"_"类似"*"，代表导入math下的所有方法
    import scala.math._

    scala> max(1,2)
    res0: Int = 2
    ```
- 自定义函数：def关键字
    - 求和
    ```
    scala> def sum(x:Int,y:Int):Int = x+y
    sum: (x: Int, y: Int)Int

    scala> sum(1,2)
    res0: Int = 3
    ```
    - 阶乘：递归算法
    ```
    scala> def myFactor(x:Int):Int = {
     | if(x<=1)
     |      1
     | else
     |      myFactor(x-1)*x
     | }
    myFactor: (x: Int)Int

    scala> myFactor(5)
    res0: Int = 120
    ```
- 注：scala函数中，最后一句话即函数的返回值

***

<h4 id='2'>第二节 Scala编程基础</h4>

条件表达式：if...else...
- 参考阶乘实现方式

循环:for/while/do...while.../foreach(迭代)
```
    var list = List("zhangsan", "lisi", "wangwu")

    // for
    println("=========================")
    for (s <- list) // "<-"：提取符，提取list中的值赋予s
      println(s)

    println("=========================")
    for {
      s <- list
      if (s.length > 4)
    } {
      println(s)
    }

    println("=========================")
    for (s <- list if s.length > 4)
      println(s)

    // 使用yield关键字生成一个新的集合
    println("=========================")
    var newList = for {
      s <- list
      s1 = s.toUpperCase()
    } yield(s1)
    for (s <- newList)
      println(s)

    // while
    println("*************************")
    var i = 0
    while (i < list.length) {
      println(list(i))
      i += 1 // 不能写作i++
    }

    // do...while...
    println("*************************")
    var j = 0
    do {
      println(list(j))
      j += 1
    } while(j < list.length)

    // foreach
    println("*************************")
    list.foreach(println) // foreach接受了另一个函数println作为值
```

Scala的函数参数
- 函数参数的求值策略
    - call by value：对函数的实参进行求值，并且只求一次
        - 定义：":"
        ```
        def func1(x:Int, y:Int):Int = x + x
        func1(3+4, 8)
        ```
        - 执行过程
            1. func1(7,8)
            2. 7 + 7
            3. 14
    - call by name：对函数实参在函数内部调用的时候进行求值，每次调用都会求值
        - 定义：": =>"
        ```
        def func2(x: =>Int, y: =>Int):Int = x + x
        func2(3+4, 8)
        ```
        - 执行过程
            1. (3+4) + (3+4)
            2. 7 + (3+4)
            3. 7 + 7
            4. 14
- 函数参数的类型
    - 默认参数
        ```
        def func1(name:String="Tom"):String = "Hello " + name
        func1()
        func1("Mary")
        ```
    - 代名参数
        ```
        def func2(str:String="Good Morning", name:String="Tom", age:Int=20) = str + name + age
        func2(name="Mary")
        ```
    - 可变参数
        ```
        def sum(args:Int*) = {
            var result = 0
            for (arg <- args)
                result += arg
            result
        }

        sum(1,2,3)
        sum(1,2,3,4,5)
        ```

Scala的lazy（懒值）
- 当一个变量被申明为lazy，他的初始化会被推迟，直到第一次使用他的时候
```
scala> val x:Int = 10
x: Int = 10

scala> lazy val y:Int = x + 1
y: Int = <lazy>

scala> y
res1: Int = 11
```

异常处理
- Scala异常处理机制类似Java，使用throw抛出异常
```
try {
    val words = scala.io.Source.fromFile("1.txt").mkString
} catch {
    case exl:java.io.FileNotFoundException => {
        println(exl)
    }
    case _:Exception => {
        println("Other Exception")
    }
} finally {
    println("finally")
}
```

数组
- 定长数组：Array
    ```
    val a = new Array[Int](10)
    val b = new Array[String](5)
    val c = Array("zhangsan", "lisi", "wangwu")
    ```
- 变长数组：ArrayBuffer
    ```
    import scala.collection.mutable._
    val d = ArrayBuffer[Int]()
    d += 1
    d += 2
    d += 3
    d += (10,20,30)
    d.trimEnd(2) // 删除数组中最后两个元素
    ```
```
import scala.collection.mutable.ArrayBuffer

object array {
  def main(args: Array[String]): Unit = {
    var a = Array("zhangsan", "lisi", "wangwu")

    for (n <- a)
      println(n)

    a.foreach(println)

    println("=================================")
    val b = Array(1,10,2,3,8,7)
    println(b.max)
    println(b.min)
    println(b.sum)

    println("=================================")
    val c = ArrayBuffer(1,10,2,3,8,7)
    val c1 = c.sortWith(_>_)
    c1.foreach(println)
    val c2 = c.sortWith(_<_)
    c2.foreach(println)
  }
}
```

映射和元祖
- 映射：就是Map集合，由一个KV组成
    - 可变的Map
    ```
    val scores = Map("Tom"->80, "Mary"->90)
    ```
    - 不可变的Map
    ```
    val scores = scala.collectioin.immutable.Map("Tom"->80, "Mary"->90)
    ```
    - 常见操作
        - 获取Map值
        ```
        scores("Mary")
        scores("Jerry")    -->Exception

        if (scores.contains("Jerry))
            scores("Jerry")
        else
            -1

        scores.getOrElse("Jerry",-1)

        scores.foreach(println)
        ```
        - 更新Map值
        ```
        scores("Marry") = 100
        scores += "Jerry"->88
        ```
- 元祖（Tuple）：是不同类型的值的集合
    ```
    val t1 = (1, 3.14, "Hello")
    val t2 = new Tuple3(1, 3.14, "Hello")
    t2._1
    t2._2
    t2._3
    t2.productIterator.foreach(println)
    ```

***

<h4 id='3'>第三节 Scala面向对象基础</h4>

面向对象
- 把数据和数据的操作放在一起，作为一个整体
- 特性：封装、继承、多态
```
class Student {

  private var name:String="Tom"
  private var age:Int=20

  def getName():String = name
  def setName(name:String) = this.name = name

  def getAge():Int = age
  def setAge(age:Int) = this.age = age

}

object Student {
  def main(args: Array[String]): Unit = {
    var s1 = new Student()
    println(s1.getName() + s1.getAge())
    println(s1.name + s1.age) //为什么private的属性可以直接访问？
  }
}
```
- 属性的get和set方法
    - 当定义属性的时候，如果是private类型，scala会自动为其生成对应的get和set方法
        private var name:String = "Tom"
        - get方法：name
        - set方法：name_
    - 希望只有get方法，没有set方法
        - 可以定义为一个常量
        private val name:String = "Tom"
    - 希望只属于类私有，不希望生成get和set方法
        - 使用[this]关键字
        private[this] val name:String = "Tom"

内部类（嵌套类）
- 在类的内部定义一个类
```
import scala.collection.mutable.ArrayBuffer

class Student3 {
  private var name:String="Tom"
  private val age:Int=20

  private var courseList = new ArrayBuffer[Course]()

  def addNewCourse(cname:String, credit:Int): Unit = {
    var c1 = new Course(cname, credit)
    courseList += c1
  }

  class Course(val courseName:String, val credit:Int) {
    //...
  }
}

object Student3 {
  def main(args: Array[String]): Unit = {
    var s = new Student3
    s.addNewCourse("Chinese",4)
    s.addNewCourse("English",3)
    s.addNewCourse("Math",5)

    println(s.name + s.age)
    for (c <- s.courseList)
      println(c.courseName + c.credit)
  }
}
```

类的构造器
- 主构造器
    - 和类的声明在一起，只能有一个主构造器
    - 具有对应的get和set方法
- 辅助构造器
    - 可以有多个辅助构造器
    - 使用关键字this
```
class Student4(val name:String, val age:Int) { // 主构造
  def this(age:Int) { // 辅助构造器
    // 调用主构造器
    this("no name", age)
  }
}

object Student4 {
  def main(args: Array[String]): Unit = {
    var s = new Student4("Tom", 20)
    println(s.name + s.age)

    var s2 = new Student4(25)
    println(s2.name + s2.age)
  }
}
```

***

<h4 id='4'>第四节 Scala面向对象高级编程</h4>

1. 掌握Scala中的Object对象
2. Scala中的apply方法
3. Scala中的继承
4. Scala中的trait（特质）
5. 包的使用
6. 包对象

---

Scala中的Object对象
- Scala中没有static修饰符，但Object对象中，所有的成员都可以看成是静态的
- 如果Object对象和class重名，表示这个Object对象是这个类的伴生对象
- Scala中的Object对象，相当于Java中的静态块（static）
- 单例模式：一个类只有一个对象
    ```
    object CreditCard {

    private[this] var creditCardNum:Long = 0

    def generateNewCCNum():Long = {
        creditCardNum += 1
        creditCardNum
    }

    def main(args: Array[String]): Unit = {
        println(CreditCard.generateNewCCNum())
        println(CreditCard.generateNewCCNum())
        println(CreditCard.generateNewCCNum())
    }

    }
    ```
- 使用应用程序对象：可以省略main方法
    ```
    object HelloWorld extends App {
        println("hello, world")
    }
    ```

Scala中的apply方法
- 遇到如下形式的表达式时，apply方法就会被调用
    - Object(arg0, arg1, ..., argN)
    ```
    var myArray = Array(1,2,3) // 没有使用new关键字，其实就是使用了apply方法
    ```
- 通常，这样一个apply方法返回的是伴生类的对象
- 作用：省略new关键字，用于解决复杂对象的初始化问题
```
class Student5(val name:String)

object Student5 {
  // apply方法需要定义在伴生对象中
  def apply(name: String) = {
    new Student5(name)
  }

  def main(args: Array[String]): Unit = {
    var s = new Student5("Tom")
    println(s.name)

    // 如果没有apply方法，创建Mary会报错
    var s2 = Student5("Mary")
    println(s2.name)
  }
}
```

继承
- 和Java一样，使用关键字extends继承
- 基本的继承
- 子类重写父类的方法：使用关键字override
- 匿名子类：子类没有名字
```
class Person(val name:String, val age:Int) {
  def sayHello():String = "Hello " + name + " and the age is " + age
}

// 子类：如果希望使用子类中的值去覆盖父类中的值/方法，需要使用override
class Employee(override val name:String,override val age:Int, val salary:Int) extends Person(name, age) {
  override def sayHello():String = name + "'s Salary is " + salary
}

object extendsDemo {
  def main(args: Array[String]): Unit = {
    val p1:Person = new Person("Tom", 20)
    println(p1.sayHello())

    val p2:Person = new Employee("Mary", 21, 3000)
    println(p2.sayHello())

    // 创建Person的一个匿名子类
    val p3:Person = new Person("Mike", 24) {
      override def sayHello():String = "Hello " + name
    }
    println(p3.sayHello())
  }
}
```
- 抽象类：包含抽象方法，只能用来继承
```
abstract class Vehicle {
  def checkType():String
}

class Car extends Vehicle {
  def checkType():String = {
    "This is a Car"
  }
}

object extendsDemo2 {
  def main(args: Array[String]): Unit = {
    var v:Vehicle = new Car
    println(v.checkType())
  }
}
```
- 抽象字段：没有初始值的字段
```
abstract class Person {
  var name:String
}

// 子类应该提供父类抽象字段的初始值，否则也应该定义为抽象类
abstract class Employee1 extends Person {
}
class Employee2 extends Person {
  override var name: String = "no name"
}
```

trait（特质）
- trait就是抽象类
- trait和抽象类的区别：trait支持多重继承
```
trait Human {
  val id:Int
  val name:String

  def sayHello():String = "Hello " + name
}

trait Action {
  def getActionName():String
}

class Student6(val id:Int, val name:String) extends Human with Action {
  def getActionName(): String = "Action is Running"
}

object traitDemo {
  def main(args: Array[String]): Unit = {
    var s = new Student6(1, "Tom")
    println(s.sayHello())
    println(s.getActionName())
  }
}
```

包
- 和Java中的包或者C++中的命名空间的目的是相同的
- 管理大型程序中的名称
- package可以嵌套
- 包的引入：import
    - Scala中的import可以写在任意地方
    - Java中的import需要写在class的最前面
- 包对象
    - Java虚拟机的局限：包可以包含类、对象和特质，但不能包含函数或者变量的定义
    - Scala中的包对象：变量、常量、方法、类、对象、trait、包