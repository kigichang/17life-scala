# Scala Parallel Computing

平行化處理，很適合用在 I/O bound 的程式，讓 I/O 可以同時間被處理，讓 CPU 等待的時間縮到最短。

## Future and Await

以往在 Java ，是使用 `Thread` 來進行計算。Scala 提供 `Future` 來儲存**尚未**完成的結果，在使用 `Future` 時，需要 `import concurrent.ExecutionContext.Implicits.global`，這一行的用意是使用 Scala 內建的 Thread Pool。在使用 `Future`  時，會需要使用 `Thread` 來進行計算。

在 Multi-Thread 環境下，通常主程式需要等待所有的 Thread 完成後，才能結束程式。Scala 提供 `Await` 等待 Thread 執行的結果。

eg: 同時讀取兩個檔案的內容

```
package com.example

import scala.concurrent.Await
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future
import scala.concurrent.TimeoutException
import scala.concurrent.duration.Duration
import scala.io.Source

object FutureTest {
  
  def readFile(file: String): StringBuilder = {
    val ret = new StringBuilder
    
    Source.fromFile(file).getLines() foreach { line =>
      ret ++= (line + "\r\n")
    }
    
    ret
  }
  
  def main(args: Array[String]) {
    
    println("start")
    
    val time = System.currentTimeMillis()
   
    val future1 = Future { readFile("ufo_awesome_1.tsv") }
    val future2 = Future { readFile("ufo_awesome_2.tsv") }
    
    val result = Await.result(Future.sequence(Seq(future1, future2)), Duration.Inf)
    
    println(s"end and cost: ${System.currentTimeMillis() - time} ms")
  }
}
```

使用 `Future` 將 `readFile` 工作包裝起來，在  `val future1 = Future { readFile("ufo_awesome_1.tsv") }` 會產生一個 `Thread` 來處理，並且立即執行下一行的工作。最後使用 `Await.result` 取得結果，或者也可以使用  `Await.ready`。

如果需要等待多個 `Future` 的執行結果，先將需要等待的 `Future`，組成一個 `Seq`，再使用 `Future.sequence` 組合後回傳單一個 `Future`後，再使用 `Await`  等待執行的結果。

如果移除 `val result = Await.result(Future.sequence(Seq(future1, future2)), Duration.Inf)` 會發現主程式一下子就執行完畢。

### Callback

#### onSuccess

顧名思意就是當 `Future` 包裝的工作執行**成功**時，會執行的工作。

eg:

```
package com.example

import scala.concurrent.Await
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future
import scala.concurrent.TimeoutException
import scala.concurrent.duration.Duration
import scala.io.Source

object FutureTest {
  
  def readFile(file: String): StringBuilder = {
    val ret = new StringBuilder
    
    Source.fromFile(file).getLines() foreach { line =>
      ret ++= (line + "\r\n")
    }
    
    ret
  }
  
  def main(args: Array[String]) {
    
    println("start")
    
    val time = System.currentTimeMillis()
    
    println(s"${System.currentTimeMillis()} - create future")
    val future = Future { readFile("ufo_awesome_1.tsv"); println(s"${System.currentTimeMillis()} - read complete") }
    
    println(s"${System.currentTimeMillis()} - register onSuccess")
    future onSuccess {
      case sb => println(s"${System.currentTimeMillis()} - success")
    }
    
    println(s"${System.currentTimeMillis()} - await")
    
    val result = Await.result(future, Duration.Inf)
    
    println(s"end and cost: ${System.currentTimeMillis() - time} ms")
  }
}
```

結果：

```
start
1436100618543 - create future
1436100618816 - register onSuccess
1436100618818 - await
1436100620040 - read complete
end and cost: 1502 ms
1436100620042 - success
```

注意當宣告完 `onSuccess`  時，主程式並不會 `Future` 執行結束，而是往下繼續，一直到 `Future` 的工作完成後，才會執行 `onSuccess`。


#### onFailure

當 `Future` 內的工作，有發生 `Exception` or  `Error` 時 (也就是有 `Throwable` )。 `onFailure` 並不會做 `catch` 的動作。這一點要特別注意

eg:

