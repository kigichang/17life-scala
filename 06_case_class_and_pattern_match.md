# Case Class and Pattern Match

## Case Class

宣告：

```scala
case class Person(name: String, age: Int)
```

Compiler 後，會有 `Person.class` 及 `Person$.class`

**scalap -private Person**: 

```
case class Person(name: scala.Predef.String, age: scala.Int) extends scala.AnyRef with scala.Product with scala.Serializable {
  val name: scala.Predef.String = { /* compiled code */ }
  val age: scala.Int = { /* compiled code */ }
  def copy(name: scala.Predef.String, age: scala.Int): Person = { /* compiled code */ }
  override def productPrefix: java.lang.String = { /* compiled code */ }
  def productArity: scala.Int = { /* compiled code */ }
  def productElement(x$1: scala.Int): scala.Any = { /* compiled code */ }
  override def productIterator: scala.collection.Iterator[scala.Any] = { /* compiled code */ }
  def canEqual(x$1: scala.Any): scala.Boolean = { /* compiled code */ }
  override def hashCode(): scala.Int = { /* compiled code */ }
  override def toString(): java.lang.String = { /* compiled code */ }
  override def equals(x$1: scala.Any): scala.Boolean = { /* compiled code */ }
}
object Person extends scala.runtime.AbstractFunction2[scala.Predef.String, scala.Int, Person] with scala.Serializable {
  def this() = { /* compiled code */ }
  final override def toString(): java.lang.String = { /* compiled code */ }
  def apply(name: scala.Predef.String, age: scala.Int): Person = { /* compiled code */ }
  def unapply(x$0: Person): scala.Option[scala.Tuple2[scala.Predef.String, scala.Int]] = { /* compiled code */ }
  private def readResolve(): java.lang.Object = { /* compiled code */ }
}
```

**javap -p Person**

```
Compiled from "Person.scala"
public class Person implements scala.Product,scala.Serializable {
  private final java.lang.String name;
  private final int age;
  public static scala.Option<scala.Tuple2<java.lang.String, java.lang.Object>> unapply(Person);
  public static Person apply(java.lang.String, int);
  public static scala.Function1<scala.Tuple2<java.lang.String, java.lang.Object>, Person> tupled();
  public static scala.Function1<java.lang.String, scala.Function1<java.lang.Object, Person>> curried();
  public java.lang.String name();
  public int age();
  public Person copy(java.lang.String, int);
  public java.lang.String copy$default$1();
  public int copy$default$2();
  public java.lang.String productPrefix();
  public int productArity();
  public java.lang.Object productElement(int);
  public scala.collection.Iterator<java.lang.Object> productIterator();
  public boolean canEqual(java.lang.Object);
  public int hashCode();
  public java.lang.String toString();
  public boolean equals(java.lang.Object);
  public Person(java.lang.String, int);
}
```

**javap -p Person$**

```
Compiled from "Person.scala"
public final class Person$ extends scala.runtime.AbstractFunction2<java.lang.String, java.lang.Object, Person> implements scala.Serializable {
  public static final Person$ MODULE$;
  public static {};
  public final java.lang.String toString();
  public Person apply(java.lang.String, int);
  public scala.Option<scala.Tuple2<java.lang.String, java.lang.Object>> unapply(Person);
  private java.lang.Object readResolve();
  public java.lang.Object apply(java.lang.Object, java.lang.Object);
  private Person$();
}
```

需注意的重點：

* 由 `scalap` 及 `javap` 來看，在宣告 `case class` 後，會產生兩個 class。
* 當我們在產生 case class 時，是呼叫 `object` (singeton) 的 `apply` function.
* case class contructor 的參數，會自動變成 **read only** 的 member data.

```scala
case class Person(name: String, age: Int)

val p1 = Person("abc", 10)
p1: Person = Person(abc,10)
```

與 Pattern Match 有直接關係的 function: `apply` and `unapply`. 以 `Person` 為例：

