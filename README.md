# Scala / Akka 分享
### 參考資料
* [Scala 官網](http://www.scala-lang.org/)
* [Twitter Scala School](http://twitter.github.io/scala_school/)
* [Effective Scala - Twitter 使用 Scala 心得與建議](http://twitter.github.io/effectivescala/)
* [參考書 - 作者寫的，使用版本比較舊，但可以學到設計的原理](http://www.amazon.com/Programming-Scala-Comprehensive-Step-Step/dp/0981531644)
* [Coursera 課程](https://www.coursera.org/course/progfun)

### 前言
* Scala 簡介
* 由 Java 跨入 Scala 要注意的事項 - 安裝好 Scala (時間：2015-03-18 09:30 ~ 10:30)
  * 原來沒有分號 `;` 這麼爽
  * 不再限制一個 source file，只能有一個 public class
  * Primitive v.s. Rich
  * Null v.s. Option
  * Can Ignore Checked Exception 
  * Access Level
  * 沒有 return, break, continue 世界 - 思維需要 Reset
  * 簡潔的 package 宣告
  * 宣告後，會是 Unit 不會再傳宣告值
   
  ```
  scala: val a = 0 <- Unit
  java: int a = 0 <- 0
  ```
  * 沒有 `++`, `--`；強制用 `+=`, `-=`
  
  ```
  scala> var a = 0
  a: Int = 0

  scala> val b = (a += 1)
  b: Unit = ()
  ```
   
### 先把 Scala 當 Java 寫；不過當然不行
* 基本語法與 Data Type
  * val v.s. var
  * can ignore data type when declaration
  * primitive variable 宣告
  * function 宣告
      * default value
      * named argument
  * if else - 沒有三元運算; 可回傳值 
  * for yield - 可回傳值
  * while
  * match case - switch case
  * tuple - 似 Bean 非 Bean。更簡潔靈活的 DataType
  * Option
* 物件導向 - Eclipse 開發環境
  * OOP 簡介
      * access level: private, protected, public, package
      * override
  * class
  * object - simple singleton
  * trait - multi-inheritance
  * enum
  * generic
* Scala Collection
  * mutable v.s. immutable
  * List
  * Map
  * Array

### 進入 Scala 世界
* Activator and SBT - Scala 的類 Maven 工具
* 再談 function
  * High Order Function
  * Currying
  * Partial Function
* 再談 OOP
  * 自定運算子
  * 左，右運算子
  * case class
* 再談 match case - Pattern Match
  * apply and unapply
  * pattern match
  * Regular Expression
  * List operator
* Map-Reduce 
  * map
  * flatMap
  * reduce
* 其他
  * Tail Recursion - 優化的遞迴
  * Implicit v.s. Explicit - 讓code更簡潔，但也更討厭的鬼東西
  * Try, Success, Failure - 重新改寫 try - catch - finally
  * Either, Left, Right - 向左走，向右走
  * Multi-Threading: Future, Await
  * lazy

### Akka - Parallel / Distributed Computing
* 基本概念 - Parallel Computing
* Distributed Computing
* Routing
* SuperVisor

### Playframework - 一起玩 web
* 基本功能
  * Routing
  * Cookie
  * Session (偽 Session)
* Form
* JDBC
* JSON