```
package com.example

import scala.concurrent.Await
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future
import scala.concurrent.TimeoutException
import scala.concurrent.duration.Duration
import scala.io.Source
import scala.util.control.NonFatal

object FutureTest {
  
  def readFile(file: String): StringBuilder = {
    val ret = new StringBuilder
    
    Source.fromFile(file).getLines() foreach { line =>
      ret ++= (line + "\r\n")
    }
    
    ret
  }
  
  def main(args: Array[String]) {
    
    println("start")
    
    val time = System.currentTimeMillis()    
    
    println(s"${System.currentTimeMillis()} - create future")
    
    /* ufo_awesome_3.tsv 不存在*/
    val future = Future { readFile("ufo_awesome_3.tsv"); println(s"${System.currentTimeMillis()} - read complete") }
    
    println(s"${System.currentTimeMillis()} - register onSuccess")
    future onSuccess {
      case sb => println(s"${System.currentTimeMillis()} - success")
    }
    
    println(s"${System.currentTimeMillis()} - register onFailure")
    future onFailure {
      case ex: Exception => println(s"${System.currentTimeMillis()} - failure")
    }
    
    println(s"${System.currentTimeMillis()} - await")
    
    val result = Await.result(future, Duration.Inf)
    
    println(s"end and cost: ${System.currentTimeMillis() - time} ms")
  }
}
```

結果：

```
start
1436102289048 - create future
1436102289345 - register onSuccess
1436102289350 - register onFailure
1436102289351 - failure
1436102289352 - await
Exception in thread "main" java.io.FileNotFoundException: ufo_awesome_3.tsv (No such file or directory)
	at java.io.FileInputStream.open0(Native Method)
	at java.io.FileInputStream.open(FileInputStream.java:195)
	at java.io.FileInputStream.<init>(FileInputStream.java:138)
	at scala.io.Source$.fromFile(Source.scala:91)
	at scala.io.Source$.fromFile(Source.scala:76)
	at scala.io.Source$.fromFile(Source.scala:54)
	at com.example.FutureTest$.readFile(FutureTest.scala:19)
	at com.example.FutureTest$$anonfun$1.apply$mcV$sp(FutureTest.scala:44)
	at com.example.FutureTest$$anonfun$1.apply(FutureTest.scala:44)
	at com.example.FutureTest$$anonfun$1.apply(FutureTest.scala:44)
	at scala.concurrent.impl.Future$PromiseCompletingRunnable.liftedTree1$1(Future.scala:24)
	at scala.concurrent.impl.Future$PromiseCompletingRunnable.run(Future.scala:24)
	at scala.concurrent.impl.ExecutionContextImpl$AdaptedForkJoinTask.exec(ExecutionContextImpl.scala:121)
	at scala.concurrent.forkjoin.ForkJoinTask.doExec(ForkJoinTask.java:260)
	at scala.concurrent.forkjoin.ForkJoinPool$WorkQueue.runTask(ForkJoinPool.java:1339)
	at scala.concurrent.forkjoin.ForkJoinPool.runWorker(ForkJoinPool.java:1979)
	at scala.concurrent.forkjoin.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:107)
```

#### onComplete

當 `Future` 內的工作執行完畢，不論成功或失敗。

eg:

```
package com.example

import scala.concurrent.Await
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future
import scala.concurrent.TimeoutException
import scala.concurrent.duration.Duration
import scala.io.Source
import scala.util.control.NonFatal
import scala.util.Success
import scala.util.Failure

object FutureTest {
  
  def readFile(file: String): StringBuilder = {
    val ret = new StringBuilder
    
    Source.fromFile(file).getLines() foreach { line =>
      ret ++= (line + "\r\n")
    }
    
    ret
  }
  
  def main(args: Array[String]) {
    
    println("start")
    
    val time = System.currentTimeMillis()

    println(s"${System.currentTimeMillis()} - create future")
    
    /* ufo_awesome_3.tsv 不存在*/
    val future = Future { readFile("ufo_awesome_3.tsv"); println(s"${System.currentTimeMillis()} - read complete") }
        
    println(s"${System.currentTimeMillis()} - register onComplete")
    future onComplete {
      case Success(sb) => println(s"${System.currentTimeMillis()} - onComplete - success")
      case Failure(error) => println(s"${System.currentTimeMillis()} - onComplete - failure ${error.toString()}")
    }
    
    println(s"${System.currentTimeMillis()} - await")
    
    val result = Await.result(future, Duration.Inf)
    
    println(s"end and cost: ${System.currentTimeMillis() - time} ms")
  }
}
```

