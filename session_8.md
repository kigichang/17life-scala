# Scala Error Handle

在 Function Language 中，Exception 是一種 Side Effect。在實際的環境中，只要跟 I/O 相關的，都會需要處理 Exception。Scala  除了保留原本 Java 的 `try`-`catch`-`finally`的機制外，提供下列三種方式來處理 Exception，再結合 `Option` 的方式，讓程式更可以專注在資料處理上。

## Try
**Try** 與 **Option** 相似，本身有兩個 subclass: **Success** 及 **Failure**。當沒有發生 Exception 時，會回傳 **Success** 否則回傳 **Failure**。

舉例：

```
scala> def parseInt(value: String) = Try { value.toInt }
parseInt: (value: String)scala.util.Try[Int]
```

Try 常用的幾個 Function:

* map

```
scala> val t1 = parseInt("1000") map { _ * 2 }
t1: scala.util.Try[Int] = Success(2000)

scala> val t2 = parseInt("abc") map { _ * 2 }
t2: scala.util.Try[Int] = Failure(java.lang.NumberFormatException: For input string: "abc")
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

* 註1: Try 只會處理所謂的 **NonFatal** Exception；**不**包含：
	* VirtualMachineError
	* ThreadDeath
	* InterruptedException
	* LinkageError
	* ControlThrowable

* 註2: Option 及 Try 都是 ADT (Algebraic Data Type)

## Catching

## Either

## Java: try - catch - finally