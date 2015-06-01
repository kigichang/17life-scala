# Scala Collection

## Array

Scala 有 `Array` 的 Class，語法上與 Java 差異很大。
eg:

Java String Array: `String[] test = Strig[5];`

Scala String Array: `val test = Array.fill(5) { "" }`

Q: Array 定義？！

Scala `Array` 是一組 mutable indexed 集合。

宣告:

```
scala> val arr1 = Array(1, 2, 3)
arr1: Array[Int] = Array(1, 2, 3)

scala> val arr2 = Array("a", "b", "c")
arr2: Array[String] = Array(a, b, c)

scala> val arr3 = Array.fill(3) { math.random }
arr3: Array[Double] = Array(0.6325626123459638, 0.7135533929240558, 0.6461315533714135)

scala> class Test
defined class Test

scala> val arrTest = Array.fill[Test](3) { null }
arrTest: Array[Test] = Array(null, null, null)
```

取值：

```
arr1(1)
``` 


## Immutable v.s. Mutable

Scala Collection Class 分成 **Immutable** 及 **Mutable**。兩者主要差別是 **Mutale** 可以修改集合裏面的內容；但 **Immutable** 不行。使用類似 `append` 時，`Immutable` 的集合，會回傳一個新的集合，而非就現有的集合擴充，在使用時，要特別小心，以免造成記憶體浪費。

p.s. 在 Java 最常用的 `immutable` 是 `String`


### Scala Collection Hierarchy

![Scala Collection](http://www.scala-lang.org/docu/files/collections-api/collections.png)


### Scala Immutable Collection

![Scala Immutable Collection](http://www.scala-lang.org/docu/files/collections-api/collections.immutable.png)

### Scala Mutable Collection

![Scala Mutable Collection](http://www.scala-lang.org/docu/files/collections-api/collections.mutable.png)


### Twitter Coding Style

完整指出使用 `mutable`。**不要使用** `import scala.collection.mutable._` 

```
import scala.collection.mutable

val set = mutable.Set()

```

## Compare with Java and Conversions

Scala 與 Java Collection 互轉的原則：

```
Iterator               <=>     java.util.Iterator
Iterator               <=>     java.util.Enumeration
Iterable               <=>     java.lang.Iterable
Iterable               <=>     java.util.Collection
mutable.Buffer         <=>     java.util.List
mutable.Set            <=>     java.util.Set
mutable.Map            <=>     java.util.Map
mutable.ConcurrentMap  <=>     java.util.concurrent.ConcurrentMap

```

`import collection.JavaConversions._`

