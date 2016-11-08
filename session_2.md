# 基本語法與 Data Type

## 前言
* Scala 的 `Tab` 是**兩個**空隔，與 Java 預設四個不同
* Scala 已經沒有 `break` 與 `continue` 關鍵字可用
* Scala 還保有 `return` 但不建議使用
* Scala 還保有 `throw` Exception，但不建議使用。建議 refactor 成 Option or Either；IO相關可例外。
* Scala 依然可以用 `try`-`catch`-`finally`。

## 變數宣告方式

變數的宣告方式如下：

```
var 變數名稱: 資料型別 = 值
val 變數名稱: 資料型別 = 值
```
ex:

```
val a: Int = 10
var b: Double = 20.0

```

其中，資料型別可以省略，如下：

```
val a = 10
var b = 20.0
```

`val` 等同在 java 使用 `final`。

`var` 則如同 java 一般的宣告。

scala 鼓勵儘量使用 `val`。

使用 `var` 宣告時，可以使用 `_` (萬用字元) 當預設值，此時 scala 會依型別，給予預設值。`val` 則不可用 `_`。

```
var a: Int = _ /* a 會是 0 */

class Test
var b: Test = _ /* b 會是 null */
```

## Java Primitive Type
Java primitive type are

* byte
* char
* short
* int
* long
* float
* double
* boolean

以上這些在 Scala 是被轉成所謂的 `Value Class` 全都繼承自 `AnyVal`。

Scala 定義的 `Value Class` 有：

* Unit -> ()
* Byte -> byte
* Char -> char
* Short -> short
* Int -> int
* Long -> long
* Float -> float
* Double -> double
* Boolean -> boolean

**注意：省略型別時，整數值會預設是 `Int`，浮點數值會是 `Double`。**

## String
Scala 的 String **等於** Java 的 String。唯一要注意的是 `.equals`。在 Java 比對字串的內容時，要使用 `.equals` 不能使用 `==`。但在 Scala 則可以。

**In Java**

```
String str = "hello world!";
"hello".equals(str);

```

**In Scala**

```
val str = "Hello world!"
"hello" == str

```

###Scala 的字串有兩個方便的功能，讓寫作程式更方便

* String Interpolation：字串與變數結合，字串的開頭為`s`, 可使用 `$` 來將變數加入字串中，或者用 `${}`將 statement 加入。讓程式寫作更方便，可讀性也變高。

```
val a = 10
val b = 20

println(s"$a + $b = ${a + b}")
```

* Raw String Delimiter：在 Scala 中，可以使用 `"""`，不處理脫序字元，且可多行。這項功能在宣告正規表示式很好用。

```
/* Java 列印 \r\n */
System.out.println("\\r\\n");

/* Scala */
println("""\r\n""")

/* 多行字串 */
val a = """abc
def
  ccc
eee
"""
println(a)

/* Scala Regex */
val a = """\d+""".r
```

## Option
`Option` 是用來取代 `null`。`Option` 有兩個 subclass：

* Some
* None

宣告時用 generic 方式，來指定回傳的型別。

從`Option` 取值前，先用 `isDefined` 來判斷是否是 `Some`，是的話，再用 `get` 來取值。

```
def parseInt(str: String): Option[Int] = {
  try {
    Option(str.toInt)
  }
  catch {
    case ex: Exception => None
  }
}

val a = parseInt("abc")

if (a.isDefined) println(a.get) else println("None")

```
在使用 Functional Languge 方式時寫程式時，不會這樣子做。

## Tuple
Tuple 可以將兩種以上，不同型別的資料組合起來使用；可以把它當作更精簡的 Bean 來看。

宣告：

```
val a: (Int, String) = (10, "ABC")
```
或省略型別

```
val a = (10, "ABC")
```
取值使用時，從 **_1** 開始，指定第一個值。以此類推。

```
val a = (10, "ABC")

println(a._2)
```

**Tuple 最多只可以用到 22 個參數。**

## Function

完整的 function 宣告如下：

```
def 函數名稱(變數名稱: 型別, 變數名稱: 型別, ...): 回傳型別 = {
  函數內容
}

/* 等同 Java:
boolean test(int a, int b) {
    return a == b;
} */
def test(a: Int, b: Int): Boolean = {
  a == b
}

/* 等同 Java:
void test(int a) {
    System.out.println(a);
} */
def print(a: Int): Unit = {
  println(a)
}
```
**函式的回傳值，是看函式的最後一個 statement 的回傳值決定**

