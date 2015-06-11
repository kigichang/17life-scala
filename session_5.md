# Scala Functional Language 簡介

Functional Language 其實很早就有了，OOP 概念比較能讓人接受，因此 Functional Language 就被忽略。近來大數據需要做大量平行與分散式的運算，Functional Lanauge 才又被重視。

Functional Language 簡單來說，就是：

treats computation as the evaluation of **mathematical functions** and avoids **changing-state and mutable data (Side Effects)**

截自 [Wiki: Functional Lanauge](http://en.wikipedia.org/wiki/Functional_programming)

## 名詞解釋

### Mathematical Functions

Mathematical Functions 簡單來說，就是兩個集合間的關係，且每一個輸入值，只會對應到一個輸出值。

比如說：Square

\\[
f(x) = x^2
\\]

比如 f(3) = 9, f(-3) = 9 ，每次計算 f(3) 一定都會是 **9** 不會變成其他值。

### Side Effect

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

舉例來說：String 是 immutable，當每次呼叫 `reverse` 時，都會回傳固定的值。

```
scala> val x = "Hello, World"
x: String = Hello, World

scala> val r1 = x.reverse
r1: String = dlroW ,olleH

scala> val r2 = x.reverse
r2: String = dlroW ,olleH
```

此時，可以將上述的 `x` 直接替換成 `Hello, World`。此特性，就是 **Referential Transparency**。如下：

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

會發現 `r1` 及 `r2` 的值並不一致，這樣子就沒有 **Referential Transparency** 。

範例來自 [Functional Language in Scala](http://www.amazon.com/Functional-Programming-Scala-Paul-Chiusano/dp/1617290653)

#### 為什麼 Referential Transparency 如此重要

當程式設計都符合 Referential Transparency 時，就代表程式可以分散在不同Thread, CPU核心，甚至不同主機上處理(空間)，而且不論什麼時候被處理(時間)，都不會影響輸出的結果。

Funcational Language 的終極目標就是 Referential Transparency。

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

#### Function Composition

在數學上，有所謂的複合函數。

\\[
	f: X \to Y
\\]

\\[
	g: Y \to Z
\\]

\\[
	g \circ f: X \to Z
\\]

\\[
	 (g \circ f )(x) = g(f(x))
\\]

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

#### Currying

#### Partial Function

