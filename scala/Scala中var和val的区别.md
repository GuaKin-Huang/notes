## Scala 中 var 和 val 的区别

一想到这两个的区别，大多数人第一反应就是，`var` 修饰的变量可改变，`val` 修饰的变量不可改变；但真的如此吗？事实上，`var` 修饰的对象引用可以改变，`val` 修饰的则不可改变，但对象的状态却是可以改变的。例如：

```scala
class A(n: Int) {
  var value = n
}

class B(n: Int) {
  val value = new A(n)
}

object Test {
  def main(args: Array[String]) {
    val x = new B(5)
    x = new B(6) // 错误，因为 x 为 val 修饰的，引用不可改变
    x.value = new A(6) // 错误，因为 x.value 为 val 修饰的，引用不可改变
    x.value.value = 6 // 正确，x.value.value 为var 修饰的，可以重新赋值
  }
}
```

对于变量的`不变性`有很许多的好处。

一是，如果一个对象不想改变其内部的状态，那么由于`不变性`，我们不用担心程序的其他部分会改变对象的状态；例如

```scala
x = new B(0)
f(x)
if (x.value.value == 0)
  println("f didn't do anything to x")
else
  println("f did something to x")
```

这对于多线程（多进程）的系统来说尤其重要。在一个多线程系统中，以下可能发生：

```scala
x = new B(1)
f(x)
if (x.value.value == 1) {
  print(x.value.value) // Can be different than 1!
}
```

如果你只使用 `val` ，并且只使用不可变的数据结构（即是，避免使用 `arrays` ，`scala.collection.mutable` 内的任何东西，等等），你可以放心，这是不会发生的；除非有一些代码，例如一个框架，进行反射——反射可以改变`不可变`的值。

二是，当用 `var` 修饰的时候，你可能在多个地方重用 `var` 修饰的变量，这样会产生下面的问题：

- 对于阅读代码的人来说，在代码的确定部分中知道变量的值是比较困难的；
- 你可能会在使用代码前初始化代码，这样会导致错误；

因此，简单地说，使用 `val` 是安全和增强代码可读性的。



既然 `val` 有这么多的好处，那为什么还要使用（或者说存在） `var` ； 的确，有些编程语言只存在 `val`的情况 ，但是有些情况下，使用可变性可以大幅度提高程序的执行效率。

例如，对于一个不可变的 `Queue` ，当每次对队列进行 `enqueue` 和 `dequeue` 操作时，将会得到一个新的 `Queue` 对象，那么 ，如何处理所有的项目呢？

下面我们通过例子进行解释。假设一个 `Int` 类型的队列，对队列内的所有数字进行组合；例如队列的元素有 1，2，3，那么组合后的数字就是 123。下面是第一种解决方案：采用的是 `mutable.Queue`

```scala
def toNum(q: scala.collection.mutable.Queue[Int]) = {
  var num = 0
  while (!q.isEmpty) {
    num *= 10
    num += q.dequeue
  }
  num
}
```

上面的代码易于阅读和理解，但存在一个主要的问题是，会改变源数据，因此在调用 `toNum` 方法前必须对源数据进行拷贝，避免对源数据产生污染。这时一种对对象进行不变性管理的方法。

接下来采用 `immutable.Queue` :

```scala
def toNum(q: scala.collection.immutable.Queue[Int]) = {
  def recurse(qr: scala.collection.immutable.Queue[Int], num: Int): Int = {
    if (qr.isEmpty)
      num
    else {
      val (digit, newQ) = qr.dequeue
      recurse(newQ, num * 10 + digit)
    }
  }
  recurse(q, 0)
}
```

因为 `num` 不能被重新分配值，就像在前面的例子中一样，因此需要使用递归。这是一个尾部递归，它的性能很好。但情况并非总是如此：有时根本就没有好的(可读的、简单的)尾递归解决方案。

下面采用 `immutable.Queue` 和 `mutable.Queue` 对代码进行重写：

```scala
def toNum(q: scala.collection.immutable.Queue[Int]) = {
  var qr = q
  var num = 0
  while (!qr.isEmpty) {
    val (digit, newQ) = qr.dequeue
    num *= 10
    num += digit
    qr = newQ
  }
  num
}
```

这段代码非常有效，不需要递归，也无需担心是否需要在调用`toNum`之前复制队列。自然地，我避免了由于其他用途而对变量进行重用，由于在这个函数之外没有任何代码可以看到（修改）它们，所以我不需要担心它们的值在代码的其他地方会发生——除非我明确地这么做。

如果程序员认为某种解决方案是最好的解决方案，那么 `Scala` 就允许程序员这么做。其他变成语言则没有这么大的灵活性。`Scala` (以及任何具有广泛可变性的语言)的代价是，编译器在优化代码方面没有足够的灵活性。Java的提供解决方案是基于运行时来优化代码。