# Map & Reduce in Scala Collection


## What's Map & Reudce

**Map** 是指將 collection 每個元素，經過指定的 function 處理，產生新的值 (也可以是另一個 Collection)

Map 的特性：

* Collection 的每一個元素都是獨立被處理，也就是說跟 collection 的其他元素無關。
* 經 Map 程序處理後，最後回傳的 collection 資料型別不變。比如說： List 經過 map 程序後，回傳值依然是 List，但其中的元素資料型別視處理的 function 而有所不同。
* 原本的 collection 內的元素值都不會被改變，最後是回傳一個**新**的 collection。

延伸： **flatMap**

**Reduce** 是指將 collection 的元素做**歸納**處理後，回傳一個跟元素資料型別相同或者是**supertype** 的值。

Reduce 特性：

* 指定的**歸納** function，第一次會處理兩個元素值。
* 每次處理後的結果，再跟下一個元素，再做一次處理，以此類推，直到所有的元素都被處理過。
* 處理的過程，不見得會依照預期的順序，因此指定的 function 如果沒有結合律的特性，也許結果會不如預期。
* 回傳最終處理的結果，且資料型別是跟元素相同或者是 supertype。

註：結合律是 (x + y) + z = x + (y + z)。像四則運算的 \\(+\\) 與 \\(\times\\) 有結合律，但 \\(-\\) 與 \\(\div\\) 沒有。

延伸：**reduceLeft** & **reduceRight**


舉例：輸入一個字串的 List，計算其中字串長度的總和。

```scala
val lst = List("ABC", "Hello, World!!!", "Apple", "Microsoft")
val lenLst = lst map { _.length }
val total = lenLst reduce { _ + _ }
```

或

```scala
val lst = List("ABC", "Hello, World!!!", "Apple", "Microsoft")
lst map { _.length } reduce { _ + _ }
```

Function Languge Map-Reduce 的概念，後來被應用到 Hadoop 的 Map-Reduce。

## Map, flatMap & for-yield

Scala 的 `for-yield` 處理，實際上是轉成 `flatMap` 與 `map` 處理。

舉例：

一層迴圈

```scala
for (i <- 1 to 9) yield i + 1

(1 to 9) map { _ + 1 }
```

二層迴圈

```scala
for (i <- 1 to 9; j <- 1 to i) yield i * j
(1 to 9) flatMap { i => (1 to i) map { i * _ } }
```

Option 處理

單個：

```scala
val s1 = Some("s1")
for (s <- s1) yield s + s
s1 map { s => s + s }
```

多個：

```scala
val s1 = Option("s1")
val s2 = None

for (a <- s1; b <- s2) yield (a, b)

s1 flatMap { a => s2 map { b => (a, b) } }

for (a <- s1; b <- s2) yield a + b

s1 flatMap { a => s2 map { b => a + b } }
```

`flatMap` or `for-yield` 很適合處理 `AND` 的情況

舉例：每個 Store 都有一個歸屬的 Store，目前要查詢歸屬的 Store 名稱。

```scala
case class Store(id: Int, name: String, belong: Int)

val map = Map(0 -> Store(0, "3C", 0))
```

if-else 的版本

```scala
val s1 = map.get(0)

if (s1.isDefined) {
  val s2 = map.get(s1.get.belong)
	
  if (s2.isDefined) Some(s2.get.name)
  else None
}
else None
```

for-yield 版本

```scala
for (s1 <- map.get(0); s2 <- map.get(s1.belong)) yield s2.name
```

## Collection 相關函數

### fold, foldLeft, foldRight

`fold` 與 `reduce` 很類似，`fold` 多了可以指定 **初始值**
 
```scala
val lst = List(1, 2, 3, 4, 5, 6, 7, 8, 9)

lst reduce { _ + _ }

lst.fold(100) { _ + _ }
```

### scan, scanLeft, scanRight

`scan` 可以指定 **初始值**，第一個元素與初始值處理的結果，再與第二個元素處理，以此類推，最後回傳原本 collection 的資料型別，初始值當作第一個元素。概念跟 `map` 有點像，`map` 是獨立處理每個元素，`scan` 會與上一次處理的結果有關。

```scala
val lst = List(1, 2, 3)
lst map { _ + 1 }

lst.scan(10) { (a, b) => println(a, b); a + b  }
```

### groupBy

`groupBy` 利用指定的 function 回傳值當作 **key**，自動依 key 分群，回傳一個 `Map`，Map 內的 value 資料型別，會與原來的 collection 相同。

