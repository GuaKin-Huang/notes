>  `for` 的作用结合了 `flatMap`,  `map` 和 `filter` ；如果打算做的事情非常简单，下面两种方式的写法可读性都非常好；但是如何需要把 `flatMap`, `map`, 和 `filter` 串联起来实现复杂的操作，那么 `for` 语法糖的作用就非常简单明了了。

```scala
val filterMapFlatmapVersion: List[String] = 
  fileNames.filter(_.endsWith("txt")).flatMap { file =>
    io.Source.fromFile(file).getLines().toList.map(_.trim).filter(_ == pattern)
}

val forVersion: List[String] =
  for {
    fileName <- fileNames
    if fileName.endsWith("txt")
    line <- io.Source.fromFile(fileName).getLines()
    if line.trim().matches(pattern)
  } yield line.trim
```

