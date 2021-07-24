# Learning Play Framework

最近需要使用HBase和elasticsearch来模仿google做一个网站，前端准备用bootstrap，后端我一开始准备用
django，但是发现没有django与hbase和elasticsearch的集成方案，所以我就找到了这里，主要是这个框架
可以使用scala而不使用java，笔者只是不想学自己学过的东西。

Requirements:
* Java.18 or later
* sbt

I have installed the jdk1.8, so I just need install sbt.

On mac:
```shell
$ brew install sbt
```

Verifying java.
```shell
$ java --version
java version "1.8.0_291"
Java(TM) SE Runtime Environment (build 1.8.0_291-b10)
Java HotSpot(TM) 64-Bit Server VM (build 25.291-b10, mixed mode)
```

I don't want to use Play by java, so I choice scala.

Create a New Application:
```shell
$ sbt new playframework/play-scala-seed.g8
```

1. Enter `sbt run` will download dependencies and start the system.
2. Open a browser, enter http://localhost:9000/ to view the welcome page.

Play application layout:
```shell
app                      → Application sources
 └ assets                → Compiled asset sources
    └ stylesheets        → Typically LESS CSS sources
    └ javascripts        → Typically CoffeeScript sources
 └ controllers           → Application controllers
 └ models                → Application business layer
 └ views                 → Templates
build.sbt                → Application build script
conf                     → Configurations files and other non-compiled resources (on classpath)
 └ application.conf      → Main configuration file
 └ routes                → Routes definition
dist                     → Arbitrary files to be included in your projects distribution
public                   → Public assets
 └ stylesheets           → CSS files
 └ javascripts           → Javascript files
 └ images                → Image files
project                  → sbt configuration files
 └ build.properties      → Marker for sbt project
 └ plugins.sbt           → sbt plugins including the declaration for Play itself
lib                      → Unmanaged libraries dependencies
logs                     → Logs folder
 └ application.log       → Default log file
target                   → Generated stuff
 └ resolution-cache      → Info about dependencies
 └ scala-2.13
    └ api                → Generated API docs
    └ classes            → Compiled class files
    └ routes             → Sources generated from routes
    └ twirl              → Sources generated from templates
 └ universal             → Application packaging
 └ web                   → Compiled web assets
test                     → source folder for unit or functional tests
```

我只是想用java来提供API给python的django，用户输入文本搜索，然后django使用该文本向java搜索结果，java
再搜索elasticsearch然后将结果返回给django。这里需要维护java，hbase和elasticsearch的索引。

## Install scala
但是我这里准备使用scala而非java，所以，我必须先安装scala.
```shell
$ brew install coursier/formulas/coursier
$ cs setup
```

正在等待安装中，所以这里闲扯一下，笔者只是想将搜索服务做成rest api的，大概api这么设计:
```text
/api/v1/search_engine/?keyword={用户搜索的句子}
```

然后，就是hbase和elasticsearch的集成，也许读者会问，为何我不用django搜索elasticsearch，你们大概
是不知道那个haystack最高支持版本才5，elasticsearch7才刚刚加进去，根本不能用，想了想，感觉坑特别多，还是
考虑换大数据的方案。

我这里由于是做google仿站，所以边做技术研讨，边思考与自己的应用需求能不能匹配。

爬虫抓取网页，然后将标题，url，简介，网页指纹，分数，点击量存入存入hbase，每存一个就更新索引到elasticsearch
中，这样就变成了实时的搜索，或者说每天一更新索引也可以，也比较合适，因为网页毕竟不是每天都改变的数据。

简单的业务逻辑确定之后就比较容易了。

然后回到我们的scala安装，看看装完了没。

上一步`cs setup`官方文档上说会安装默认以下工具:

1. 一个jdk
2. sbt和mill构建工具
3. ammonite
4. scalafmt
5. coursier cli
6. scala和scalac 

运行交互式命令行创建一个scala的项目:
```shell
$ sbt new scala/scala3.g8.
```