```scala
val lst = List(1, 2, 3, 4, 5, 6, 7, 8, 9)
lst groupBy { _ % 3 }

val arr = Array(1, 2, 3, 4, 5, 6, 7, 8, 9)
arr groupBy { _ % 3 }
```

### Zip

將兩個 collection 中的元素，一對一的方式組成兩個元素的 **tuple**。行為很類似 **拉鏈**。

```scala
val lst1 = List("a", "b", "c", "d")

val lst2 = List(1, 2, 3)

lst1 zip lst2

lst1.zipWithIndex
```

## Sample: Min Heap Tree

```scala
object MyTree {

  sealed trait Tree

  object Empty extends Tree

  case class Node(value: Int, left:  Tree, right: Tree) extends Tree


  def depth(tree: Tree): Int = tree match {
    case Empty => 0
    case Node(value, left, right) => 1 + Seq(depth(left), depth(right)).max
  }


  def addNode(value: Int, tree: Tree): Tree = tree match {
    case Empty => Node(value, Empty, Empty)
    case Node(nodeValue, left, right) => {
      val leftDepth = depth(left)
      val rightDepth = depth(right)

      if (value <= nodeValue) {
        if (leftDepth <= rightDepth) Node(value, addNode(nodeValue, left), right)
        else Node(value, left, addNode(nodeValue, right))
      }
      else {
        if (leftDepth <= rightDepth) Node(nodeValue, addNode(value, left), right)
        else Node (nodeValue, left, addNode(value, right))
      }
    }
  }


  def travel(tree: Tree): Unit = {
    tree match {
      case Empty => println("*")
      case Node(value, left, right) =>
        println(value)
        print("left:")
        travel(left)
        print("right:")
        travel(right)
    }
  }

  def minHeapTree(values: Int*): Tree = {
    val a: Tree = Empty
    values.foldLeft(a) { (b, x) => addNode(x, b) }
  }

}


object Hello {
  def main(args: Array[String]): Unit = {

    import MyTree._
    //val tree = MyTree.minHeapTree(3, 9, 8, 2, 6, 4, 5, 1, 7, 10)
    val tree = MyTree.minHeapTree(3, 9, 8, 2, 6, 4, 5, 1, 7, 10)
    println(MyTree.depth(tree))
    MyTree.travel(tree)

    val a: Tree = Empty

    List(3, 9, 8, 2, 6, 4, 5, 1, 7, 10).scanLeft(a) { (t, x) => addNode(x, t) } foreach { t =>
      println("---------------------")
      travel(t)
    }

  }
}
```

# 附錄

# Scala Variance & Bounds

## Varince

Variance 主要是在討論

If \\(T_1\\) is subclass of \\(T\\), is Container[\\(T_1\\)] is subclass of Container[\\(T\\)] ?


|   Variance    | Meaning         | Scala notation |
|:------------- |:----------------|:---------------|
| covariant     | C[\\(T_1\\)] is subclass of C[\\(T\\)]| [+T] |
| contravariant | C[\\(T\\)] is subclass of C[\\(T_1\\)]| [-T] |
| invariant     | C[\\(T_1\\)] and C[\\(T\\)] are not related | [T] |


舉例：**Covariant**

```scala
class Covariant[+A]
val cv: Covariant[AnyRef] = new Covariant[String]
val cv: Covariant[String] = new Covariant[AnyRef] // ERROR
```

舉例：**Contravariant**

```scala
class Contravariant[-A]
val cv: Contravariant[String] = new Contravariant[AnyRef]
val cv: Contravariant[AnyRef] = new Contravariant[String] // ERROR
```

範例來自：[Twitter's Scala School - Type & polymorphism basics](https://twitter.github.io/scala_school/type-basics.html)

記憶方法：

\\[
\forall T_1 \in +T, \text{then }T_1 \text{ is subclass of } T
\\]

\\[
\forall T_1 \in -T, \text{then }T_1 \text{ is superclass of } T
\\]


\\[
\begin{equation}
-T \\
\uparrow\\
T \\
\uparrow \\
+T
\end{equation}
\\]

ContraVariant 案例：`Function1[-T, +R]`

eg:

```scala
class Test[+A] { def test(a: A): String = a.toString } // ERROR
```

fix: 

```scala
class Test[+A] { def test[B >: A](b: B): String = b.toString }
```

## Bounds

### Lower Type Bound

`A >: B` => `A` is superclass of `B`. `B` is the lower bound.

### Upper Type Bound

`A <: B` => A is subclass of `B`. `B` is the upper bound.

參考：

* [Twitter's Scala School - Type & polymorphism basics](https://twitter.github.io/scala_school/type-basics.html)