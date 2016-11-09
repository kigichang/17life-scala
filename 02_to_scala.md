# 由 Java/C++ 跨入 Scala 要注意的事項

## Semicolon
Semicolon `;` is not needed at end of line. Semicolon is still needed between two statement in the same line.

**Java**

```java
int a = 10;
int b = 20;

a = 20; b = 30;
```

**Scala**

```scala
var a = 10
var b = 20

a = 20; b = 30
```

## Omit Data Type
Data type can be omitted in declaration with default value. If the default value is
* Integer, the type is `Int`
* Floating-Point, the type is `Double`

**C++**

```c++
int a = 10;
double b = 1.0;
```

**In Scala**

```scala
val a = 10    // omit data type, and a is Int
val b = 1.0   // omit data type, and b is Double
```

## Package
Scala package management is the same as Java, like c++ `namespace`. Package wildcard character in Scala is `_` (in Java is `*`).

### Package Naming Conventions
| Domain Name                 | Package Name Prefix         |
|:--------------------------- |:--------------------------- |
| hyphenated-name.example.org | org.example.hyphenated_name |
| example.int                 | int_.example                |
| 123name.example.com         | com.example._123name        |

### Packages in Physical Folders
Create folders `src/main/scala/com/example/` for package `com.example`.

### Package Declaration
Use keyword `pacakage` to declare package. Package declaration must be the **First** line in a source file, and put source file in corresponding folder.

eg: Hello.scala

```scala
package com.example

object Hello {
  def main(args: Array[String]): Unit = {
    println("Hello, world!")
  }
}
```

Declare `Hello` is member of package `com.example`.  
Hello.scala must be in `src/main/scala/com/example`.

### Import Packages
* use keyword `import` (the same as Java) eg:

  ```scala
  import scala.collection.mutable
  import scala.collection.immutable
  ```

* Import classes in the same package

  ```scala
  /* in Java
   * import java.nio.file.Files
   * import java.nio.file.Paths
   */
  import java.nio.file.{ Files, Paths }
  ```

* use wildcard card

  ```scala
  /* in Java: import java.io.* */
  import java.io._

  /* in Java Static import */
  import org.apache.commons.lang3.StringUtils._
  ```

* import package with alias name

  ```scala
  import org.apache.commons.lang3.{ StringUtils => su }

  su.split(str, ",")
  ```

## Return Value of Assignment
Unlike Java/C++ returning assigned value, Scala assignment return **Unit**.

**In C++**

```c++
int a = 10;
int b = a = 30;
```
the value of b is **30**.

**In Scala**

```scala
var a = 10
val b = a = 30
```
the value of b is **Unit**.

## OO Access Level
* Java
	* Default (no explicit modifier) is **package-private**。
	* **protected** member can be accessed within its own package.

The following table shows the access to members permitted by each modifier.

Modifier | Class | Package | Subclass | World
:-:|:-:|:-:|:-:|:-:
public | Y | Y | Y | Y
protected | Y | Y | Y | N
no modifier <br /> (__package-private__) | Y | Y | N | N
private | Y | N | N | N

![example](http://docs.oracle.com/javase/tutorial/figures/java/classes-access.gif)

Modifier | Alpha | Beta | AlphaSub | Gamma
:------: | :---: | :--: | :------: | :---:
public | Y | Y | Y | Y
protected | Y | Y | Y | N
no modifier <br /> (__package-private__) | Y | Y | N | N
private | Y | N | N | N

from: [Controlling Access to Members of a Class](http://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html)

* Scala
  * **No public** modifier
  * Default (no explicit modifier) is **public**.
  * Access **protected** member in same package (like Java) is **Not Allowed**

Modifier | Alpha | Beta | AlphaSub | Gamma
:------: | :---: | :--: | :------: | :---:
no modifier <br /> (__public__) | Y | Y | Y | Y
protected | Y | N | Y | N
private | Y | N | N | N

**Summary**

Modifier | Scala | Java
:------: | :---: | :--:
no modifier | public | package-private
protected | self, subclass | self, subclass, same package

## No break, continue

**NO** `break` and `continue` in Scala control flow.

## No Increment(`++`) and Decrement(`--`)
Use compound assignment operators such as `+=` and `-=` instead of `++` and `--`.
## Return in Scala

The return value is decided by last statement in function block. eg:

```scala
def test(): Int = {
  val a = 10
  var b = a + 20

  b // return value of b
}
```

Early exit is **NOT** suggested in Scala. eg:

```scala
def greet(person: Map[String, String]): Unit = {
  if (!person.get("name").isDefined) return // Early Return, Not Suggested

  println(s"Hello ${person("name")}")

  val location = if (person.get("location").isDefined) s"I hope the weather is nice in ${person("location")}" else "I hope the weather is nice near you."

  println(location)
}

greet(Map("name" -> "John"))
greet(Map("name" -> "Jane", "location" -> "Cupertino"))
```

Modify `greet` function

```scala
def greet(person: Map[String, String]): Unit = {
  for (name <- person.get("name")) {
    println(s"Hello ${name}")

    person.get("location") match {
      case Some(location) => println(s"I hope the weather is nice in ${location}")
      case None => println("I hope the weather is nice near you")
    }
  }
}

greet(Map("name" -> "John"))
greet(Map("name" -> "Jane", "location" -> "Cupertino"))
```

## Multiple Public Class in Same Source File
In Java, there is **ONLY ONE** public class in source file, and source file name must be same as public class.

It is unlimited in Scala. eg: in `Test.scala`, it can have two or more public class.

```scala
class Test1
class Test2
```

## Java Primitive Type and Scala AnyVal
All value type in Scala inherit from `AnyVal`. Scala convert all primitive type of Java such as `int` and `double`.  
According to program logistic, `AnyVal` may be converted to Java primitive type in compilation.

## Use Option Instead of Null
Null is an special type in Java, and remains in Scala.  
Strongly recommended to use `Option` instead of null.

eg:

**In Java**

```java
public static Integer parseInt(String str) {
    try {
        return Integer.parseInt(str);
    }
    catch(Exception ex) {
        return null;
    }
}
```

**In Scala**

```scala
def parseInt(str: String) = try {
  Option(str.toInt)
} catch {
  case ex: Throwable => None
}
```


## Catch Checked Exception is NOT necessary
It is necessary to catch checked exception in Java, but not in Scala.  
Be careful with function throwing exception in Java library.  
Scala provide `Try` to handle exception. eg:

**In Scala**

```scala
def parseInt(str: String) = Try { str.toInt } toOption
```

## Object Comparison

  Language | Compare Reference | Compare Object Value
 --------: | :-----: | :----:
 Java | == | equals
 Scala| eq | ==

eg:

**In Java**

```java
String str = "hello world!";
"hello".equals(str);
```

**In Scala**

```scala
val str = "Hello world!"
"hello" == str
```