这个会提示你输入项目的名字，输入之后按确认就会创建项目，这个据说是从github上拉取项目模版。

运行项目:
```shell
$ cd hello
# 运行sbt。这将打开 sbt 控制台。
$ sbt
# 键入~run。这~是可选的，会导致 sbt 在每次文件保存时重新运行，从而实现快速的编辑/运行/调试周期。sbt 还会生成一个target目录供自己使用，您可以忽略它。
> ~run
# 完成此项目的试验后，按[Enter]可中断run命令。然后键入exit或按[Ctrl][d]退出 sbt 并返回到命令行提示符。
> run
[info] running hello
Hello world!
I was compiled by Scala 3. :)
[success] Total time: 0 s, completed 2021年7月24日下午1:52:21
```

我也不知道好不好，好像进入了一个大坑。成功打印出了Hello world!

接下来是scala语言的学习，这里有官方的教程：
[scala book](https://docs.scala-lang.org/scala3/book/introduction.html).

但是，笔者是为了使用Play 框架开发API的。所以，笔者直接跳过，直奔主题了。

但是，最少需要了解scala语言到怎么倒入包的知识点那里。不然就不会开发scala程序。那么来吧。

w3c菜鸟教程已经过时了，必须取官网学。

## Learning scala by the fastest way

Hello world!
```scala
@man def hello = println("Hello, world!")
```

这里通过@main注解将hello这个方法声明为main方法，其它就没什么好说的了。

由于这里我们用的sbt来运行，所以就不用 scala来测试了。

Two types of variables.

1. val 定义常量，类似final(Java)
2. var 定义变量

```scala
val a = 0
var b = 1
```

```scala
val msg = "Hello, world"
msg = "Aloha"   // "reassignment to val" error; this won’t compile
```
上面会导致编译错误

```scala
val x: Int= 1
val x = 1
```
这里可以指定类型，也可以这样。编译器会推断类型

基本类型
```scala
val b: Byte = 1
val i: Int = 1
val l: Long = 1
val s: Short = 1
val d: Double = 2.0
val f: Float = 3.0

val i = 123   // defaults to Int
val j = 1.0   // defaults to Double

val x = 1_000L   // val x: Long = 1000
val y = 2.2D     // val y: Double = 2.2
val z = 3.3F     // val z: Float = 3.3

var a = BigInt(1_234_567_890_987_654_321L)
var b = BigDecimal(123_456.789)

val name = "Bill"   // String
val c = 'a'         // Char

val firstName = "John"
val mi = 'C'
val lastName = "Doe"

//只需在字符串前面加上字母 s，然后在字符串内的变量名称之前放置一个 $ 符号。
//要将任意表达式嵌入字符串中，请将它们括在花括号中：
println(s"Name: $firstName $mi $lastName")   // "Name: John C Doe"
println(s"2 + 2 = ${2 + 2}")   // prints "2 + 2 = 4"

val x = -1
println(s"x.abs = ${x.abs}")   // prints "x.abs = 1"

//和python甚至比python更舒服的多行字符串
val quote = """The essence of Scala:
               Fusion of functional and object-oriented
               programming in a typed setting."""
```

条件语句
```scala
// 条件判断
if x < 0 then
  println("negative")
else if x == 0 then
  println("zero")
else
  println("positive")

// 一行的条件表达式
val x = if a < b then a else b

// for循环
val ints = List(1, 2, 3, 4, 5)
for i <- ints do println(i)

for
  i <- ints
  if i > 2
do
  println(i)
  
for
  i <- 1 to 3
  j <- 'a' to 'c'
  if i == 2
  if j == 'b'
do
  println(s"i = $i, j = $j")   // prints: "i = 2, j = b"
  
// yield用法，这里很特别python里是这样写
// doubles = [i * 2 for i in ints]
val doubles = for i <- ints yield i * 2

val names = List("chris", "ed", "maurice")
val capNames = for name <- names yield name.capitalize

// 多行的yield的写法
val fruits = List("apple", "banana", "lime", "orange")

val fruitLengths = for
  f <- fruits
  if f.length > 4
yield
  // you can use multiple lines
  // of code here
  f.length

// fruitLengths: List[Int] = List(5, 6, 6)

// switch, python里没有，但是可以实现通过字典
val i = 1

// later in the code ...
i match
  case 1 => println("one")
  case 2 => println("two")
  case _ => println("other")
  
val result = i match
  case 1 => "one"
  case 2 => "two"
  case _ => "other"
  
val p = Person("Fred")

// later in the code
p match
  case Person(name) if name == "Fred" =>
    println(s"$name says, Yubba dubba doo")

  case Person(name) if name == "Bam Bam" =>
    println(s"$name says, Bam bam!")

  case _ => println("Watch the Flintstones!")
  
 // getClassAsString is a method that takes a single argument of any type.
def getClassAsString(x: Matchable): String = x match
  case s: String => s"'$s' is a String"
  case i: Int => "Int"
  case d: Double => "Double"
  case l: List[_] => "List"
  case _ => "Unknown"

// examples
getClassAsString(1)               // Int
getClassAsString("hello")         // 'hello' is a String
getClassAsString(List(1, 2, 3))   // List

// try  catch finaly
try
  writeTextToFile(text)
catch
  case ioe: IOException => println("Got an IOException.")
  case nfe: NumberFormatException => println("Got a NumberFormatException.")
finally
  println("Clean up your resources here.")
  
// while
while x >= 0 do x = f(x)

var x = 1

while
  x < 3
do
  println(x)
  x += 1
```
条件语句是写逻辑的重要工具，必须掌握。

面向对象领域建模
```scala
// 接口的定义
trait Speaker:
  def speak(): String  // has no body, so it’s abstract

trait TailWagger:
  def startTail(): Unit = println("tail is wagging")
  def stopTail(): Unit = println("tail is stopped")

trait Runner:
  def startRunning(): Unit = println("I’m running")
  def stopRunning(): Unit = println("Stopped running")
  
// 实现接口
class Dog(name: String) extends Speaker, TailWagger, Runner:
  def speak(): String = "Woof!"

// 定义类，集成接口，并重写接口的方法
class Cat(name: String) extends Speaker, TailWagger, Runner:
  def speak(): String = "Meow"
  override def startRunning(): Unit = println("Yeah ... I don’t run")
  override def stopRunning(): Unit = println("No need to stop")
  
// 实例化，并调用方法
val d = Dog("Rover")
println(d.speak())      // prints "Woof!"

val c = Cat("Morris")
println(c.speak())      // "Meow"
c.startRunning()        // "Yeah ... I don’t run"
c.stopRunning()         // "No need to stop"

//类
class Person(var firstName: String, var lastName: String):
  def printFullName() = println(s"$firstName $lastName")

val p = Person("John", "Stephens")
println(p.firstName)   // "John"
p.lastName = "Legend"
p.printFullName() 

// 定义枚举
enum CrustSize:
  case Small, Medium, Large

enum CrustType:
  case Thin, Thick, Regular

enum Topping:
  case Cheese, Pepperoni, BlackOlives, GreenOlives, Onions

//引用枚举  
import CrustSize.*
val currentCrustSize = Small

// enums in a `match` expression
currentCrustSize match
  case Small => println("Small crust size")
  case Medium => println("Medium crust size")
  case Large => println("Large crust size")

// enums in an `if` statement
if currentCrustSize == Small then println("Small crust size")

enum Nat:
  case Zero
  case Succ(pred: Nat)
```

Scala 案例类允许您使用不可变数据结构对概念进行建模。案例类具有类的所有功能，并且还具有使它们对函数式编程有用的附加特性。当编译器在类前面看到 case 关键字时，它具有以下效果和好处：

* Case类构造函数参数默认是public val字段，所以字段是不可变的，并且为每个参数生成访问器方法。
* 生成了一个 unapply 方法，它允许您在匹配表达式中以更多方式使用案例类。
* 在类中生成了一个复制方法。这提供了一种在不更改原始对象的情况下创建对象的更新副本的方法。
* 生成 equals 和 hashCode 方法来实现结构相等。
* 生成默认的 toString 方法，有助于调试。

```shell
// define a case class
case class Person(
  name: String,
  vocation: String
)

// create an instance of the case class
val p = Person("Reginald Kenneth Dwight", "Singer")

// a good default toString method
p                // : Person = Person(Reginald Kenneth Dwight,Singer)

// can access its fields, which are immutable
p.name           // "Reginald Kenneth Dwight"
p.name = "Joe"   // error: can’t reassign a val field

// when you need to make a change, use the `copy` method
// to “update as you copy”
val p2 = p.copy(name = "Elton John")
p2               // : Person = Person(Elton John,Singer)
```

方法
```shell
def methodName(param1: Type1, param2: Type2): ReturnType =
  // the method body
  // goes here
  
// 不定义返回值的方法
def sum(a: Int, b: Int): Int = a + b
def concatenate(s1: String, s2: String): String = s1 + s2

// 定义返回值的方法
def sum(a: Int, b: Int) = a + b
def concatenate(s1: String, s2: String) = s1 + s2

val x = sum(1, 2)
val y = concatenate("foo", "bar")

//  另外一种定义返回值的方法
def getStackTraceAsString(t: Throwable): String =
  val sw = new StringWriter
  t.printStackTrace(new PrintWriter(sw))
  sw.toString
  
// 定义参数默认值
def makeConnection(url: String, timeout: Int = 5000): Unit =
  println(s"url=$url, timeout=$timeout")
  
// 和python一样
makeConnection("https://localhost")         // url=http://localhost, timeout=5000
makeConnection("https://localhost", 2500)   // url=http://localhost, timeout=2500

makeConnection(
  url = "https://localhost",
  timeout = 2500
)

//扩展方法允许您向封闭类添加新方法。
//例如，如果要向 String 类添加两个名为 hello 和 aloha 的方法，请将它们声明为扩展方法：
extension (s: String)
  def hello: String = s"Hello, ${s.capitalize}!"
  def aloha: String = s"Aloha, ${s.capitalize}!"

"world".hello    // "Hello, World!"
"friend".aloha   // "Aloha, Friend!"

extension (s: String)
  def makeInt(radix: Int): Int = Integer.parseInt(s, radix)

"1".makeInt(2)      // Int = 1
"10".makeInt(2)     // Int = 2
"100".makeInt(2)    //
```

一些函数
```shell
// map方法使用
// If you haven’t seen the map method before, it applies a given 
// function to every element in a list, yielding a new list that 
// contains the resulting values.

val a = List(1, 2, 3).map(i => i * 2)   // List(2,4,6)
val b = List(1, 2, 3).map(_ * 2)        // List(2,4,6)

def double(i: Int): Int = i * 2
val a = List(1, 2, 3).map(i => double(i))   // List(2,4,6)
val b = List(1, 2, 3).map(double)           // List(2,4,6)

// 当您使用不可变的集合（如 List、Vector 以及不可变的 Map 和 Set 类）
// 时，重要的是要知道这些函数不会改变它们被调用的集合；相反，它们返回一个
// 带有更新数据的新集合。因此，以“流畅”的方式将它们链接在一起来解决问题也很常见
val nums = (1 to 10).toList   // List(1,2,3,4,5,6,7,8,9,10)

// methods can be chained together as needed
val x = nums.filter(_ > 3)
            .filter(_ < 7)
            .map(_ * 10)

// result: x == List(40, 50, 60)
```

在 Scala 中，object 关键字创建一个 Singleton 对象。换句话说，一个对象定义了一个只有一个实例的类。
对象有多种用途：

* 它们用于创建实用方法的集合。
* 伴生对象是一个与它共享文件的类同名的对象。在这种情况下，该类也称为伴生类。
* 它们用于实现特征以创建模块。
```shell
object StringUtils:
  def isNullOrEmpty(s: String): Boolean = s == null || s.trim.isEmpty
  def leftTrim(s: String): String = s.replaceAll("^\\s+", "")
  def rightTrim(s: String): String = s.replaceAll("\\s+$", "")

// 就像调用类的静态方法一样调用  
val x = StringUtils.isNullOrEmpty("")    // true
val x = StringUtils.isNullOrEmpty("a")   // false
```

伴随对象
```shell
//此示例演示了伴随类中的 area 方法如何访问其伴随对象中的私有 calculateArea 方法：

import scala.math.*

class Circle(radius: Double):
  import Circle.*
  def area: Double = calculateArea(radius)

object Circle:
  private def calculateArea(radius: Double): Double =
    Pi * pow(radius, 2.0)

val circle1 = Circle(5.0)
circle1.area   // Double = 78.53981633974483
```

Creating modules from traits
```shell
trait AddService:
  def add(a: Int, b: Int) = a + b

trait MultiplyService:
  def multiply(a: Int, b: Int) = a * b

// implement those traits as a concrete object
object MathService extends AddService, MultiplyService

// use the object
import MathService.*
println(add(1,1))        // 2
println(multiply(2,2))   // 4
```

数据结构
```shell
// 创建列表
val a = List(1, 2, 3)           // a: List[Int] = List(1, 2, 3)

// Range methods
val b = (1 to 5).toList         // b: List[Int] = List(1, 2, 3, 4, 5)
val c = (1 to 10 by 2).toList   // c: List[Int] = List(1, 3, 5, 7, 9)
val e = (1 until 5).toList      // e: List[Int] = List(1, 2, 3, 4)
val f = List.range(1, 5)        // f: List[Int] = List(1, 2, 3, 4)
val g = List.range(1, 10, 3)    // g: List[Int] = List(1, 4, 7)

// 列表方法调用
// a sample list
val a = List(10, 20, 30, 40, 10)      // List(10, 20, 30, 40, 10)

a.drop(2)                             // List(30, 40, 10)
a.dropWhile(_ < 25)                   // List(30, 40, 10)
a.filter(_ < 25)                      // List(10, 20, 10)
a.slice(2,4)                          // List(30, 40)
a.tail                                // List(20, 30, 40, 10)
a.take(3)                             // List(10, 20, 30)
a.takeWhile(_ < 30)                   // List(10, 20)

// flatten
val a = List(List(1,2), List(3,4))
a.flatten                             // List(1, 2, 3, 4)

// map, flatMap
val nums = List("one", "two")
nums.map(_.toUpperCase)               // List("ONE", "TWO")
nums.flatMap(_.toUpperCase)           // List('O', 'N', 'E', 'T', 'W', 'O')

val firstTen = (1 to 10).toList            // List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

firstTen.reduceLeft(_ + _)                 // 55
firstTen.foldLeft(100)(_ + _)              // 155 (100 is a “seed” value)

// 元祖
case class Person(name: String)
val t = (11, "eleven", Person("Eleven"))
t(0)   // 11
t(1)   // "eleven"
t(2)   // Person("Eleven")

val (num, str, person) = t

// result:
// val num: Int = 11
// val str: String = eleven
// val person: Person = Person(Eleven)
```

顶层定义
```shell
import scala.collection.mutable.ArrayBuffer

enum Topping:
  case Cheese, Pepperoni, Mushrooms

import Topping.*
class Pizza:
  val toppings = ArrayBuffer[Topping]()

val p = Pizza()

extension (s: String)
  def capitalizeAllWords = s.split(" ").map(_.capitalize).mkString(" ")

val hwUpper = "hello, world".capitalizeAllWords

type Money = BigDecimal

// more definitions here as desired ...

@main def myApp =
  p.toppings += Cheese
  println("show me the code".capitalizeAllWords)
```

替换包对象（我真是搞不懂真有意思）
```shell
package foo {
  def double(i: Int) = i * 2
}

package foo {
  package bar {
    @main def fooBarMain =
      println(s"${double(1)}")   // this works
  }
}
```
如果您熟悉 Scala 2，此方法将替换包对象。但是虽然更容易使用，但它们的工作方式相似：当您将定义放在名为 foo 的包中时，您可以在 foo 下的所有其他包下访问该定义，例如在本例中的 foo.bar 包中：

这种方法的好处是您可以将定义放在名为 com.acme.myapp 的包下，然后这些定义可以在 com.acme.myapp.model、com.acme.myapp.controller 等中引用。

我想，这里大概的意思是说定义一个package名为foo里面的方法可以被其子包访问。

原本以为半小时学完，结果花了2个小时还在看。

不过也基本了解完了。

总结，学会了：

* 如何使用 Scala REPL
* 如何使用 val 和 var 创建变量
* 一些常见的数据类型
* 控制结构
* 如何使用 OOP 和 FP 样式对现实世界进行建模
* 如何创建和使用方法
* 如何使用 lambdas（匿名函数）和高阶函数
* 如何将对象用于多种用途
* 上下文抽象简介
* 我们还提到，如果您更喜欢使用基于浏览器的 Playground 环境而不是 Scala REPL，您也可以使用 Scastie.scala-lang.org 或 ScalaFiddle.io。
* Scala 有更多的特性，在这个旋风之旅中没有涉及到。有关更多详细信息，请参阅本书的其余部分和参考文档

taste原以为是很美味的那种意思，结果是气味的意思。

接下来应该开始看Play来开发项目了吧。

<img src='data:image/png;base64,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'>

这里看到已经成功启动了项目，然后浏览器中打开http://localhost:9000结果报错:
```shell
Unexpected exception
UncheckedExecutionException: java.lang.IllegalStateException: Unable to load cache item

No source available, here is the exception stack trace:
->com.google.common.util.concurrent.UncheckedExecutionException: java.lang.IllegalStateException: Unable to load cache item
     com.google.common.cache.LocalCache$Segment.get(LocalCache.java:2051)
...
```
[解决方案](https://github.com/playframework/playframework/issues/10866)

需要更新java到java12，我还有个flutter开发环境需要java8，这可怎么办呢？

先休息下，准备玩一会再想怎么解决。

## Custom java version for scala

SBT runs within the JVM, so you would use whatever the standard mechanism is to determine which java you get when you run java,
for instance the `JAVA_HOME` environment variable.

也就是说需要设置JAVA_HOME就可以了。

OK，试试。在执行`sbt run`的命令前先设置这个环境变量.
```shell
$ export JAVA_HOME=/Users/congshi/development/jdk-12.0.2.jdk/Contents/Home
```
我下载了java12的可执行版本解压到了我的这个`development`目录下。

<img src='data:image/png;base64,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'>

从上面的结果看，情况良好，显示了 java版本为12。现在再来运行我们的scala play项目，看有没有错误。
```shell
$ cd playtutorial
$ sbt run
```
这次没有问题了.
<img src='data:image/png;base64,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'>

## 编写接口

接下来就是准备开发我们的项目需要编写接口，

1. 用户搜索接口 
2. 爬虫推送数据接口
3. 用户浏览反馈接口

3这个接口主要是用于用户反馈信息，用户浏览完这个网页之后，觉得有用没用需要用户点一下，如果用户点了就发过来。

2和3都是需要操作HBase数据库的，同时也要写elasticsearch，只有1是查询，所以，这个业务变成了读少，写多的问题。

那么，我们需要判断以下读多写少，是否符合我们的需求，还是说需要均衡以下，不要让爬虫每次推送都写elasticsearch，
让爬虫1天1更。但是试想以下，就算是1天1更，那1天积攒下来的数据，也足够更新1天了。为了24小时不间断服务，还是逃
步调读少写多的问题，写的多，注定要影响读。不过我们可以试试。

编写接口之前需要搞定如何在elasticsearch中创建索引。需要搜索以下，elasticsearch在scala play框架下的库，如果
有，就更好了，没有的话，找出elasticsearch的索引表的创建，然后继续一步一步来实施我们的每一步。
