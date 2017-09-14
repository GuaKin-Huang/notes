## 从源码层面理解Either、Option、Try

> 差异

- Either

代表一个结果的两个可能性，一个是 `Right` ，一个是 `Left` 

- Option

代表可选择的值，一个是 `Some`（代表有值），一个是 `None` （值为空）；常用于结果可能为 `null` 的情况；

- Try

运算的结果有两种情况，一个是运行正常，即 `Success` ，一个是运行出错，抛出异常 ，即 `Failure` ，其中 `Failure` 里面包含的是异常的信息；



> 共同点

三者都存在两种可能性的值；都可以在结果之上进行 `map` 、 `flatMap` 等操作；



---

- Either

`Right` 和 `Left` 是继承自 `Either` 的两个 `case` 类；

```scala
//Left
final case class Left[+A, +B](@deprecatedName('a, "2.12.0") value: A) extends Either[A, B]

//Right
final case class Right[+A, +B](@deprecatedName('b, "2.12.0") value: B) extends Either[A, B]
```

`Eihter` 代表一个结果的两个可能性，一个是 `Right` ，一个是 `Left` ; 

```scala
import scala.io.StdIn._
val in = readLine("Type Either a string or an Int: ")
val result: Either[String,Int] =
  try Right(in.toInt)
  catch {
    case e: NumberFormatException => Left(in)
  }
result match {
  case Right(x) => s"You passed me the Int: $x, which I will increment. $x + 1 = ${x+1}"
  case Left(x)  => s"You passed me the String: $x"
}
```

`Either` 是偏向 `Right` 值的，在 `Either` 使用 `map` 、`flatMap` 等操作时，只有 `Either` 的结果是 `Right` 时，才会触发操作；习惯性地将`Left` 值代表不好的结果（失败的结果），`Right` 代表好的结果（成功的结果）;

```scala
def doubled(i: Int) = i * 2
Right(42).map(doubled) // Right(84)
Left(42).map(doubled)  // Left(42)
```

由于`Either` 定义了 `flatMap` 和 `map` ，所以可以对 `Either` 使用 `for comprehensions`；

```scala
val right1 = Right(1)   : Right[Double, Int] //确定right1的类型
val right2 = Right(2)
val right3 = Right(3)
val left23 = Left(23.0) : Left[Double, Int]  //确定left23的类型
val left42 = Left(42.0)
for {
  x <- right1
  y <- right2
  z <- right3
} yield x + y + z // Right(6)
for {
  x <- right1
  y <- right2
  z <- left23
} yield x + y + z // Left(23.0)
for {
  x <- right1
  y <- left23
  z <- right2
} yield x + y + z // Left(23.0)
```

但是不支持使用守卫表达式

```scala
for {
  i <- right1
  if i > 0
} yield i
// error: value withFilter is not a member of Right[Double,Int]
```

 同样，下面也是不支持的

```scala
for (x: Int <- right1) yield x
// error: value withFilter is not a member of Right[Double,Int]
```

由于  `for comprehensions` 使用 `map` 和 `flatMap` ，所以必须要推导参数的类型，并且该类型必须是 `Either` ；特别的地方在于，由于`Either` 是偏向`Right` 的，所以是对于`Either`的值为`Left`必须要指定其类型，否则，该位置的默认类型为`Nothing`；

```scala
for {
  x <- left23
  y <- right1
  z <- left42  // type at this position: Either[Double, Nothing]
} yield x + y + z
//            ^
// error: ambiguous reference to overloaded definition,
// both method + in class Int of type (x: Char)Int
// and  method + in class Int of type (x: Byte)Int
// match argument types (Nothing)
for (x <- right2 ; y <- left23) yield x + y  // Left(23.0)
for (x <- right2 ; y <- left42) yield x + y  // error
for {
  x <- right1
  y <- left42  // type at this position: Either[Double, Nothing]
  z <- left23
} yield x + y + z
// Left(42.0), but unexpectedly a `Either[Double,String]`
```

- Option

`Some` 和 `None` 是继承自 `Option` 的两个 `case` 类；

```scala
//Some
final case class Some[+A](@deprecatedName('x, "2.12.0") value: A) extends Option[A]

//None
case object None extends Option[Nothing]
```

对`Option` 的习惯用法是把它当作集合或者`monad` ，通过`map` 、`flatMap` 、`filter` 和 `foreach` ：

```scala
//方式一
val name: Option[String] = request getParameter "name"
val upper = name map { _.trim } filter { _.length != 0 } map { _.toUpperCase }
println(upper getOrElse "")

//方式一等价于方式二
val upper = for {
  name <- request getParameter "name" //由于For表达式的作用，如何此处返回None，那么整个表达式将返回None
  trimmed <- Some(name.trim)
  upper <- Some(trimmed.toUpperCase) if trimmed.length != 0
} yield upper
println(upper getOrElse "")
```


另外一个习惯用法是（不太推荐）通过模式匹配：

```scala
val nameMaybe = request getParameter "name"
nameMaybe match {
  case Some(name) =>
    println(name.trim.toUppercase)
  case None =>
    println("No name value")
}
```

- Try

`Failure` 和 `Success` 是继承自 `Try` 的两个 `case` 类；

```scala
//Failure
final case class Failure[+T](exception: Throwable) extends Try[T]

//Success
final case class Success[+T](value: T) extends Try[T]
```

`Try` 常用于那些存在异常的地方，通过`Try` 不用确定地对可能出现的异常进行处理；

```scala
import scala.io.StdIn
import scala.util.{Try, Success, Failure}
def divide: Try[Int] = {
  val dividend = Try(StdIn.readLine("Enter an Int that you'd like to divide:\n").toInt)
  val divisor = Try(StdIn.readLine("Enter an Int that you'd like to divide by:\n").toInt)
  val problem = dividend.flatMap(x => divisor.map(y => x/y))
  problem match {
    case Success(v) =>
      println("Result of " + dividend.get + "/"+ divisor.get +" is: " + v)
      Success(v)
    case Failure(e) =>
      println("You must've divided by zero or entered something that's not an Int. Try again!")
      println("Info from the exception: " + e.getMessage)
      divide
  }
}
```

在上面的例子中，可以看出 `Try` 的一个重要的特性，就是`Try` 具有管道的功能 ，`flatMap` 和 `map` 将那些成功完成的操作的结果包装成`Success` ，将那些异常包装成 `Failure` ，而对于 `recover` 和 `recoverWith` 则是默认对 `Failure` 结果进行触发；