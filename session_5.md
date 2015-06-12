# Scala Functional Language 簡介

Functional Language 其實很早就有了，只是 OOP 概念比較能讓人接受，因此 Functional Language 就被忽略。近來大數據需要做大量平行與分散式的運算，Functional Lanauge 才又被重視。

Functional Language 簡單來說，就是：

Treats computation as the evaluation of **mathematical functions** and avoids **changing-state and mutable data (Side Effects)**

截自 [Wiki: Functional Lanauge](http://en.wikipedia.org/wiki/Functional_programming)

## 名詞解釋

### Mathematical Functions

Mathematical Functions 簡單來說，就是兩個集合間的關係，且每一個輸入值，只會對應到一個輸出值。

最簡單的數學函數的表示式：

\\[f: X \mapsto Y\\]

比如說：Square

\\[Square: Int \mapsto Int \\]
\\[f(x) = x^2\\]

比如 \\(f(3) = 9\\), \\(f(-3) = 9\\) ，每次計算 \\(f(3)\\) 一定都會是 **9** 不會變成其他值。

### Side Effects

程式的函式有以下的行為時，就會稱該函式有 **Side Effects**。

* Reassigning a variable (val v.s. var)
* Modify a data structure in place (mutable v.s. immutable)
* Setting a field on an object (change object state)
	* 這裏指的 object 是 OOP 的 Object 不是 Scala 的 object (singleton)
	* OOP 修改物件的值，在程式語言的術語：改變物件的狀態，如上說的 **changing-state**
* Throwing an exception or halting with error
* Printing to the console or reading user input (I/O)
* Reading from or write to a file (I/O)
* Drawing on the screen (I/O)

截自 [Functional Language in Scala](http://www.amazon.com/Functional-Programming-Scala-Paul-Chiusano/dp/1617290653)

就實際狀況來說，我們寫程式不可能不去碰 I/O。

### Purely Functions

**Purely functional functions (or expressions) have no side effects (memory or I/O).**

截自 [Wiki: Purely Function](http://en.wikipedia.org/wiki/Functional_programming#Pure_functions)

### Referential Transparency (RT)

**An expression is said to be referentially transparent if it can be replaced with its value without changing the behavior of a program**

截自 [Wiki: Referential Transparency](http://en.wikipedia.org/wiki/Referential_transparency_%28computer_science%29)

簡單來說，程式碼中的變數，可以用此變數的值或運算式子來取代，而且不會改變輸出的結果。

舉例來說：String 是 immutable，當每次呼叫 `reverse` 時，都會回傳固定的值。

```
scala> val x = "Hello, World"
x: String = Hello, World

scala> val r1 = x.reverse
r1: String = dlroW ,olleH

scala> val r2 = x.reverse
r2: String = dlroW ,olleH
```

此時，可以將上述的 `x` 直接替換成 `Hello, World`，程式的結果都不會被改變。此特性，就是 **Referential Transparency**。如下：

```
scala> val r1 = "Hello, World".reverse
r1: String = dlroW ,olleH

scala> val r2 = "Hello, World".reverse
r2: String = dlroW ,olleH
```

另舉反例：StringBuilder 是 mutable，`append` 會修改 StringBuffer 內的值 (change object state)。

```
scala> val x = new StringBuilder("Hello")
x: StringBuilder = Hello

scala> val y = x.append(", World")
y: StringBuilder = Hello, World

scala> val r1 = y.toString
r1: String = Hello, World

scala> val r2 = y.toString
r2: String = Hello, World
```

當將 `y` 替換成 `x.append(", World")` 時：

```
scala> val r1 = x.append(", World").toString
r1: String = Hello, World, World

scala> val r2 = x.append(", World").toString
r2: String = Hello, World, World, World
```

此時 `r1` 及 `r2` 的值並不一致，這樣子就沒有 **Referential Transparency** 。

範例截自： [Functional Language in Scala](http://www.amazon.com/Functional-Programming-Scala-Paul-Chiusano/dp/1617290653)

#### 為什麼 Referential Transparency 如此重要

當程式設計都符合 Referential Transparency 時，就代表程式可以分散在不同 Thread, CPU核心，甚至不同主機上處理(空間)，而且不論什麼時候被處理(時間)，都不會影響輸出的結果。

**Funcational Language 程式設計的終極目標就是 Referential Transparency。**

### First-Class Function and High Order Function

#### First-Class Function

一個程式語言有 **First-Class Function** 特性，是指此程式語言將 **Function** 當作是一種資料型態。

在 Scala 中，有定義 `Function` 這個 class。如下：

```
scala> val max = (x: Int, y:Int) => if (x > y) x else y
max: (Int, Int) => Int = <function2>

scala> max(3, 4)
res5: Int = 4
```

#### High Order Function

Hight Order Function 是指 Function 其中一個參數的資料型別是 Function。比如 `List` 的 `foreach`。

```
scala> List(1, 2, 3, 4) foreach { x => println(x + x) }
2
4
6
8
```

### Function Composition

數學的複合函數：

\\[f: X \mapsto Y\\]

\\[g: Y \mapsto Z\\]

\\[	g \circ f: X \mapsto Z\\]

\\[	(g \circ f )(x) = g(f(x))\\]

在 Scala 上的實作，有 `compose` 及 `andThen`

`f andThen g` 等同於 \\(  g \circ f \\)

`f compose g` 等同於 \\( f \circ g \\)

eg:

```
scala> val f = (x: Int) => x * x
f: Int => Int = <function1>

scala> val g = (x: Int) => x + 1
g: Int => Int = <function1>

scala> val goff = f andThen g
goff: Int => Int = <function1>

scala> goff(10)
res10: Int = 101

scala> val fofg = f compose g
fofg: Int => Int = <function1>

scala> fofg(10)
res11: Int = 121
```

#### 轉成 Function Class

一般會用 `def` 宣告 function；可以使用 **`_`**  轉換成 Function Class。如下：

```
scala> def f(x: Int) = x * x
f: (x: Int)Int

scala> def g(x: Int) = x + 1
g: (x: Int)Int

scala> f andThen g
<console>:10: error: missing arguments for method f;
follow this method with `_' if you want to treat it as a partially applied function
              f andThen g
              ^
<console>:10: error: missing arguments for method g;
follow this method with `_' if you want to treat it as a partially applied function
              f andThen g
                        ^
                        
 scala> f _ andThen g _
res1: Int => Int = <function1>
```

### Currying

假設有個 function 由兩個以上的集合對應到一個集合：

\\[f: X \times Y \mapsto Z\\]

比如說：

\\[f(x, y) = \frac{y}{x}\\]

我們可以定義一個 **Curring Function** ：

\\[h(x) = y \mapsto f(x, y)\\]

\\(h(x)\\) 是一個 Function ，它的輸入值是 x ，回傳值是 Function 。

比如說：

\\[h(2) = y \mapsto f(2, y)\\]

這時候的 \\( h(2) \\) 是一個 Function：

\\[ f(2, y) = \frac {y}{2} \\]

此時，我們可以再定義一個 Function: \\(g(y) \\)

\\[g(y) = h(2) = y \mapsto f(2, y)\\]

也就是

\\[g(y) = f(2, y) = \frac {y}{2}\\]

在 Scala 的實作：

定義  \\( f: X \times Y \mapsto Z \\)

```
scala> def f(x: Int)(y: Int) = y / x
f: (x: Int)(y: Int)Int

scala> f(4)(2)
res7: Int = 0

scala> f(2)(4)
res8: Int = 2
```

定義 \\( g(y) = h(2) = y \mapsto f(2, y) \\) i.e. 
\\[g(y) = f(2, y) = \frac {y}{2} \\]

```
scala> val h = f(2) _
h: Int => Int = <function1>
```

當  \\( y = 4 \\) 時，

\\[g(4) = f(2, 4) = \frac {4} {2} = 2\\]

```
scala> h(4)
res9: Int = 2
```
範例截自 [Wiki: Curring](http://en.wikipedia.org/wiki/Currying)

 另一種使用時機：
 
```
scala> def modN(n: Int)(x: Int) = ((x % n) == 0)
modN: (n: Int)(x: Int)Boolean

scala> val nums = List(1, 2, 3, 4, 5, 6, 7, 8)
nums: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8)

scala> nums filter { modN(2) }
res10: List[Int] = List(2, 4, 6, 8)

scala> nums filter { modN(3) }
res11: List[Int] = List(3, 6)
```

數學式對應：

\\[modN: Int \times Int \mapsto Boolean\\]

\\[modN(n, x) = ((x \bmod n ) == 0)\\]

\\[mod2(x) = modN(2, x) = ((x \bmod 2) == 0)\\]

\\[mod3(x) = modN(3, x) = ((x \bmod 3) == 0)\\]

範例截自：[Scala Document: Currying](http://docs.scala-lang.org/tutorials/tour/currying.html)

### Scala Partial Function

一般定義 Function 都會去處理輸入值的所有情況。比如說：

```
def plusOne(x: Int) = x + 1
```

所有 Int 整數值，傳進 `plusOne` 都會被處理。

**Partial Function** 換言之就是只處理某些狀況下的值。

定義：

```
scala> val one: PartialFunction[Int, String] = { case 1 => "one" }
one: PartialFunction[Int,String] = <function1>
```

使用：如果輸入沒有要處理的值時，會出現 **Exception**。比如 **1** 有定義，但 **2** 沒有，所以輸入 **1** 沒問題，輸入 **2** 就會有 **Exception**。

```
scala> one(1)
res0: String = one

scala> one(2)
scala.MatchError: 2 (of class java.lang.Integer)
  at scala.PartialFunction$$anon$1.apply(PartialFunction.scala:253)
  at scala.PartialFunction$$anon$1.apply(PartialFunction.scala:251)
  at $anonfun$1.applyOrElse(<console>:7)
  at $anonfun$1.applyOrElse(<console>:7)
  at scala.runtime.AbstractPartialFunction.apply(AbstractPartialFunction.scala:36)
  ... 33 elided
```

查詢輸入值，是否已在處理的範圍內：

```
scala> one.isDefinedAt(1)
res2: Boolean = true

scala> one.isDefinedAt(2)
res3: Boolean = false
```

#### Composition of Partial Function

可以使用多個 Partial Function 組成一個複合函數。

```
scala> val two: PartialFunction[Int, String] = { case 2 => "two" }
two: PartialFunction[Int,String] = <function1>

scala> val three: PartialFunction[Int, String] = { case 3 => "three" }
three: PartialFunction[Int,String] = <function1>

scala> val wildcard: PartialFunction[Int, String] = { case _ => "something else" }
wildcard: PartialFunction[Int,String] = <function1>

scala> val partial = one orElse two orElse three orElse wildcard
partial: PartialFunction[Int,String] = <function1>

scala> partial(5)
res4: String = something else

scala> partial(3)
res5: String = three

scala> partial(2)
res6: String = two

scala> partial(1)
res7: String = one

scala> partial(0)
res8: String = something else

scala> partial.isDefinedAt(10)
res9: Boolean = true

scala> partial.isDefinedAt(1000)
res10: Boolean = true
```

範例截自：[Twitter Scala School - Partial Function](https://twitter.github.io/scala_school/pattern-matching-and-functional-composition.html#PartialFunction)

### 總結

開始使用 Functional Lanague 時，思維需要做改變，程式設計時，以往用 OO 處理的設計，要轉換到是否可以切割成 Function 來處理，尤其是 Function 要符合數學函數或 Purly Function 的定義，Functional Language 程式的設計思維，會更倚重數學邏輯。


進階：

* [Wiki: Monad](http://en.wikipedia.org/wiki/Monad_(functional_programming))
* [Coursera: 響應式編程原理 - Week 1](https://zh-tw.coursera.org/course/reactive)