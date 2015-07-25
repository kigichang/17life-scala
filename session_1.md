# 由 Java 跨入 Scala 要注意的事項

## Java 開發者需注意事項

### 分號 `;`
Java 一定要使用 `;` 隔開 statement；在 Scala 中，除非同一行有兩個以上的 statement，才需要用 `;` 隔開，否則可**省略**。

**In Java**

```
int a = 10;
int b = 20;

a = 20; b = 30;

```

**In Scala**

```
var a = 10
var b = 20

a = 20; b = 30

```

### 宣告時，型別可以省略
在宣告變數了，可以省略型別；雖然不用寫型別，但 Scala 依然是 **Strong-Type**，變數的型別一旦決定後，就不能更改，這一點和 Java 是一樣的。

**In Java**

```
public class Test { }

Test a = new Test();

```

**In Scala**

```
/* Scala */
class Test

val a = new Test()

```

### Import Package 宣告
Scala 在 import package 有更彈性的宣告方式：

**萬用字元**

```
/* 等同 Java: import java.io.* */
import java.io._ 

/* 等同 Java Static import */
import org.apache.commons.lang3.StringUtils._ 

```

**相同 Package 下的 Class 整理成一行**

```
/* 等同 Java
 * import java.nio.file.Files
 * import java.nio.file.Paths
 */
import java.nio.file.{ Files, Paths }

```

**常用或太長的 Class Name 取別名**

```
/* Alias Name */
import org.apache.commons.lang3.{ StringUtils => su }

su.split(str, ",")

```

### Ruturn Value of Assignment

在 Scala 指定值後，會回傳 __Unit__，並不會像 Java 回傳指定的值。

**In Java**

```
int a = 10;
int b = a = 30;

```

**b 的值是 30。**

**In Scala**

```
var a = 10
val b = a = 30

```

**b 的值是 Unit。**

### OO Access Level
* Java
	* 在 Java 沒有指定時，是 __package-private__。
	* 可存取同 __package__ 的 __protected__ 變數。

[Java 說明文件](http://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html)

![實例](http://docs.oracle.com/javase/tutorial/figures/java/classes-access.gif "實例")

Modifier | Class | Package | Subclass | World
:------: | :---: | :-----: | :------: | :---:
public | Y | Y | Y | Y 
protected | Y | Y | Y | N
no modifier <br /> (__package-private__) | Y | Y | N | N
private | Y | N | N | N

Modifier | Alpha | Beta | AlphaSub | Gamma
:------: | :---: | :--: | :------: | :---:
public | Y | Y | Y | Y
protected | Y | Y | Y | N
no modifier <br /> (__package-private__) | Y | Y | N | N
private | Y | N | N | N

* Scala
	* Scala 拿掉 __public__ modifier
	* 在 Scala 沒有指定時，則是 __public__。
	* __不能__存取同 __package__ 的 __protected__ 變數。

Modifier | Alpha | Beta | AlphaSub | Gamma
:------: | :---: | :--: | :------: | :---:
no modifier <br /> (__public__) | Y | Y | Y | Y
protected | Y | N | Y | N
private | Y | N | N | N

總結

Modifier | Scala | Java
:------: | :---: | :--:
no modifier | public | package-private
protected | 自己, subclass | 自己, subclass, 同 package

### 沒有 break 及 continue 關鍵字

Scala 拿掉了 break 及 continue 功能，雖然有替代方案，不建議使用。

### 保有 return 功能，但不建議使用
Scala 為了遵守 Functional Lanaguage 精神，不建議在程式隨時可以 return 來中斷執行。雖然 `return` 還可以使用，但不建議使用。

如果需要回傳值，請在函式的最後一行，指定回傳值。 Ex:

```
def test(): Int = {
  val a = 10
  var b = a + 20

  b /* 回傳 b 的值 */
}
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