# 目录 #

- [第一节 Scala语言的函数式编程](#1)
- [第二节 Scala中的集合](#2)
- [第三节 Scala语言高级特性](#3)

***

<h4 id='1'>第一节 Scala语言的函数式编程</h4>

1. 掌握Scala的函数
2. 掌握Scala的匿名函数
3. 掌握Scala的闭包和柯里化
4. 掌握Scala常用的高阶函数

---

函数
- Scala中的函数非常重要
- 值函数：可以在变量中存放函数
```
def f(name:String):String = "Hello" + name
// def 函数名(参数名:参数类型):返回类型 = 函数返回值
```

匿名函数：没有名字的函数
```
(x:Int) => x*3
// (参数名:参数类型) => 函数返回值
Array(1,2,3).map((x:Int) => x*3) // 把匿名函数作为参数传递给map方法
```

高阶函数：带有函数参数的函数
```
def f1(f:(Double)=>Double) = f(10)
// def 高阶函数名(参数名:参数类型)) = 高阶函数返回值
// 其中(Double) => Double是一个匿名函数，作为高阶函数的参数
import scala.math._
f1(sqrt)
```

闭包和柯里化
- 闭包：函数的嵌套，即在定义一个函数的时候包含了另外一个函数的定义，并且在内函数中可以访问外函数的变量
```
def mulBy(factor:Double) = (x:Double)=>x*factor
val triple = mulBy(3)
val half = mulBy(0.5)
println(triple(10))
println(half(10))
```
- 柯里化(Curring)：把具有多个参数的函数转换为一条函数链，每个节点上的函数都是单一参数的函数
```
def add(x:Int, y:Int) = x + y
// 柯里化写法
def add(x:Int)(y:Int) = x + y
```

常用的高阶函数
- map：在列表中的每个元素上计算一个函数，并且返回一个包含相同数目元素的列表
- foreach：和map相似，只不过没有返回值，只是为了对参数进行作用
- filter：移除任何使得传入的函数返回false的元素
    - 返回Boolean类型的函数一般被称为断言函数
```
val numbers = List(1,2,3,4,5,6,7,8,9,10)
numbers.map((i:Int) => i*2)
numbers.foreach((i:Int) => i*2)
numbers.filter((i:Int) => i%2==0)
```
- zip：把两个列表的元素合成一个由元素对组成的列表里
```
scala> List(1,2,3).zip(List(4,5,6))
res0: List[(Int, Int)] = List((1,4), (2,5), (3,6))
```
- partition：根据断言函数的返回值对列表进行拆分
```
scala> numbers.partition((i:Int) => i%2==0)
res0: (List[Int], List[Int]) = (List(2, 4, 6, 8, 10),List(1, 3, 5, 7, 9))
```
- find：返回集合里的第一个匹配断言函数的元素
```
scala> numbers.find(_%3==0)
res0: Option[Int] = Some(3)
```
- flatten：把嵌套的结构分开
```
scala> List(List(1,2,3), List(4,5,6)).flatten
res0: List[Int] = List(1, 2, 3, 4, 5, 6)
```
- flatMap：常用的combinator，结合了map和flattern的功能
```
scala> List(List(1,2,3), List(4,5,6)).flatMap(x=>x.map(_*2))
res0: List[Int] = List(2, 4, 6, 8, 10, 12)
```

***

<h4 id='2'>第二节 Scala中的集合</h4>

1. 熟练掌握Scala中的各种集合的使用

---

可变集合/不可变集合（Map）
- scala.collectioin.immutable.Map
- scala.collectioin.mutable.Map
- 详见[Scala基础课程](./Scala基础课程.md)的映射相关知识
- 不可变集合，集合从不改变，可以安全地共享使用，甚至在多线程中也没问题

可变列表/不可变列表（List）
- scala.collection.immutable.List
- scala.collection.mutable.LinkedList
```
// 不可变列表
val nameList = List("Tom", "Mary", "Mike")
val intList = List(1,2,3,4,5)
val nullList:List[Nothing] = List()
val dim:List[List[Int]] = List(List(1,2,3),List(4,5,6))

println(nameList.head)
println(nameList.tail) // 返回除第一个元素的其他元素
println(nullList.isEmpty)

// 可变列表
val myList = scala.collection.mutable.LinkedList(1,2,3,4,5)
var cur = myList
while (cur != Nil) {
    cur.elem *= 2
    cur = cur.next
}
println(myList)
```

序列
- Vector
    - 支持快速的查找和更新
    ```
    val v = Vector(1,2,3,4,5)
    v.find(_>3) // 返回第一个满足条件的值
    v.updated(2,100)
    ```
- Range
    - 通常表示一个整数序列
    ```
    val r1 = Range(0,5)
    val r2 = (0 until 5)
    val r3 = (0 to 4)

    ('0' to '9') ++ ('A' to 'Z')
    ```

集（Set）
- 是不重复元素的集合
- 不保留元素插入的顺序
- 默认的实现方式：HashSet
    ```
    val s1 = Set(2,0,1)
    s1 + 1    --> Set(2,0,1)
    s1 + 3    --> Set(2,0,1,3)

    // 可排序的Set
    var s2 = scala.collection.mutable.SortedSet(2,0,1)    --> TreeSet(0,1,2)

    // 判断是否是另一个Set的子集
    Set(1,2).subsetOf(Set(1,2,3,4))

    // 并集、交集、差集
    var set1 = Set(1,2,3,4,5,6)
    var set2 = Set(5,6,7,8,9,10)
    set1 union set2 // 并集：去掉重复元素
    set1 intersect set2 // 交集：只保留重复元素
    set1 diff set2 // 差集：从set1中除去set2中有的元素
    ```

Scala中的模式匹配
- 应用场合：switch语句/类型检查
    ```
    // switch语句
    var sign = 0
    val ch1 = '-'
    ch1 match {
      case '+'  => sign = 1
      case '-'  => sign = -1
      case _    => sign = 0
    }
    println(sign)

    // Scala的守卫：匹配某种类型的所有值
    val ch2 = '6'
    var digit:Int = -1
    ch2 match {
      case _  if Character.isDigit(ch2) => digit = Character.digit(ch2, 10)
      case _    => println("others")
    }
    println(digit)

    // 模式匹配中使用变量
    val str3 = "Hello World"
    str3(7) match {
      case ch => println(ch) // ch:match匹配的变量
    }

    // 类型的匹配
    val v4:Any = 100
    v4 match {
      case x:Int  => println("Int")
      case s:String => println("String")
      case _  => println("Others")
    }

    // 匹配数组(Array)/列表(List)
    val myArray = Array(1,2,3)
    myArray match {
      case Array(0) => println("Array(0)")
      case Array(x,y) => println(x+y)
      case Array(x,y,z) => println(x+y+z)
      case Array(x,_*) => println("Others")
    }
    ```

样本类（CaseClass）
- 在普通类的定义前加case关键字，可以对这些类进行模式匹配
- case class最大的好处就是，它们支持模式匹配
- 一般写法
```
class Fruit

class Apple(name:String) extends Fruit
class Banana(name:String) extends Fruit

object caseClassDemo {
  def main(args: Array[String]): Unit = {
    val aApple:Fruit = new Apple("apple");
    val bBanana:Fruit = new Banana("banana");

    println(aApple.isInstanceOf[Fruit])
    println(aApple.isInstanceOf[Apple])
    println(bBanana.isInstanceOf[Apple])
  }
}
```
- caseclass写法
```
class Fruit

case class Apple(name:String) extends Fruit
case class Banana(name:String) extends Fruit

object caseClassDemo {
  def main(args: Array[String]): Unit = {
    val aFruit:Fruit = new Apple("apple");

    aFruit match {
      case Apple(name)  => println("apple")
      case Banana(name)  => println("banana")
      case _  => println("Others")
    }
  }
}
```

***

<h4 id='3'>第三节 Scala语言高级特性</h4>

1. 熟练掌握Scala中的泛型类型
2. 熟练掌握Scala中的泛型函数
3. 泛型的上界和下界
4. 视图界定
5. 协变和逆变
6. 隐式转换函数
7. 隐式参数
8. 隐式类

---

泛型类
- 和Java、C++类似，类和特质可以带类型参数
- 在Scala中使用方括号来定义类型参数
```
class GenericClass[T] {
  private var content:T = _

  def set(value:T) = {content = value}
  def get():T = {content}
}

object genericClassDemo {
  def main(args: Array[String]): Unit = {
    val i = new GenericClass[Int]
    i.set(10)
    println(i.get())

    val s = new GenericClass[String]
    s.set("hello world")
    println(s.get())
  }
}
```

泛型函数
- 函数和方法也可以带类型参数
- 和泛型类一样，我们需要把类型参数放在方法名之后
```
def mkIntArray(elem:Int*) = Array[Int](elem:_*)
mkIntArray(1,2,3)

def mkStrArray(elem:String*) = Array[String](elem:_*)
mkStrArray("Tom","Marry")

// 泛型函数
import scala.reflect.ClassTag // ClassTag代表运行时的信息，比如类型
def mkGenericArray[T:ClassTag](elem:T*) = Array[T](elem:_*)
mkGenericArray(1,2,3)
mkGenericArray("Tom","Marry")
```

泛型的上界和下界
- 规定泛型变量的取值范围
- 例：类的继承关系A-->B-->C-->D
    - 定义一个泛型变量T
    - 若D<:T<:B，则T只能是B/C/D
- 上界：S <: T，表示S的类型必须是T的子类或者本身
- 下界：U >: T，表示U的类型必须是T的父类或者本身
```
class Vehicle2 {
  def drive() = {println("Driving")}
}

class Car extends Vehicle2 {
  override def drive(): Unit = {println("Car")}
}

class Bike extends Vehicle2 {
  override def drive(): Unit = {println("Bike")}
}

object upperBoundsDemo {

  def takeVehicle[T <: Vehicle2](v:T) = {v.drive()}

  def main(args: Array[String]): Unit = {
    var v:Vehicle2 = new Vehicle2
    takeVehicle(v)

    var c:Car = new Car
    takeVehicle(v)

//    var a:apple = new Apple
//    takeVehicle(a)
  }
  
}
```

视图界定（View bounds）
- 比上界适用的范围更加广泛，除了可以使用所有的子类型之外，还可以允许使用能够通过隐式转换过去的类型
- 使用“<%”
    - 可以接受String和String子类（同上界）
    - 可以接受能够转换成String类型的其他类型
        - 转换规则：隐式转换函数
        - 转换规则必须定义
```
scala> def addTwoString[T <: String](x:T, y:T) = {println(x+y)}
addTwoString: [T <: String](x: T, y: T)Unit

scala> addTwoString("abc","xyz")
abcxyz

scala> addTwoString(100,200)
<console>:14: error: inferred type arguments [Int] do not conform to method addT
woString's type parameter bounds [T <: String]
       addTwoString(100,200)
       ^
<console>:14: error: type mismatch;
 found   : Int(100)
 required: T
       addTwoString(100,200)
                    ^
<console>:14: error: type mismatch;
 found   : Int(200)
 required: T
       addTwoString(100,200)
                        ^
```
```
scala> def addTwoString[T <% String](x:T, y:T) = {println(x+y)}
addTwoString: [T](x: T, y: T)(implicit evidence$1: T => String)Unit

scala> addTwoString(100,200)
<console>:14: error: No implicit view available from Int => String.
       addTwoString(100,200)
                   ^

scala> implicit def int2String(n:Int):String = {n.toString}
warning: there was one feature warning; re-run with -feature for details
int2String: (n: Int)String

scala> addTwoString(100,200)
100200
```

协变和逆变
- 协变：在类型参数的前面加上“+”
    - 泛型变量的值可以是本身的类型或者其子类的类型
- 逆变：在类型参数的前面加上“-”
    - 泛型变量的值可以是本身的类型或者其父类的类型

隐式转换函数
- 以implicit关键字申明的带有单个参数的函数
- 会由scala自动调用
- 参考视图界定中Int转String的样例

隐式参数
- 使用implicit申明的参数
```
def testParam(implicit name:String) = {println(name)}

implicit val name:String = "Hello World"

testParam
```
- 可以使用隐式参数完成隐式转换
```
def smaller[T](a:T, b:T)(implicit order:T=> Ordered[T]) = if (a < b) a else b

smaller(100,23)
smaller("hello","world")
```

隐式类
- 对类增加implicit限定的类，对类的功能加强
    ```
    object implicitClassDemo {
        implicit class Calc (x:Int) {
            def add(y:Int):Int = x + y
        }

        def main(args: Array[String]): Unit = {
            println(1.add(2))
        }
    }
    ```
    1. 寻找一个隐式类，把Int转换为Calc
    2. 调用Calc.add