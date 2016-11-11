# Scala Error Handle

在 Function Language 中，Exception 是一種 Side Effect。在實際的環境中，只要跟 I/O 相關的，都會需要處理 Exception。Scala  除了保留原本 Java 的 `try`-`catch`-`finally`的機制外，提供下列三種方式來處理 Exception，再結合 `Option` 的方式，讓程式更可以專注在資料處理上。

## Java: try - catch - finally

一般寫法：

```
try {
  ...
} catch {
	case ex: Exception =>
	  ...
} finally {
  ...
}
```

千萬不要這麼寫：

```
try {
  ...
} catch {
  case _ =>
    ...
} finally {
 ...
}
```

或

```
try {
  ...
} catch {
  case _: Throwable =>
    ...
} finally {
  ...
}
```

偷懶的寫法：使用 `NonFatal`

```
import scala.util.control.NonFatal

try {
  ...
} catch {
  case NonFatal(_) =>
    ...
} finally {
  ...
}
```

Q: 為什麼不能使用：`case _ =>` or `case _: Throwable =>` ?

A: `Throwable` 有兩個 subclass: `Exception` 及 `Error`，一般在 Java 我們都是處理 `Exception`。`Error` 通常都是 JVM 發生重大錯誤時發出，如: `OutOfMemoryError`；此時應該是讓 JVM 中止執行，而不是繼續執行。

**NonFatal** 不會處理以下錯誤：

* VirtualMachineError (包括 **OutOfMemoryError** and other fatal errors)
* ThreadDeath
* InterruptedException
* LinkageError
* ControlThrowable


## Try
**Try** 與 **Option** 相似，本身有兩個 subclass: **Success** 及 **Failure**。當沒有發生 Exception 時，會回傳 **Success** 否則回傳 **Failure**。

舉例：

```
scala> import scala.util.Try
import scala.util.Try
scala> def parseInt(value: String) = Try { value.toInt }
parseInt: (value: String)scala.util.Try[Int]
```

Try 常用的幾個 Function:

* map

```
scala> val t1 = parseInt("1000") map { _ * 2 }
t1: scala.util.Try[Int] = Success(2000)

scala> for (t1 <- parseInt("1000")) yield t1
res0: scala.util.Try[Int] = Success(1000)


scala> val t2 = parseInt("abc") map { _ * 2 }
t2: scala.util.Try[Int] = Failure(java.lang.NumberFormatException: For input string: "abc")

scala> for (t2 <- parseInt("abc")) yield t2
res1: scala.util.Try[Int] = Failure(java.lang.NumberFormatException: For input string: "abc")
```

* recover

```
scala> import scala.util.control.NonFatal
import scala.util.control.NonFatal

scala> t1 recover { case NonFatal(_) => -1 }
res5: scala.util.Try[Int] = Success(2000)

scala> t2 recover { case NonFatal(_) => -1 }
res6: scala.util.Try[Int] = Success(-1)
```

* toOption

```
scala> t1.toOption
res7: Option[Int] = Some(2000)

scala> t2.toOption
res8: Option[Int] = None
```

* getOrElse

```
scala> t1.getOrElse(-1)
res9: Int = 2000

scala> t2.getOrElse(-1)
res10: Int = -1
```

* 註1: Try 使用 **NonFatal** 來處理 Exception。
* 註2: Option 及 Try 都是 ADT (Algebraic Data Type)

## Catch

Catch 是用來處理 `catch` 及 `finally`。搭配 `Option` 及 `Either` 來處理 Exception。

```
scala> import scala.util.control.Exception._
import scala.util.control.Exception._

scala> def parseInt(value: String) = nonFatalCatch[Int] opt { value.toInt }
parseInt: (value: String)Option[Int]

scala> def parseInt(value: String) = nonFatalCatch[Int] either { value.toInt }
parseInt: (value: String)scala.util.Either[Throwable,Int]

scala> def parseInt(value: String) = nonFatalCatch[Int] andFinally { println("finally") } opt { value.toInt }
parseInt: (value: String)Option[Int]

scala> def parseInt(value: String) = nonFatalCatch[Int] andFinally { println("finally") } opt { println("begin"); value.toInt }
parseInt: (value: String)Option[Int]

scala> parseInt("abc")
begin
finally
res2: Option[Int] = None

scala> parseInt("123")
begin
finally
res3: Option[Int] = Some(123)

scala> def parseInt(value: String) = catching(classOf[Exception]) opt { value.toInt }
parseInt: (value: String)Option[Int]

scala> parseInt("456")
res5: Option[Int] = Some(456)
```

## Either

`Either` 可以讓 Fuction 達到回傳不同型別資料效果。`Either` 有兩個 subclass: `Right` 及 `Left`。可以使用 `match`-`case` 來確認是回傳 `Right` or `Left`；進而了解是成功或失敗。

```
scala> def parseInt(value: String) = try { Right(value.toInt) } catch { case ex: Exception => Left(value) } 

parseInt: (value: String)Product with Serializable with scala.util.Either[String,Int]

scala> parseInt("123") match {
     | case Right(v) => println(s"success ${v}")
     | case Left(s) => println(s"failure ${s}")
     | }
success 123

scala> parseInt("abc") match {
     | case Right(v) => println(s"success ${v}")
     | case Left(s) => println(s"failure ${s}")
     | }
failure abc
```