簡潔的寫法：

* 沒有回傳值時

```
原：
def test(a: Int): Unit = {

}

簡：
def test(a: Int) {

}
```

* 省略寫回傳值，依函式的最後一行來決定

```
原:
def test(a: Int): Int = {
  val b = a + 10
  b
}

簡：
def test(a: Int) = {
  val b = a + 10
  b
}
```

* 沒有傳入參數時，可以省略 `()`，呼叫時，也可以省略。

```
原：
def test(): Int = {
  val a  = 10
  a
}

簡：
def test: Int = {
  val a = 10
  a
}

再簡：
def test = {
  val a = 10
  a
}

呼叫時：
test
```

* 如果函式只有一行 statement，`{` 和 `}` 可省略

```
原：
class Bean {
  private var age: Int = 0

  def getAge(): Int = {
    age
  }

  def setAge(a: Int): Unit = {
    age = a
  }
}

簡：
class Bean {
  private var age = 0

  def getAge = age

  def setAge(a: Int) = age = a

}

使用：
val b = new Bean
b.setAge(10)
b.getAge
```

### Function 進階
* 傳入的參數，預設都是 `val`，也就是不能再修改值。

```
def test(a: Int) = {
  a = 10 /* error: reassignment to val */
}
```

* Named Arguments：在呼叫函式時，可以加入函式宣告的參數名稱，尤其是當函式的參數很多時，來增加可讀性；有指定參數的名稱時，就不一定要依照函式參數的順序傳入。

```
def test(a: Int, b: Int) = a + b

/* a,b 的順序就可互換 */
test(b = 10, a = 30)
```

* Default Parameter Value：宣告函式時，可以給定參數預設值，當呼叫時，沒有傳入值時，就會使用此預設值。這項功能，常見於動態語言, ex: PHP。在設計程式，或者做 refactor 時，可利用這項特性，來增強相容性。

```
def test(a: Int, b: Int = 0) = a + b

test(10) /* b 會用 0 傳入 */

test(10, 20)
```

refactor 案例：

```
原始：
def test(a: Int): Int = a + 10
```

後來希望可以控制 10 這個值，函式改寫成：

```
def test(a: Int, b:Int) = a + b
```

但已經存在的程式碼，太多地方使用，不想去更動它，所以可以寫成：

```
def test(a: Int, b: Int = 10) = a + b
```

Java 也行。但要多一個函式的宣告：

```
原：

int test(int a) {
    return a + 10;
}

改：

int test(int a, int b) {
    return a + b;
}

int test(int a) {
    return test(a, 10);
}
```

**有預設值的參數，不一定是要放在最後，但建議放在最後。目前我遇到有設計這項功能的程式語言，都會限定要放在最後。**

```
/* 沒有放在最後的話，就要給定參數名稱 */
def test(a: Int, b: Int = 0, c: Int) = a + b + c

test(10, 20) /* error: not enough arguments for method test */

test(a = 10, c = 20)
```

* Repeapted arguments: 可以宣告不定個數的參數的函式，並且可以使用 Array 或 List 傳入。

```
def test(args: String*) {
  args.foreach { println }
}

test("1", "2")

test("A", "B", "C")

val args = Array("a", "b", "c", "d")
test(args: _*)

val lst = List("a", "1", "b", "2")
test(lst: _*)
```

## IF - ELSE IF - ELSE
在 Scala 拿掉了 Java 的三元運算子 `?:` (ex: w = x ? y : z)；需改用 `if else`

```
val w = if (x) y else z
```

也從這個例子中，Scala 在 `if - else` 的設計，是可以回傳值，在程式寫作時，可以多利用，ex:

```
/* Java */
int a = 0;

if (xx)
  a = 10;
else if (yy)
  a = 20;
else
  a = 30;

或者
int a = xx ? 10 : (yy ? 20 : 30);


/* Scala */
val a = if (xx) 10 else if (yy) 20 else 30
```

## While Loop
`while` 的使用方式，跟 java 一樣

```
def gcdLoop(x: Long, y: Long): Long = {  var a = x  var b = y  while (a != 0) {    val temp = a
    a = b%a    b = temp  }  b
}

var line = ""do {  line = readLine()  println("Read: "+ line)} while (line != "")
```

## For Loop
Scala 的 `for` 語法，跟 Java 的 `for` 用在 `Iterable` 很像。

