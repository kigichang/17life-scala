# Scala Collection

Scala Collection 功能比 Java 強大很多。在語法上也差異很大。因此在學習過程，建議直接依書上的範例做過一遍。

建議閱讀資料：

- [Twitter Effective Scala - Collections](http://twitter.github.io/effectivescala/#Collections)
- [Scala Collection API](http://www.scala-lang.org/docu/files/collections-api/collections.html)
- [Scala Collection API - Performance Characteristics](http://www.scala-lang.org/docu/files/collections-api/collections_40.html)
- [Referential Transparency](http://en.wikipedia.org/wiki/Referential_transparency_%28computer_science%29)

## Scala Right Operator

- Right Operator `:`，Scala 定義 Operator，如果結尾是 `:`，就當作 Right Operator 處理。
- 一般熟悉的是 Left Operator。eg: `a + b`，實際是 `a.+(b)`；Right Operator 是 `a +: b`，則是 `b.+:(a)`。

### Scala 慣例：

- `+` 用在加單一元素
- `++` 用在加一個 collection
- 只有 `mutable` 集合有類似 `+=`, `++=`, `-=`, `--=`。

## Array

Scala 有 `Array` 的 Class，語法上與 Java 差異很大。
eg:

Java String Array: `String[] test = Strig[5];`

Scala String Array: `val test = Array.fill(5) { "" }`

Q: Array 特性？！

Scala `Array` 是一組 mutable indexed 集合。

宣告:

```scala {.line-numbers}
val arr1 = Array(1, 2, 3)
val arr2 = Array("a", "b", "c")
val arr3 = Array.fill(3) { math.random }

class Test
val arrTest = Array.fill[Test](3) { null }
```

取值：

```scala {.line-numbers}
arr1(1)
```

## Immutable v.s. Mutable

Scala Collection Class 分成 **Immutable*- 及 **Mutable**。兩者主要差別是 **Mutale*- 可以修改集合裏面的內容；但 **Immutable*- 不行。使用類似 `append` 時，`Immutable` 的集合，會回傳一個新的集合，而非就現有的集合擴充，在使用時，要特別小心，以免造成記憶體浪費。

Mutable Collection Class 都會有 `toXXXX` 去轉成 Immutable Collection。

p.s. 在 Java 最常用的 immutable 是 **String**

### Scala Collection Hierarchy

![Scala Collection](http://www.scala-lang.org/docu/files/collections-api/collections.png)

### Scala Immutable Collection

![Scala Immutable Collection](http://www.scala-lang.org/docu/files/collections-api/collections.immutable.png)

### Scala Mutable Collection

![Scala Mutable Collection](http://www.scala-lang.org/docu/files/collections-api/collections.mutable.png)

### 效能問題

Scala 的 Collection 有很完整的繼承結構，在效能上，越底層的 Class 效能會越差。因此使用時，要確定好你需要的功能是什麼。如果只用到 `Seq` ，就不需要用到 `List`。

### Immutable or Mutable

用那一個？

- 基本上，如果宣告後，就不會再更動內容，就直接用 **Immutable**
- 需要用程式處理初始化 (如：從 DB 讀資料，組出 List)；這類 case，一開始會用 mutable，但做完初始化後，即刻轉成 immutable。
- API 的參數設計，儘量使用 immutable 型態。

Q: 為什麼建議使用 **Immutable**?

A: 因為 Muli-Thread。學 Scala 最主要的目的，就是加速開發平行運算的程式，也就是 multi-thread 的程式。Immutable Read Only 的特性，使得有 **Thread-Safe*- 效果，因此在程式撰寫時，建議儘量使用 Immutable class。

### Twitter Coding Style

由於 collection class 在 mutable 及 immutable 都會有。所以在使用時，為了避免誤用，Twitter 建議明確指出是目前是使用那一種。如完整指出使用 `mutable`。**不要使用*- `import scala.collection.mutable._`

eg:

```scala {.line-numbers}
import scala.collection.mutable

val set = mutable.Set()
```

## 常用的 Collection Sample

### List

宣告：

```scala {.line-numbers}
val colors = List("red", "blue", "green")
```

取頭：

```scala {.line-numbers}
colors.head
```

去頭：

```scala {.line-numbers}
scala> colors.tail
```

取其中一個值：

```scala {.line-numbers}
colors(1)
```

取最後一個：

```scala {.line-numbers}
scala> colors.last
```

去掉最後一個：

```scala {.line-numbers}
colors.init
```

長度：

```scala {.line-numbers}
colors.length
```

串成字串：

```scala {.line-numbers}
colors.mkString(",")

colors.mkString("[", "],[", "]")
```

加資料在前面 (prepend)：

```scala {.line-numbers}
val colors2 = "yellow" +: colors

"yellow" :: colors
```

加資料在後面 (append)：

```scala {.line-numbers}
val color3 = colors2 :+ "white"
List[String] = List(yellow, red, blue, green, white)
```

接另一個 List 在前面 (prepend)：

```scala {.line-numbers}
val colors4 = colors2 ++: colors
colors4: List[String] = List(yellow, red, blue, green, red, blue, green)

colors2 ::: colors
```

接另一個 List 在後面 (append)：

```scala {.line-numbers}
val colors5 = colors2 ++ colors
colors5: List[String] = List(yellow, red, blue, green, red, blue, green)
```

拿掉前兩個：

```scala {.line-numbers}
colors4.drop(2)
```

拿掉最後兩個：

```scala {.line-numbers}
colors4.dropRight(2)
```

取前兩個：

```scala {.line-numbers}
scala> colors4.take(2)
```

取最後兩個：

```scala {.line-numbers}
colors4.takeRight(2)
```

foreach:

```scala {.line-numbers}
colors4 foreach { println }
```

判斷是否沒有資料：

```scala {.line-numbers}
colors4.isEmpty
List().isEmpty
```

只留符合條件的資料：

```scala {.line-numbers}
colors4 filter { _.length > 4 }
```

計算符合條件的資料數量：

```scala {.line-numbers}
colors4 count { _.length > 4 }
```

去除前面符合條件的資料，當遇到不符合條件就停止 (while - break)：

```scala {.line-numbers}
colors4
colors4 dropWhile { _.length > 4 }
```

取前面符合條件資料，當遇到不符合條件就停止 (while - break)：

```scala {.line-numbers}
colors4
colors4 takeWhile { _.length > 4 }
```

收集並加工符合條件的資料 (filter 的加強版) ：

```scala {.line-numbers}
colors4 collect { case x if x.length > 4 => x + x }
```

找第一個符合條件的資料

```scala {.line-numbers}
val colors4 = List("yellow", "red", "blue", "green", "red")
colors4.find { _ == "red" }

colors4.find { _ == "black" }
```

找到第一個符合條件的資料，並加工

```scala {.line-numbers}
colors4.collectFirst { case x if (x.length == 3) => x + x }
```

找到第一個符合條件的位置。

```scala {.line-numbers}
colors4 = List("yellow", "red", "blue", "green", "red")

colors4.indexWhere( _ == "red")

colors4.indexWhere( _ == "red", 2)

colors4.indexWhere( _ == "black", 2)

color4.lastIndexWhere( _ == "red")

color4.lastIndexWhere( _ == "red", 3)
```

由於 Scala 拿掉 `break`，因此在原本 Java 使用 `break;` 的情境，就可以使用 `dropWhile`, `takeWhile`, `collect`, `collectFirst`, `indexWhere`, `lastIndexWhere`。

入門 Scala Collection 建議可以由 List 下手。 List 有的 function，大都的 Collection 也都有。

### ListBuffer

宣告：

```scala {.line-numbers}
val list = mutable.ListBuffer.empty[String]

val list = mutable.ListBuffer("a", "b", "c")
```

Append:

```scala {.line-numbers}
list += "d"
```

Prepend:

```scala {.line-numbers}
"e" +=: list
```

### Range

```scala {.line-numbers}
1 to 10

1 until 10
```

### Set

Immutable 及 mutable 都有 Set，以下以 `mutable.Set` 當例子。

宣告

```scala {.line-numbers}
import scala.collection.mutable

val set = mutable.Set('a', 'b', 'c')
```

資料數：

```scala {.line-numbers}
set.size
```

加值進 Set：

```scala {.line-numbers}
set += 'd'
```

與另一個 Set 合併：

```scala {.line-numbers}
set ++= mutable.Set('a', 'b', 'e', 'f')
```

移除一個值：

```scala {.line-numbers}
set -= 'a'
```

判斷是否有值：

```scala {.line-numbers}
set('a')

set('b')
```

### Map

Immutable 及 mutable 都有 Map，以下以 `mutable.HashMap` 當例子。

宣告：

```scala {.line-numbers}
import scala.collection.mutable

val map = mutable.HashMap.empty[String, Int]

val map = mutable.HashMap("i" -> 1, "ii" -> 2)
```

資料數：

```scala {.line-numbers}
map.size
```

Keys:

```scala {.line-numbers}
map.keys
```

Key Set:

```scala {.line-numbers}
map.keySet
```

Values:

```scala {.line-numbers}
map.values
```

Put：

```scala {.line-numbers}
map += ("iii" -> 3)
```

Remove：

```scala {.line-numbers}
scala> map -= ("i")
```

直接取值：

```scala {.line-numbers}
map("iii")	// Error
```

直接取值的方式，key 不存在時，會出現 Exception。

使用的時機：你確定一定有值，或者沒值，是很嚴重的錯誤，需要用 Exception 來通知系統。

間接方式：

```scala {.line-numbers}
map.get("iiii")

map.get("iii")
```

用 `get` 的方式，取回 `Option` 的型別，再做下一步處理，一定沒問題。

## Compare with Java and Conversions

Scala 與 Java Collection 互轉的原則：

Scala                 | Java
--------------------: | :---------------------------------
Iterator              | java.util.Iterator
Iterator              | java.util.Enumeration
Iterable              | java.lang.Iterable
Iterable              | java.util.Collection
mutable.Buffer        | java.util.List
mutable.Set           | java.util.Set
mutable.Map           | java.util.Map
mutable.ConcurrentMap | java.util.concurrent.ConcurrentMap

使用 Scala 大都會用到 Java 既有的 Library，所以會遇到取得的資料型別是 Java 的 Collection。如果要使用 Scala 提供的功能時，會需要將 Java 的 Collection 轉成 Scala 的 Collection。

遇到這種情形時，只要 import JavaConversions 即可，程式碼：`import scala.collection.JavaConversions._`。

**注意**：

- 這類型的轉換，在 Scala 是利用 `implicit` 機制完成，原理是產生新的 Scala Collection Class 去 wrap 原來的 Java Class，因此在使用時要小心，以免吃光記憶體。
- 在 Eclipse 上，會提醒 `implicit` 的轉換。
- 依 Java 習慣，不建議直接用 `import scala.collection.JavaConversions._`，可以利用 Eclipse 的 **Source** -> **Organize Imports** 來整理 import，會自動整理成明確 import 那些東西。
