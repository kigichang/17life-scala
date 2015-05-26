# Scala 的物件導向

## OOP

* 封裝 (Encapsulation)
* 繼承 (Inheritance)
* 多型 (Polymorphism)

## Class

### 宣告方式

```
class 名稱 [extends Class or Trait 名稱] [with Trait 名稱] {
  
}
```
eg:

```
scala> class Test
defined class Test

scala> class Test2 extends Test 
defined class Test2

```

```
class 名稱([存取等級] [val or var] 變數名稱: 型別) [extends Class or Trait 名稱] [with Trait 名稱] [with Trait 名稱] {
  def this() = this(...)
}

```

eg:

```
scala> class Test(a: Int, b: Double) {
     |   println(s"${a}, ${b}")
     | }
defined class Test

scala> class Test2(a: Int) extends Test(a, 10.0) {        
     | def this() = this(20)
     | }
defined class Test2

```
__Java Bean Like__

```
scala> class Test(var name: String, var age: Int, var address: String)

scala> val a = new Test("abc", 10, "aaaaaa")
a: Test = Test@74d1dd7e

scala> a.name
res0: String = abc

scala> a.age
res1: Int = 10

scala> a.address
res2: String = aaaaaa

scala> a.name = "abcdef"
a.name: String = abcdef

scala> a.age = 20
a.age: Int = 20

scala> a.address = "bbbbbb"
a.address: String = bbbbbb

```

### Primary and Auxiliary Contructors

在 Scala 及 Swift 中，Constructor 有主、副之分。

eg:

```
scala> class Test(a: Int, b: Double) {
     |   println(s"${a}, ${b}")
     |   
     |   def this() = this(10, 20.0)
     | }
defined class Test

```

__Primary Constructor__ 是 

```
class Test(a: Int, b: Double) { 
	println(s"${a}, ${b}")
}

```

__Auxiliary Constructor__ 是

```
	def this() = this(10, 20.0)
```

轉換成 Java 語法來看：

```
class Test {
	public Test(int a, double b) {
		System.out.println("" + a + ", " + b)
	}
	
	public Test() {
		this(10, 20.0);
	}
}

```

* `public Test(int a, double b)` 是 __Primary Constructor__
* `public Test()` 是 __Auxiliary Constructor__

### Override
當子類別要改寫父類別的函式時，在宣告函式時，一定要用 `override`，否則會報錯。 

eg:

```
scala> class Test {
     | def echo = "Echo"
     | }
defined class Test

```
__沒用 `override` 會報錯__

```
scala> class Test2 extends Test {
     | def echo = "Echo2"
     | }
<console>:9: error: overriding method echo in class Test of type => String;
 method echo needs `override' modifier
       def echo = "Echo2"
```

__正解__

```
scala> class Test2 extends Test {
     | override def echo = "Echo2"
     | }
```

### 注意事項

宣告 class 時，如果 primary constructor 的變數，沒有加 `val` or `var` 則不會自動變成 member data。

```
scala> class Test(a: Int)
defined class Test

scala> val t = new Test(10)
t: Test = Test@7c8c0c57

scala> t.a
<console>:10: error: value a is not a member of Test
              t.a
```

## Object

__Scala 的 `object` 等同於 Java Singleton__

Scala 允許 `object` 的名稱與 `class` 相同，且可以放在同一份 source code 中。這類的 `object` 稱作 __Companion Object__。

類比 Java 的實作，就是把 __`static`__ 相關的變數及函式(Class member data and function)，放到 `object` 中。

在 `object` 中，就不用 __`static`__ 這個關鍵字。

eg:

```
scala> object Test {
     | def myTest = println("abc")
     | }
defined object Test

scala> Test.myTest
abc