```scala
def apply(name: scala.Predef.String, age: scala.Int): Person = { /* compiled code */ }

def unapply(x$0: Person): scala.Option[scala.Tuple2[scala.Predef.String, scala.Int]] = { /* compiled code */ }
```

## Pattern Match

上例 `Person` 的 Pattern Match 範例：

```scala
p1 match {
  case Person(n, a) => println(n, a)
  case _ => println("not match")
}
```

### Extractor

一個 class or object 有以下之一的 function 時，就可以稱作 **Extractor**。

* unapply
* unapplySeq

這類的 function ，稱為 __extraction__；反之，`apply` 則稱為 __injection__。

Extractor 只要有實作 `unapply` or `unapplySeq` 即可；但如果 Extractor 沒有實作 `apply`, 則 `unapply` 回傳型別必須是 `Boolean`。

`unapplySeq` 是用在 __variable argument__ 也就是類似 `func(lst: String*)`。

Extractor 可以是 `object` or `class`。`class` 可以存當時的條件，但 `object` 則沒有這樣的效果 (因為 object 是 singleton，無法存每次不同的比對條件)

### Pattern, Extractor, and Binding

#### Extractor only with extraction and binding

```scala
package com.example

object EMail {

  /* Injection */
  def apply(u: String, d: String) = s"${u}@${d}"

  /* Extraction */
  def unapply(s: String): Option[(String, String)] = {
    println("EMail.unapply")
    var parts = s.split("@")
    if (parts.length == 2) Some(parts(0), parts(1)) else None
  }
}

/* Extraction */
object UpperCase {
  def unapply(s: String): Boolean = {
    println("UpperCase.unapply")
    s == s.toUpperCase()
  }
}

object PatternTest {

  def main(args: Array[String]) {
    "Test@test.com" match {
      case EMail(user @ UpperCase(), domain) => println(user, domain) /* 注意：UpperCase 後面一定要加 () (括號) */
      case _ => println("not match")
    }

    "TEST@test.com" match {
      case EMail(user @ UpperCase(), domain) => println(user, domain)
      case _ => println("not match")
    }
  }

}
```

執行結果：

```
EMail.unapply
UpperCase.unapply
not match
EMail.unapply
UpperCase.unapply
(TEST,test.com)
```

當在執行 pattern match 至  `case EMail`  時，會去呼叫 `EMail.unapply(s: String)` 看是否符合；當符合時，再呼叫 `UpperCase.unapply(s: String)`。

`Test@test.com` 結果是 `not match`, 因為在 `UpperCase` 是 `false`. `TEST@test.com` 則是 `(TEST, test.com)`
	
