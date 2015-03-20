# 基本語法與 Data Type

## 變數宣告方式

變數的宣告方式如下：

```
var 變數名稱: 資料型別 = 值
val 變數名稱: 資料型別 = 值

ex:

val a: Int = 10
var b: Double = 20.0

```

其中，資料型別可以省略，如下：

```
val a = 10
var b = 20.0
```

**注意：省略型別時，整數值會預設是 `Int`，浮點數值會是 `Double`。**

## Value Class

[http://www.scala-lang.org/files/archive/spec/2.11/12-the-scala-standard-library.html](http://www.scala-lang.org/files/archive/spec/2.11/12-the-scala-standard-library.html)