```

在 Java 要寫一個應用程式 (Application)，需要在一個 `public class` 內，宣告一個 `public static void main (String[] args)` 函式。

eg:

```
public class Test {
	public static void main(String[] args) {
	}
}
```

改用 Scala 後，則在一個 `object` 宣告 `def main(args: Array[String])`

eg:

```
object Test {
  def main(args: Array[String]) {
  }
}
```

## Trait

Scala 的 `trait` 可以類比成 Java 的 `interface`。一個 `class` 可以繼承(實作)多個 `trait`。在使用時，如果 `class` 沒有繼承其他的 `class` or `trait` 時，則用 `extends`，否則用 `with`

eg:

```
scala> trait MyTrait
defined trait MyTrait

scala> class Test
defined class Test

scala> class Test2 extends Test with MyTrait
defined class Test2

scala> class Test3 extends MyTrait
defined class Test3

```

在 Java 8 以前，`interface` 沒有 default function 的功能，也因此無法在 `interface` 內實作函式。

Scala 的 `trait` 則打破這個限制，允許在 `trait` 內有變數、函式的實作。也因此更起達到多重繼承的效果。

eg: Trait 內含變數與實作函式

```
scala> trait MyTrait {
     |   val a = 10
     |   def test: Int
     |   def sum(x: Int, y: Int) = x + y + a
     | }
defined trait MyTrait

scala> class Test extends MyTrait {
     |   def test = a + 1000
     | }
defined class Test

scala> val t = new Test
t: Test = Test@597b40a8

scala> t.test
res0: Int = 1010

scala> t.sum(t.a, 10)
res2: Int = 30
```

eg: 多個 Trait；有多重繼承效果

```
scala> trait MyTrait1 {
     |   val a = 10
     |   def test1: Int
     | }
defined trait MyTrait1

scala> trait MyTrait2 {
     |   val b = 20
     |   def test2: Int
     | }
defined trait MyTrait2

scala> trait MyTrait3 {
     |   val c = 30
     |   def test3: Int
     | }
defined trait MyTrait3

scala> class Test extends MyTrait1 with MyTrait2 with MyTrait3 {
     |   def test1 = b + 1
     |   def test2 = c + 2
     |   def test3 = a + 3
     | }
defined class Test

scala> val t = new Test
t: Test = Test@37f7bfb6

scala> t.test1
res0: Int = 21

scala> t.test2
res1: Int = 32

scala> t.test3
res2: Int = 13
```

object 也可以繼承(實作) trait

```
scala> trait MyTrait
defined trait MyTrait

scala> object Test extends MyTrait
defined object Test
```

## Scala 增強的功能

### 自定 Access Level

Scala 允許自定 Access Level，因此在程式設計上會更有彈性，安全性也會更高。

使用方式：`modifier[this, class or package]`. eg: `private[Test]`

eg: 變數只允許自己本身的 instance 使用。比 private 更嚴格。

```
scala> class TestPrivateThis {
     |   private[this] val a = 10
     | 
     |   def func(): Int = a + 10
     | 
     |   def func(that: TestPrivateThis): Int = a + that.a
     | }
<console>:12: error: value a is not a member of TestPrivateThis
         def func(that: TestPrivateThis): Int = a + that.a
```

### Operator Overloading

在 Java 中，無法在 class 中，定義 `+`，`-`，`*`，`/` 這類四則運算。但在 Scala 則可以，這會讓程式碼更簡潔也更方便閱讀。

eg:

```
class Rational(n: Int, d: Int) {

  require(d != 0)
  
  def this(n: Int) = this(n, 1)
  
  def gcd(a: Int, b: Int): Int = if (a == 0) b else gcd(b % a, a)
  
  val g = gcd(n, d)
  
  val numer = n / g
  
  val denom = d / g
  
  
  override def toString = if (denom != 1) s"${numer} / ${denom}" else s"${numer}"
  
  def +(that: Rational) = new Rational(numer * that.denom + denom * that.numer, denom * that.denom)
  
  def -(that: Rational) = new Rational(numer * that.denom - denom * that.numer, denom * that.denom)
  