結果：

```
start
1436105851821 - create future
1436105852172 - register onComplete
1436105852175 - await
1436105852177 - onComplete - failure java.io.FileNotFoundException: ufo_awesome_3.tsv (No such file or directory)
Exception in thread "main" java.io.FileNotFoundException: ufo_awesome_3.tsv (No such file or directory)
	at java.io.FileInputStream.open0(Native Method)
	at java.io.FileInputStream.open(FileInputStream.java:195)
	at java.io.FileInputStream.<init>(FileInputStream.java:138)
	at scala.io.Source$.fromFile(Source.scala:91)
	at scala.io.Source$.fromFile(Source.scala:76)
	at scala.io.Source$.fromFile(Source.scala:54)
	at com.example.FutureTest$.readFile(FutureTest.scala:21)
	at com.example.FutureTest$$anonfun$1.apply$mcV$sp(FutureTest.scala:46)
	at com.example.FutureTest$$anonfun$1.apply(FutureTest.scala:46)
	at com.example.FutureTest$$anonfun$1.apply(FutureTest.scala:46)
	at scala.concurrent.impl.Future$PromiseCompletingRunnable.liftedTree1$1(Future.scala:24)
	at scala.concurrent.impl.Future$PromiseCompletingRunnable.run(Future.scala:24)
	at scala.concurrent.impl.ExecutionContextImpl$AdaptedForkJoinTask.exec(ExecutionContextImpl.scala:121)
	at scala.concurrent.forkjoin.ForkJoinTask.doExec(ForkJoinTask.java:260)
	at scala.concurrent.forkjoin.ForkJoinPool$WorkQueue.runTask(ForkJoinPool.java:1339)
	at scala.concurrent.forkjoin.ForkJoinPool.runWorker(ForkJoinPool.java:1979)
	at scala.concurrent.forkjoin.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:107)

```

#### 多個 Callback

一個 `Future` 允許有多個 `onSuccess`, `onFailure`, 及 `onComplete` 。執行的順序不一定會依照程式碼的順序。

eg:

```
package com.example

import scala.concurrent.Await
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future
import scala.concurrent.TimeoutException
import scala.concurrent.duration.Duration
import scala.io.Source
import scala.util.control.NonFatal
import scala.util.Success
import scala.util.Failure

/**
 * @author kigi
 */
object FutureTest {
  
  def readFile(file: String): StringBuilder = {
    val ret = new StringBuilder
    
    Source.fromFile(file).getLines() foreach { line =>
      ret ++= (line + "\r\n")
    }
    
    ret
  }
  
  def main(args: Array[String]) {
    
    println("start")
    
    val time = System.currentTimeMillis()

    /*
    val future1 = Future { readFile("ufo_awesome_1.tsv") }
    val future2 = Future { readFile("ufo_awesome_2.tsv") }
    
    val result = Await.result(Future.sequence(Seq(future1, future2)), Duration.Inf)
    */
    
    
    println(s"${System.currentTimeMillis()} - create future")
    //val future = Future { readFile("ufo_awesome_1.tsv"); println(s"${System.currentTimeMillis()} - read complete") }
    
    /* ufo_awesome_3.tsv 不存在*/
    val future = Future { readFile("ufo_awesome_3.tsv"); println(s"${System.currentTimeMillis()} - read complete") }
    
    
    println(s"${System.currentTimeMillis()} - register onSuccess")
    future onSuccess {
      case sb => println(s"${System.currentTimeMillis()} - success")
    }
    
    println(s"${System.currentTimeMillis()} - register onFailure")
    future onFailure {
      case ex: Exception => println(s"${System.currentTimeMillis()} - failure")
    }
    
    println(s"${System.currentTimeMillis()} - register onComplete")
    future onComplete {
      case Success(sb) => println(s"${System.currentTimeMillis()} - onComplete - success")
      case Failure(error) => println(s"${System.currentTimeMillis()} - onComplete - failure ${error.toString()}")
    }
    
    println(s"${System.currentTimeMillis()} - await")
    
    val result = Await.result(future, Duration.Inf)
    
    println(s"end and cost: ${System.currentTimeMillis() - time} ms")
  }
}
```

結果:

