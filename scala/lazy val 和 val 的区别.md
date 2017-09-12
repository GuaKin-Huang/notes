## val 和 lazy val的区别

`val` 和 `lazy val` 的区别是：`val` 在它定义的时候就执行（发生作用），而 `lazy val` 当它第一次被访问时才被执行（发生作用）：

```scala
scala> val x = { println("x"); 15 }
x   //定义的时候发生作用了
x: Int = 15

scala> lazy val y = { println("y"); 13 }
y: Int = <lazy>
------------------------------------------------------------------
scala> x
res2: Int = 15

scala> y
y   //第一次使用时才发生作用
res3: Int = 13

scala> y
res4: Int = 13
```

和在方法（`def` 修饰）内形成对比的是，一个 `lazy val`只会被执行一次；这个特性就非常适用于耗时的操作和不确定使用时间的地方，例如：

```scala
scala> class X { val x = { Thread.sleep(2000); 15 } }
defined class X

scala> class Y { lazy val y = { Thread.sleep(2000); 13 } }
defined class Y

scala> new X
res5: X = X@262505b7 // 等待 

scala> new Y
res6: Y = Y@1555bd22 // this appears immediately
```

在这里，当 `x` 和 `y` 的值从未被使用时，但 `x` 不必要地浪费资源。如果我们假设y没有副作用，并且我们不知道它被访问了多少次(从未，一次，数千次)，声明它为 `def` 是没有用的，因为我们不想多次执行它。