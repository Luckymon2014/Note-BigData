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

***

<h4 id='3'>第三节 Scala语言高级特性</h4>