```
start
1436105970847 - create future
1436105971148 - register onSuccess
1436105971151 - register onFailure
1436105971153 - register onComplete
1436105971154 - failure
1436105971155 - await
1436105971155 - onComplete - failure java.io.FileNotFoundException: ufo_awesome_3.tsv (No such file or directory)
Exception in thread "main" java.io.FileNotFoundException: ufo_awesome_3.tsv (No such file or directory)
	at java.io.FileInputStream.open0(Native Method)
	at java.io.FileInputStream.open(FileInputStream.java:195)
	at java.io.FileInputStream.<init>(FileInputStream.java:138)
	at scala.io.Source$.fromFile(Source.scala:91)
	at scala.io.Source$.fromFile(Source.scala:76)
	at scala.io.Source$.fromFile(Source.scala:54)
	at com.example.FutureTest$.readFile(FutureTest.scala:21)
	at com.example.FutureTest$$anonfun$1.apply$mcV$sp(FutureTest.scala:46)
	at com.example.FutureTest$$anonfun$1.apply(FutureTest.scala:46)
	at com.example.FutureTest$$anonfun$1.apply(FutureTest.scala:46)
	at scala.concurrent.impl.Future$PromiseCompletingRunnable.liftedTree1$1(Future.scala:24)
	at scala.concurrent.impl.Future$PromiseCompletingRunnable.run(Future.scala:24)
	at scala.concurrent.impl.ExecutionContextImpl$AdaptedForkJoinTask.exec(ExecutionContextImpl.scala:121)
	at scala.concurrent.forkjoin.ForkJoinTask.doExec(ForkJoinTask.java:260)
	at scala.concurrent.forkjoin.ForkJoinPool$WorkQueue.runTask(ForkJoinPool.java:1339)
	at scala.concurrent.forkjoin.ForkJoinPool.runWorker(ForkJoinPool.java:1979)
	at scala.concurrent.forkjoin.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:107)
```

### Map 及 flatMap

`Future` 也有支援 `map` 及 `flatMap`。也就是說可以利用 `map` 及 `flatMap` 來進一步做資料處理。

eg: 取出檔案每一行後，計算每一行的長度，最後加總。

```
package com.example

import scala.concurrent.Await
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future
import scala.concurrent.TimeoutException
import scala.concurrent.duration.Duration
import scala.io.Source
import scala.util.control.NonFatal
import scala.util.Success
import scala.util.Failure

object FutureTest {
  
  def readFile(file: String): StringBuilder = {
    val ret = new StringBuilder
    
    Source.fromFile(file).getLines() foreach { line =>
      ret ++= (line + "\r\n")
    }
    
    ret
  }
  
  def main(args: Array[String]) {
    
    println("start")
    
    val time = System.currentTimeMillis()
 
    /* Map Start */
    
    val future1 = Future { Source.fromFile("ufo_awesome_1.tsv").getLines().toSeq }
    
    val future2 = future1 map { seq => 
      seq.map { _.length }
    }
    
    val result = Await.result(future2, Duration.Inf)
    println(s"total: ${result.reduce( _ + _)}")
    /* Map End */
    println(s"end and cost: ${System.currentTimeMillis() - time} ms")
  }
}
```

結果：

```
start
total: 75281071
end and cost: 1161 ms
```

eg: 結合兩個檔案的內容

```
package com.example

import scala.concurrent.Await
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future
import scala.concurrent.TimeoutException
import scala.concurrent.duration.Duration
import scala.io.Source
import scala.util.control.NonFatal
import scala.util.Success
import scala.util.Failure

object FutureTest {
  
  def readFile(file: String): StringBuilder = {
    val ret = new StringBuilder
    
    Source.fromFile(file).getLines() foreach { line =>
      ret ++= (line + "\r\n")
    }
    
    ret
  }
  
  def main(args: Array[String]) {
    
    println("start")
    
    val time = System.currentTimeMillis()    
    
    /* flatMap (for) start */
    
    val future1 = Future { readFile("ufo_awesome_1.tsv") }
    val future2 = Future { readFile("ufo_awesome_2.tsv") }
    
    val future3 = for (sb1 <- future1; sb2 <- future2) yield {
      sb1.toString + "\r\n" + sb2.toString
    }
   
    val result = Await.result(future3, Duration.Inf)
    println(s"total: ${result.length()}")
    
    /* flatMap (for) end */
    
    
    println(s"end and cost: ${System.currentTimeMillis() - time} ms")
  }
}
```
