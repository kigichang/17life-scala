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

**Java**

```java
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

### Declaration Package
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

Hello.scala must be in `src/main/scala/com/example`

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

### Return Value of Assignment
Unlike Java/C++ returning assigned value, Scala assignment return **Unit**.

**In Java**

```java
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

### OO Access Level
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

### No break, continue

**NO** `break` and `continue` in Scala control flow.

### Return in Scala

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


### 單一檔案，不限制只能有一個 public class
Java 的程式碼檔案(.java)，只能有一個 public class，且檔名須與 public class 同名；Scala 則無此限制。

```
/* Test.scala */

class Test1
class Test2

```

### Java Primitive type 對應
Java 有 primitive type (ex: int, long 等)，但 Scala 都轉成物件 (AnyVal)來處理。ex: scala.Int, scala.Long。但在 compiler 時，依視程式的邏輯，再看是否要轉回 java primitive type。

### 不要再使用 null，改用 Option
在 Scala ，雖然 `null` 依然存在，但強烈建議不要再使用。如果原在 Java 的邏輯中，需要回傳 null 者，請都改用 `Option` 回傳。

### Checked Exception 不再強制要 try - catch
Java 在 checked exception (ex: IOException, SQLException) 都會強制要用 try - catch。在 Scala 則無。所以在寫程式時，要注意使用的函式，是否會 throw exception。Scala 有提供 Exception handle 的方式，其中一種如下(`Try`)：

```
/* Java */
public Integer parseInt(String str) {
  try {
    return Integer.parseInt(str);
  }
  catch(Exception ex) {
    return null;
  }
}

/* Scala */
def parseInt(str: String) = Try {
  str.toInt
}

```

### `equals` 可以回到改用 `==`
原本 Java 的 `==` 是用在比對 reference 值，要比對內容值要用 `equals`。在 Scala 可以直接用 `==`，要比對 reference 值，改用 `eq`。

```
/* Java */
String str = "hello world!";
"hello".equals(str);

/* Scala */
val str = "Hello world!"
"hello" == str

```

### 沒有 `++`, `--`
Scala 拿掉 `++`, `--`。請改用 `+=`, `-=`。