```
/* 列印 1 ~ 9 */
for (i <- 1 to 9)
  println(i)

for (i <- 1 until 10) {
  println(i)
}

/* 九九乘法 */
for (i <- 1 to 9) {
  for (j <- 1 to 9)
    println(s"$i x $j = ${i + j}")
}
```

可以在 `for` 加 `if` 判斷，多個 `if` 時，是用 `and` 來判斷

```
/* 列印 1 ~ 100 的偶數 */
for (i <- 1 to 100 if i % 2 == 0)
  println(i)

/* 多個判斷 */
for (i <- 1 to 100
     if i % 2 == 0
     if i % 5 == 0
) println(i)
```

也可以將 nested for 縮成一個 for。使用時，要用 `;` 來區分 nested for

```
/* 九九乘法 */
for (i <- 1 to 9;
     j <- 1 to 9
) println(s"$i x $j = ${i * j}")
```

使用 `for yield` 組出 collection

```
val a = for (i <- 1 to 9) yield i

val b = for (i <- 1 to 9) yield {
  if (i % 2 == 0) i
  else 0
}
/* 可以把 else 0 拿掉試看看 */
```

Scala 的 `for` 雖然語法上跟 Java 很像，但底層的實作卻是大不相同。 Scala 的 `for` 主要是依循 Functional Language 的 **Monads** 模型做出來的。因此在使用 `for yield` 時，要去注意回傳 collection 的型別是什麼。

**當轉換到 Functional Language 的思維時，其實 `for` 就會很少使用。都會直接用 monads 的模型在實作。**

## MATHCH CASE
`match case` 是 Scala 最重要的功能之一，使用的概念與 Java 的`switch case` 類似，但有更強大的功能。

由於 Scala 拿掉了 `break`，所以在每個 `case` 執行完後，會直接離開 `match` block。這個與 Java `switch case` 有很大的差別；在 Java 的 `case` 如果沒有下 `break`，會往下一個 `case` 繼續執行。

Scala 的 default case 的寫法是： `case _`，使用萬用字元 `_` 當 default case。

```
val a = 10

a match {
  case 1 =>
    println(1)  
  case 2 =>
    println(2)
  case _ =>
    println("other")
}

val b = "abc"

b match {
  case "def" =>
    println("DEF")

  case "abc" =>
    println("ABC")

  case _ =>
    println(b)
}
```

進階用法：

* case 加入型別及 `if` 的判斷

```
val a = 10

a match {
  case x if (x % 2 == 0) =>
    println("event")
  case _ =>
    println("odd")
}
```

* `match case` 可以回傳值

```
val a = 99

val isEvent = a match {
  case x if (a % 2 == 0) => true
  case _ => false
}
```

在 Scala 使用 `match case` 時，雖然不強制一定要給 default case，但還是
養成好習慣，寫入 default case 要做什麼處理，當然也可以不做任何事情。

Scala 如果比對不到時，會出現 Runtime Error。

```
val a = 5

a match {
  case a < 5 =>
    println("a < 5")

  case _ => /* 不做任何處理 */
}
```

**不同的程式語言對於 match(switch) case 設計不盡相同，像 C++, Java 需要下 break 才會離開 switch block。Scala 與 Apple Swift 則不用。**

**推薦各位學 Scala 還有一項優點，就是熟 Scala 後，想再去學 Apple Swift 會非常快。Swift 的語法與設計概念，有很多與 Scala 雷同，而且比 Scala 更好用。**

## TRY - CATCH - FINALLY
scala 還保留 `try`，`catch`，`finally`，使用觀念與注意事項與 Java相同。

* `finally` 是最後一定會被執行的區塊，**千萬不要**在此用 `return` 回傳值。**切記：Scala 禁用 `return`。**
* `catch` 的語法，有點像 `match case`，變成 `catch case`。
* 與 Java 不同的地方，就是 `try - catch` 可以回傳值

語法示意：

```
try {
  .....
}
catch {
  case ex: IOException =>
    ...

  case ex: SQLException =>
    ...

  case ex: Exception =>
    ...
}
finally {
  ....
}
```

ex:

```
/* 注意 try catch 是可以回傳值的 */
def parseInt(a: String) = {
  try {
    Option(a.toInt)
  }
  catch {
    case ex: Exception => None
  }
  finally {
    println("end")
  }
}
```

**再次提醒，不論 Java 或 Scala 都千萬不要在 `finally` 用 `return`。**

```
def f: Int = try { return 1 } finally { return 2 }
def g: Int = try { 1 } finally { 2 }

f /* return 2 */
g /* return 1 */
```
