# Map & Reduce in Scala Collection


## What's Map & Reudce

**Map** 是指將 collection 每個元素，經過指定的 function 處理，產生新的值 (也可以是另一個 Collection)

Map 的特性：

* Collection 的每一個元素都是獨立被處理，也就是說跟 collection 的其他元素無關。
* 經 Map 程序處理後，最後回傳的 collection 資料型別不變。比如說： List 經過 map 程序後，回傳值依然是 List，但其中的元素資料型別視處理的 function 而有所不同。
* 原本的 collection 內的元素值都不會被改變，最後是回傳一個**新**的 collection。

延伸： **flatMap**

**Reduce** 是指將 collection 的元素做**歸納**處理後，回傳一個跟元素資料型別相同或者是**subtype** 的值。

Reduce 特性：

* 指定的**歸納** function，第一次會處理兩個元素值。
* 每次處理後的結果，再跟下一個元素，再做一次處理，以此類推，直到所有的元素都被處理過。
* 處理的過程，不見得會依照預期的順序，因此指定的 function 如果沒有結合律的特性，也許結果會不如預期。
* 回傳最終處理的結果，且資料型別是跟元素相同或者是 subtype。

註：結合律是 (x + y) + z = x + (y + z)。像四則運算的 \\(+\\) 與 \\(\times\\) 有結合律，但 \\(-\\) 與 \\(\div\\) 沒有。

延伸：**reduceLeft** & **reduceRight**


舉例：輸入一個字串的 List，計算其中字串長度的總和。

```
scala> val lst = List("ABC", "Hello, World!!!", "Apple", "Microsoft")
lst: List[String] = List(ABC, Hello, World!!!, Apple, Microsoft)

scala> val lenLst = lst map { _.length }
lenLst: List[Int] = List(3, 15, 5, 9)

scala> val total = lenLst reduce { _ + _ }
total: Int = 32
```

或

```
scala> val lst = List("ABC", "Hello, World!!!", "Apple", "Microsoft")
lst: List[String] = List(ABC, Hello, World!!!, Apple, Microsoft)

scala> lst map { _.length } reduce { _ + _ }
res7: Int = 32
```

Function Languge Map-Reduce 的概念，後來被應用到 Hadoop 的 Map-Reduce。

## Map, flatMap & for-yield

Scala 的 `for-yield` 處理，實際上是轉成 `flatMap` 與 `map` 處理。

舉例：

一層迴圈

```
scala> for (i <- 1 to 9) yield i + 1
res5: scala.collection.immutable.IndexedSeq[Int] = Vector(2, 3, 4, 5, 6, 7, 8, 9, 10)

scala> (1 to 9) map { _ + 1 }
res6: scala.collection.immutable.IndexedSeq[Int] = Vector(2, 3, 4, 5, 6, 7, 8, 9, 10)
```

二層迴圈

```
scala> for (i <- 1 to 9; j <- 1 to i) yield i * j
res7: scala.collection.immutable.IndexedSeq[Int] = Vector(1, 2, 4, 3, 6, 9, 4, 8, 12, 16, 5, 10, 15, 20, 25, 6, 12, 18, 24, 30, 36, 7, 14, 21, 28, 35, 42, 49, 8, 16, 24, 32, 40, 48, 56, 64, 9, 18, 27, 36, 45, 54, 63, 72, 81)

scala> (1 to 9) flatMap { i => (1 to i) map { i * _ } }
res8: scala.collection.immutable.IndexedSeq[Int] = Vector(1, 2, 4, 3, 6, 9, 4, 8, 12, 16, 5, 10, 15, 20, 25, 6, 12, 18, 24, 30, 36, 7, 14, 21, 28, 35, 42, 49, 8, 16, 24, 32, 40, 48, 56, 64, 9, 18, 27, 36, 45, 54, 63, 72, 81)
```

Option 處理

單個：

```
scala> val s1 = Some("s1")
s1: Some[String] = Some(s1)

scala> for (s <- s1) yield s + s
res11: Option[String] = Some(s1s1)

scala> s1 map { s => s + s }
res13: Option[String] = Some(s1s1)
```

多個：