  def *(that: Rational) = new Rational(numer * that.numer, denom * that.denom)
  
  def /(that: Rational) = new Rational(numer * that.denom, denom * that.numer)
  
}


object Rational {
  
  def main(args: Array[String]) {
   
    val r1 = new Rational(3, 7)
    val r2 = new Rational(5, 21)
    
    println(r1 * r2)
  }
}
```

### apply Function

Scala 有類似 PHP Magic Function 的功能，當在 `class` 宣告這類型的函式後，使用時，可以省略函式的名稱。

Scala 最常被用到的是 `apply`

eg:

```
scala> class Test {
     | def apply(a: Int, b: Int): Int = a + b
     | }
defined class Test

scala> val t = new Test
t: Test = Test@300a2ae

scala> val a = t(10, 20)
a: Int = 30

```

在上例中 `val a = t(10, 20)` 就是呼叫 `apply` 函式。`apply` 函式，在 scala collection 相關 class (HashMap, Array 等) 很常用到。


#### case class

Scala 在宣告 `class` 時，可以使用 `case` 這個修飾詞，使用後，在產生 instance (以後就不用 object 這個字眼，以免跟 scala 的 `object` 混餚) 時，可以省略 `new`；如果 primary contructor 有參數時，會自動將參數轉成 __val__ 型態的 member data。

eg:

```
scala> case class Test(name: String, age: Int)
defined class Test

scala> val t = Test("abc", 20)
t: Test = Test(abc,20)

scala> t.name 
res0: String = abc

scala> t.age
res1: Int = 20

scala> t.name = "abcdef"
<console>:10: error: reassignment to val
       t.name = "abcdef"
```

`case class` 與 `apply`, `object`, pattern match (`match` - `case`) 有密不可分的關係，在第二階段會再詳述。


## Enumeration

### 基本用法

#### Java 版本

```
public enum DirectionJava {
	TOP, DOWN, LEFT, RIGHT;
}
```

在 Java 使用 `enum` 宣告即可。 

Java class 使用 `enum` 後會變成 final class，因此 enum 不能再繼承其他的 class (Java 是單一繼承)，也不能再被其他 class 繼承。

`enum` 的 Constructor 設計成 `private`，所以之後也無法自行產生 instance，只能使用內定的值 (static 值) 或 null。

總結以上，各位不覺得 `enum` 就跟 signleton 很像嗎？！

#### Scala 版本

```
object DirectionScala extends Enumeration {
  val Top, Down, Left, Right = Value
}
```

Scala 的 Enumation 歷史比 Java 早，因此實作語法上，與 Java 不太相同，但觀念差不多。

Scala 的 Enumeration 跟 Java 的很類似，但 Scala 的內定值型別是 Enumeration.Value。 

Enumeration.Value 本身是個 abstract class。在 Enumeration 內有一個實作的 class: `protected class Val`，如果需要自定 Enumeration 時，就會需要繼承這個 class。

### 進階

#### Java 版

```
public enum Planet {
    MERCURY (3.303e+23, 2.4397e6),
    VENUS   (4.869e+24, 6.0518e6),
    EARTH   (5.976e+24, 6.37814e6),
    MARS    (6.421e+23, 3.3972e6),
    JUPITER (1.9e+27,   7.1492e7),
    SATURN  (5.688e+26, 6.0268e7),
    URANUS  (8.686e+25, 2.5559e7),
    NEPTUNE (1.024e+26, 2.4746e7);

