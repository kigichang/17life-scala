# Scala Functional Language 簡介

Functional Language 其實很早就有了，只是 OOP 概念比較能讓人接受，因此 Functional Language 就被忽略。近來大數據需要做大量平行與分散式的運算，Functional Lanauge 才又被重視。

Functional Language 簡單來說，就是：

Treats computation as the evaluation of **mathematical functions** and avoids **changing-state and mutable data (Side Effects)**

截自 [Wiki: Functional Lanauge](http://en.wikipedia.org/wiki/Functional_programming)

## 名詞解釋

### Mathematical Functions

Mathematical Functions 簡單來說，就是兩個集合間的關係，且每一個輸入值，只會對應到一個輸出值。

最簡單的數學函數的表示式：

$$f: X \mapsto Y$$

比如說：Square

$$Square: Int \mapsto Int$$
$$f(x) = x^2$$

比如 $f(3) = 9$, $f(-3) = 9$ ，每次計算 $f(3)$ 一定都會是 **9** 不會變成其他值。

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

```scala {.line-numbers}
val x = "Hello, World"
val r1 = x.reverse
val r2 = x.reverse
```

此時，可以將上述的 `x` 直接替換成 `Hello, World`，程式的結果都不會被改變。此特性，就是 **Referential Transparency**。如下：

```scala {.line-numbers}
val r1 = "Hello, World".reverse
val r2 = "Hello, World".reverse
```

另舉反例：StringBuilder 是 mutable，`append` 會修改 StringBuffer 內的值 (change object state)。

```scala {.line-numbers}
val x = new StringBuilder("Hello")
val y = x.append(", World")

val r1 = y.toString
val r2 = y.toString
```

當將 `y` 替換成 `x.append(", World")` 時：

```scala {.line-numbers}
val r1 = x.append(", World").toString

val r2 = x.append(", World").toString
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

```scala {.line-numbers}
val max = (x: Int, y:Int) => if (x > y) x else y
max: (Int, Int) => Int = <function2>

max(3, 4)
```

#### High Order Function

Hight Order Function 是指 Function 其中一個參數的資料型別是 Function。比如 `List` 的 `foreach`。

```scala {.line-numbers}
List(1, 2, 3, 4) foreach { x => println(x + x) }
```

### Function Composition

數學的複合函數：

$$f: X \mapsto Y$$

$$g: Y \mapsto Z$$

$$	g \circ f: X \mapsto Z$$

在 Scala 上的實作，有 `compose` 及 `andThen`

`f andThen g` 等同於 $  g \circ f $

`f compose g` 等同於 $ f \circ g $

eg:

```scala {.line-numbers}
val f = (x: Int) => x * x
val g = (x: Int) => x + 1

val goff = f andThen g
goff(10)

val fofg = f compose g

fofg(10)
```

#### 轉成 Function Class

一般會用 `def` 宣告 function；可以使用 **`_`**  轉換成 Function Class。如下：

```scala {.line-numbers}
def f(x: Int) = x * x
f: (x: Int)Int

def g(x: Int) = x + 1
g: (x: Int)Int

f andThen g  // ERROR

f _ andThen g _
```

### Partially Applied Function

```scala {.line-numbers}
def sum(x: Int, y: Int, z: Int) = x + y + z

val a = sum _
a: (Int, Int, Int) => Int = <function3>

val b = sum(1, _: Int, 3)
b: Int => Int = <function1>

b(2)
res1: Int = 6
```

### Closure

A function object that captures free variables, and is said to be **closed** over the variables visible at the time it is created.

Closure 的功能，主要是讓 Function 可以與所處的環境做連結，以前在函式內，要引用外部的變數，有兩種方式：

1. 將要引用的變數，當參數傳入
    1. 要特別注意參數的傳入方式：By Name, By Value, By Reference
1. 將此變數變成 Global 變數

Closure 提供第三種方式，可以存取外部的變數。

舉例：

```scala {.line-numbers}
var more = 10

val addMore = (x: Int) => x + more

addMore(10)	// 20

more = 30

addMore(10) // 40


val lst = List(1, 2, 3, 4, 5)

var sum = 0

lst foreach { x => sum += x }	// sum = 15
```

`addMore` 是一個 **Closure**. `more` 這個變數是 **free variable**.  `x` 是 **bounded variable**.

### Currying

假設有個 function 由兩個以上的集合對應到一個集合：

$$f: X \times Y \mapsto Z$$

比如說：

$$f(x, y) = \frac{y}{x}$$

我們可以定義一個 **Curring Function** ：

$$h(x) = y \mapsto f(x, y)$$

$h(x)$ 是一個 Function ，它的輸入值是 x ，回傳值是 Function 。

比如說：

$$h(2) = y \mapsto f(2, y)$$

這時候的 $ h(2) $ 是一個 Function：

$$ f(2, y) = \frac {y}{2} $$

此時，我們可以再定義一個 Function: $g(y) $

$$g(y) = h(2) = y \mapsto f(2, y)$$

也就是

$$g(y) = f(2, y) = \frac {y}{2}$$

在 Scala 的實作：

定義  $ f: X \times Y \mapsto Z $

```scala {.line-numbers}
def f(x: Int)(y: Int) = y / x

f(4)(2)
f(2)(4)
```

定義 $ g(y) = h(2) = y \mapsto f(2, y) $ i.e. 
$$g(y) = f(2, y) = \frac {y}{2} $$

```scala {.line-numbers}
val h = f(2) _
```

當  $ y = 4 $ 時，

$$g(4) = f(2, 4) = \frac {4} {2} = 2$$

```scala {.line-numbers}
h(4)
```
範例截自 [Wiki: Curring](http://en.wikipedia.org/wiki/Currying)

 另一種使用時機：
 
```scala {.line-numbers}
def modN(n: Int)(x: Int) = ((x % n) == 0)

val nums = List(1, 2, 3, 4, 5, 6, 7, 8)

nums filter { modN(2) }
nums filter { modN(3) }
```

數學式對應：

$$modN: Int \times Int \mapsto Boolean$$

$$modN(n, x) \Rightarrow ((x \bmod n ) == 0)$$

$$mod2(x) = modN(2, x) \Rightarrow ((x \bmod 2) == 0)$$

$$mod3(x) = modN(3, x) \Rightarrow ((x \bmod 3) == 0)$$

範例截自：[Scala Document: Currying](http://docs.scala-lang.org/tutorials/tour/currying.html)

### Scala Partial Function

一般定義 Function 都會去處理輸入值的所有情況。比如說：

```scala {.line-numbers}
def plusOne(x: Int) = x + 1
```

所有 Int 整數值，傳進 `plusOne` 都會被處理。

**Partial Function** 換言之就是只處理某些狀況下的值。

定義：

```scala {.line-numbers}
val one: PartialFunction[Int, String] = { case 1 => "one" }
```

使用：如果輸入沒有要處理的值時，會出現 **Exception**。比如 **1** 有定義，但 **2** 沒有，所以輸入 **1** 沒問題，輸入 **2** 就會有 **Exception**。

```scala {.line-numbers}
one(1)

one(2)	// ERROR
```

查詢輸入值，是否已在處理的範圍內：

```scala {.line-numbers}
one.isDefinedAt(1)

one.isDefinedAt(2)
```

#### Composition of Partial Function

可以使用多個 Partial Function 組成一個複合函數。

```scala {.line-numbers}
val two: PartialFunction[Int, String] = { case 2 => "two" }

val three: PartialFunction[Int, String] = { case 3 => "three" }

val wildcard: PartialFunction[Int, String] = { case _ => "something else" }


val partial = one orElse two orElse three orElse wildcard

partial(5)

partial(3)

partial(2)

partial(1)

partial(0)

partial.isDefinedAt(10)

partial.isDefinedAt(1000)
```

範例截自：[Twitter Scala School - Partial Function](https://twitter.github.io/scala_school/pattern-matching-and-functional-composition.html#PartialFunction)

### 總結

開始使用 Functional Lanague 時，思維需要做改變，程式設計時，以往用 OO 處理的設計，要轉換到是否可以切割成 Function 來處理，尤其是 Function 要符合數學函數或 Purly Function 的定義，Functional Language 程式的設計思維，會更倚重數學邏輯。

進階：

* [Wiki: Monad](http://en.wikipedia.org/wiki/Monad_(functional_programming))
* [Coursera: 響應式編程原理 - Week 1](https://zh-tw.coursera.org/course/reactive)