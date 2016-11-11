# Scala Other

## Lazy

Scala 允許資源在要使用時才載入。只要在宣告時，加 `lazy` 這個關鍵字。

eg: 取得資料庫連線

```
lazy val conn = DriverManager.getConnection
lazy val stmt = conn.preparedStatement(....)
lazy val rs = stmt.executeQuery()

try {
   stmt.setInt(1, xxx)
   
   while (rs.next) {
     ...
   }
}
catch {
  case ex: Exception =>
}
finally {
  DBUtils.close(rs)
  DBUtils.close(stmt)
  DBUtils.close(conn)
}
```

當執行到 `stmt.setInt(1, xxx)` 時，才會開始初始化 `PreparedStatement`，在初始化 `PreparedStatement` 時，才會去初始化 `Connection`。

## Implicit

### Implicit Parameter
Implicit 的技術在 Scala 使用很多，以 collection 為例，從 Java Collection 轉換成 Scala Collection 的版本，都是用 implicit 的方式來完成。

在 OOP 最常見的 implicit 是 `this` 這個關鍵字。在 OOP 的 Class 內，並沒有宣告 `this` 這個變數，卻可以使用。

Python 的設計哲學是不允許 implicit 的，因此寫 Python 常會宣告 `self` 來代表 `this`。

eg:

```
scala> implicit val a = 10
a: Int = 10

scala> def test(a: Int)(implicit b: Int) = a + b
test: (a: Int)(implicit b: Int)Int

scala> test(1000)
res0: Int = 1010
```

當宣告的參數是 `implicit` 時，Scala Compiler 會去找符合條件的 `implicit` 變數。

`implicit` 的設計會讓系統更加靈活，如 `Future` 使用的 Thread Pool。Scala 有內建 Thread Pool，但也可以自定 Thread Pool 後，修改 import 的部分後，其他的部分不用再修改。

附註： How `this` work (c++)

Source code: 

```
class CRect {	private: 		int m_color;	public: 		void setcolor(int color) {			m_color = color;		}}
```

After Compiling:

```
class CRect {	private: 		int m_color;	public: 		void setcolor(int color, (CRect*)this) {			this->m_color = color;		}}
```

### Implicit Conversions

在針對 Java Collection 轉成 Scala 相對應的 Collection，都使用 implicit function 在做轉換。

eg: Scala wrapAsScala

```
implicit def asScalaBuffer[A](l: ju.List[A]): mutable.Buffer[A] = l match {
  case MutableBufferWrapper(wrapped) => wrapped
  case _ =>new JListWrapper(l)
}
```

## Tail Recursion

Scala Compiler 會針對 Tail Recursion 做最佳化。所謂的 Tail Recursion 是指，把做 recursion 放在**最後一個指令**。

eg: GCD

```
def gcd(a: Int, b: Int): Int = if (a == 0) b else gcd(b % a, a)
```

Recursion 放在最後一行，Scala Compiler 會針對這類型的寫法做最佳化處理。

eg: **非** Tail Recursion，因為最後是 recursive call 後再 `+ 1`。

```
def boom(x: Int): Int = if (x == 0) throw new Exception("boom!") else boom(x - 1) + 1
```

執行的結果：

```
scala> boom(2)
java.lang.Exception: boom!
  at .boom(<console>:7)
  at .boom(<console>:7)
  at .boom(<console>:7)
  ... 33 elided
```

上例中，最後一行是 `boom(x - 1) + 1`，因此不是 Tail Recursion。執行的結果中，有三個 `.boom`，代表有三個 stack frame。所以是以傳統 recursive call 在進行。

需改寫成：

```
scala> def boom(x: Int): Int = if (x == 0) throw new Exception("boom!") else boom(x - 1)
boom: (x: Int)Int
```

結果：

```
scala> boom(2)
java.lang.Exception: boom!
  at .boom(<console>:8)
  ... 33 elided
```

改寫後的結果，只有一個 `.boom`。Scala Compiler 會把 Tail Recursion 改寫成只用一個 stack frame 來進行。

範例來自：

[Programming in Scala: A Comprehensive Step-by-Step Guide, 2nd Edition](http://www.amazon.com/Programming-Scala-Comprehensive-Step-Step/dp/0981531644)