```
scala> val s1 = Some("s1")
s1: Some[String] = Some(s1)

scala> val s2 = None
s2: None.type = None

scala> for (a <- s1; b <- s2) yield (a, b)
res15: Option[(String, Nothing)] = None

scala> s1 flatMap { a => s2 map { b => (a, b) } }
res17: Option[(String, Nothing)] = None

scala> for (a <- s1; b <- s2) yield a + b
res16: Option[String] = None

scala> s1 flatMap { a => s2 map { b => a + b } }
res18: Option[String] = None
```

`flatMap` or `for-yield` 很適合處理 `AND` 的情況

舉例：每個 Store 都有一個歸屬的 Store，目前要查詢歸屬的 Store 名稱。

```
scala> case class Store(id: Int, name: String, belong: Int)
defined class Store

scala> val map = Map(0 -> Store(0, "3C", 0))
map: scala.collection.immutable.Map[Int,Store] = Map(0 -> Store(0,3C,0))
```

if-else 的版本

```
val s1 = map.get(0)

scala> if (s1.isDefined) {
     | val s2 = map.get(s1.get.belong)
     | if (s2.isDefined) Some(s2.get.name)
     | else None
     | } else None
res6: Option[String] = Some(3C)
```

for-yield 版本

```
scala> for (s1 <- map.get(0); s2 <- map.get(s1.belong)) yield s2.name
res0: Option[String] = Some(3C)

```

## Collection 相關函數

### fold, foldLeft, foldRight

`fold` 與 `reduce` 很類似，`fold` 多了可以指定 **初始值**
 
```
scala> val lst = List(1, 2, 3, 4, 5, 6, 7, 8, 9)
lst: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8, 9)

scala> lst reduce { _ + _ }
res20: Int = 45

scala> lst.fold(100) { _ + _ }
res25: Int = 145
```

### scan, scanLeft, scanRight

`scan` 可以指定 **初始值**，第一個元素與初始值處理的結果，再與第二個元素處理，以此類推，最後回傳原本 collection 的資料型別，初始值當作第一個元素。概念跟 `map` 有點像，`map` 是獨立處理每個元素，`scan` 會與上一次處理的結果有關。

```
scala> val lst = List(1, 2, 3)
lst: List[Int] = List(1, 2, 3)

scala> lst map { _ + 1 }
res26: List[Int] = List(2, 3, 4)

scala> lst.scan(10) { (a, b) => println(a, b); a + b  }
(10,1)
(11,2)
(13,3)
res30: List[Int] = List(10, 11, 13, 16)
```

### groupBy

`groupBy` 利用指定的 function 回傳值當作 **key**，自動依 key 分群，回傳一個 `Map`，Map 內的 value 資料型別，會與原來的 collection 相同。

```
scala> val lst = List(1, 2, 3, 4, 5, 6, 7, 8, 9)
lst: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8, 9)

scala> lst groupBy { _ % 3 }
res31: scala.collection.immutable.Map[Int,List[Int]] = Map(2 -> List(2, 5, 8), 1 -> List(1, 4, 7), 0 -> List(3, 6, 9))

scala> val arr = Array(1, 2, 3, 4, 5, 6, 7, 8, 9)
arr: Array[Int] = Array(1, 2, 3, 4, 5, 6, 7, 8, 9)

scala> arr groupBy { _ % 3 }
res32: scala.collection.immutable.Map[Int,Array[Int]] = Map(2 -> Array(2, 5, 8), 1 -> Array(1, 4, 7), 0 -> Array(3, 6, 9))

```

### Zip

將兩個 collection 中的元素，一對一的方式組成兩個元素的 **tuple**。行為很類似 **拉鏈**。

```
scala> val lst1 = List("a", "b", "c", "d")
lst1: List[String] = List(a, b, c, d)

scala> val lst2 = List(1, 2, 3)
lst2: List[Int] = List(1, 2, 3)

scala> lst1 zip lst2
res0: List[(String, Int)] = List((a,1), (b,2), (c,3))

scala> lst1.zipWithIndex
res2: List[(String, Int)] = List((a,0), (b,1), (c,2), (d,3))

```