截自：[Programming in Scala: A Comprehensive Step-by-Step Guide, 2nd Edition](http://www.amazon.com/Programming-Scala-Comprehensive-Step-Step/dp/0981531644)

#### Extractor with variable arguement

```scala
/* Extraction Only*/
class Between(val min: Int, val max: Int) {

  def unapplySeq(value: Int): Option[List[Int]] =
    if (min <= value && value <= max) Some(List(min, value, max))
    else None
}
	
object PatternTest {

  def main(args: Array[String]) {
     val between5and15 = new Between(5, 15)

    10 match {
      case between5and15(min, value, max) => println(value)
      case _ => println("not match")
    }

    20 match {
      case between5and15(min, value, max) => println(value)
      case _ => println("not match")
    }
  }
}
```

執行結果：

```
10
not match
```

因為 `Between` 的 `unapplySeq` 回傳是 `List(min, value, max)`，所以比對的 pattern 就必須是 List 的 pattern，像 `(min, value, max)` or `(_, value, max)` or `(min, _*)`
	
#### Extractor with binding

```scala
class Between(val min: Int, val max: Int) {

  def unapplySeq(value: Int): Option[List[Int]] =
    if (min <= value && value <= max) Some(List(min, value, max))
    else None
}

object PatternTest {

  def main(args: Array[String]) {
  
    (50, 10) match {
      case (n @ between5and15(_*), _) => println("first match " + n)
      case (_, m @ between5and15(_*)) => println("second match " + m)
      case _ => println("not match")
    }
  }

}
```

執行結果：

```
second match 10
```

Extractor 用在 binding 時，要注意要附上比對的 pattern (ex: `between5and15(_*)`)，如果沒寫對，會比對失敗。比如說：把 `(_, m @ between5and15(_*))` 改成 `case (_, m @ between5and15())`, 雖然 m (m = 10) 在 5 ~ 15，但會比對失敗。
	
	
## Pattern and Regex

Scala 的 Regex 有實作 `unapplySeq`, Regex 搭配 Pattern 非常好用。

```scala
object RegexTest {

  def main(args: Array[String]) {
    val digits = """(\d+)-(\d+)""".r

    "123-456" match {
      case digits(a, b) => println(a, b)
      case _ => println("not match")
    }

    "123456" match {
      case digits(a, b) => println(a, b)
      case _ => println("not match")
    }

    "abc-456" match {
      case digits(a, b) => println(a, b)
      case _ => println("not match")
    }
  }
}
```

執行結果：

```
(123,456)
not match
not match
```

因為 `digits` 有用到 `group`，所以 pattern 會是 `digits(a, b)`。如果把 `val digits = """(\d+)-(\d+)""".r` 改成 `val digits = """\d+-\d+""".r` 不使用 group 時，因為比對的 pattern 改變 (`digits(a, b)` -> `digits()`)，所以上面的三個比對都會是 `not match`。需要將程式改成如下，才會正確

```scala
val digits = """\d+-\d+""".r
  
"123-456" match {
  case digits() => println("ok")
  case _ => println("not match")
}
  
```

所以使用 `Regex` 時，儘量用 `group` 的功能，在系統設計時，彈性會比較大。


### Regex and Binding

```scala
val digits = """(\d+)-(\d+)-(\d+)""".r

("123-abc-789", "123-456-789") match {
  case (_ @ digits(a, _*), _) => println(a)
  case (_, _ @ digits(a, b, c)) => println(a, b, c)
  case _ => println("not match")
}
```

用 Binding 時，一樣要注意比對的 pattern，如： `digits(a, _*)`, `digits(a, b, c)`

## Case Class, Patch Match and Algebraic Data Type

```scala
sealed trait Tree

object Empty extends Tree
case class Leaf(value: Int) extends Tree
case class Node(left: Tree, right: Tree) extends Tree

object TreeTest {

 val depth: PartialFunction[Tree, Int] = {
    case Empty => 0
    case Leaf(value) => 1
    case Node(l, r) => 1 + Seq(depth(l), depth(r)).max
  }
  
  def max(tree: Tree): Int = tree match {
    case Empty => Int.MinValue
    case Leaf(value) => value
     case Node(l, r) => Seq(max(r), max(l)).max
  }
  
  def main(args: Array[String]) {
    
    val tree = Node(
          Node(
            Leaf(1),
            Node(Leaf(3), Leaf(4))
          ),
          Node(
              Node(Leaf(100), Node(Leaf(6), Leaf(7))),
              Leaf(2)
          )
        )
        
        
     println(depth(tree))
     
     println(max(tree))
  }
}
```

注意：使用 `sealed` 時，子類別都要與父類別放在同一個原始碼中，且如果在 pattern match 少比對一種子類別時，會出現**警告**。


範例修改自：

* [Twitter Effective Scala: Functional programming - Case classes as algebraic data types](http://twitter.github.io/effectivescala/#Functional programming-Case classes as algebraic data types)

* [Wiki: Algebraic data type](https://en.wikipedia.org/wiki/Algebraic_data_type)

進階：

[Wiki: Algebric Data Type](https://en.wikipedia.org/wiki/Algebraic_data_type)