    private final double mass;   // in kilograms
    private final double radius; // in meters
    
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
    }
    
    private double mass() { return mass; }
    private double radius() { return radius; }

    // universal gravitational constant  (m3 kg-1 s-2)
    public static final double G = 6.67300E-11;

    double surfaceGravity() {
        return G * mass / (radius * radius);
    }
    
    double surfaceWeight(double otherMass) {
        return otherMass * surfaceGravity();
    }
    
    public static void main(String[] args) {
        if (args.length != 1) {
            System.err.println("Usage: java Planet <earth_weight>");
            System.exit(-1);
        }
        double earthWeight = Double.parseDouble(args[0]);
        double mass = earthWeight/EARTH.surfaceGravity();
        for (Planet p : Planet.values())
           System.out.printf("Your weight on %s is %f%n",
                             p, p.surfaceWeight(mass));
    }
}
```

#### Scala 版

```
object Planets extends Enumeration {

  val G: Double = 6.67300E-11

  final case class Planet private[Planets] (mass: Double, radius: Double) extends Val {
    def surfaceGravity(): Double = G * mass / (radius * radius)
    def surfaceWeight(otherMass: Double): Double = otherMass * surfaceGravity()
  }

  val Mercury = Planet(3.303e+23, 2.4397e6)
  val Venus = Planet(4.869e+24, 6.0518e6)
  val Earth = Planet(5.976e+24, 6.37814e6)
  val Mars = Planet(6.421e+23, 3.3972e6)
  val Jupiter = Planet(1.9e+27,   7.1492e7)
  val Saturn = Planet(5.688e+26, 6.0268e7)
  val Uranus = Planet(8.686e+25, 2.5559e7)
  val Neptune = Planet(1.024e+26, 2.4746e7)


  def main(args: Array[String]) {
    require(args.length == 1, "Usage: java Planet <earth_weight>")

    val earthWeight = args(0).toDouble  
    val mass = earthWeight / Planets.Earth.surfaceGravity

    Planets.values.foreach(p => println("Your weight on %s is %f%n".format(p, p.asInstanceOf[Planet].surfaceWeight(mass))))
  }

}

```

## 附錄

### activator

#### 產生專案

完整的指令： `activator new [project-name] [template-name]`

一般的步驟：

* 執行：`activator new 專案名稱`
* 選擇專案的 template


```
Fetching the latest list of templates...

Browse the list of templates: http://typesafe.com/activator/templates
Choose from these featured templates or enter a template name:
  1) minimal-akka-java-seed
  2) minimal-akka-scala-seed
  3) minimal-java
  4) minimal-scala
  5) play-java
  6) play-scala
(hit tab to see a list of all templates)

```

1), 2) 是 akka 專案

3), 4) 是一般應用程式專案

5), 6) 是寫 Play Framework (web) 專案


* 在專案目錄下的 `project` 目錄，加入 `plugins.sbt` 檔案，內容如下：

```
addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "3.0.0")

addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.13.0")
```

__注意：兩行中間，一定要空一行__

* 執行：`activator eclipse` 產生 eclipse 可以匯入的專案格式
* 使用 eclipse 匯入 (General -> Existing Projects into Workspace)

#### 命令列模式

在專案的目錄下，執行 `activator`，會進入 Activator 的命令列模式。與 Maven 相同，有 `reload`, `update`,`clean`, `compile`, `package`, `test` 等指定；如果有加入 sbteclipse-plugin，也可在這使用 `eclipse`。

* reload: 重新載入 SBT 設定
* update: 更新相關的 dependency library
* clean: 將先前產生的 *.class, *.jar 清除
* compile: 編譯專案
* package: 打包專案
* test: 執行 Unit Test


#### 修改 Dependency

與 Maven 相同， Activator 可以管理專案的 dependency。修改專案目錄下的 `build.sbt`。

eg:

```
libraryDependencies += "mysql" % "mysql-connector-java" % "5.1.34"

libraryDependencies += "org.apache.commons" % "commons-lang3" % "3.3.2"

libraryDependencies += "org.apache.commons" % "commons-email" % "1.3.3"
```
__注意：每行中間，一定要空一行__

修改完 dependency，請再執行 `activator eclipse` 重新產生 eclipse 的專案，在 eclipse IDE 再重新 reload 一次，即可生效。

