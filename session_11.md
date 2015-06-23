# Scala Variance & Bounds

## Varince

Variance 主要是在討論

If \\(T_1\\) is subclass of \\(T\\), is Container[\\(T_1\\)] is subclass of Container[\\(T\\)]


|   Variance    | Meaning         | Scala notation |
|:------------- |:----------------|:---------------|
| covariant     | C[\\(T_1\\)] is subclass of C[\\(T\\)]| [+T] |
| contravariant | C[\\(T\\)] is subclass of C[\\(T_1\\)]| [-T] |
| invariant     | C[\\(T_1\\)] and C[\\(T\\)] are not related | [T] |


舉例：**Covariant**

```
scala> class Covariant[+A]
defined class Covariant

scala> val cv: Covariant[AnyRef] = new Covariant[String]
cv: Covariant[AnyRef] = Covariant@1fc2b765

scala> val cv: Covariant[String] = new Covariant[AnyRef]
<console>:8: error: type mismatch;
 found   : Covariant[AnyRef]
 required: Covariant[String]
       val cv: Covariant[String] = new Covariant[AnyRef]
```

舉例：**Contravariant**

```
scala> class Contravariant[-A]
defined class Contravariant

scala> val cv: Contravariant[String] = new Contravariant[AnyRef]
cv: Contravariant[String] = Contravariant@7bc1a03d

scala> val cv: Contravariant[AnyRef] = new Contravariant[String]
<console>:8: error: type mismatch;
 found   : Contravariant[String]
 required: Contravariant[AnyRef]
       val cv: Contravariant[AnyRef] = new Contravariant[String]
```

範例來自：[Twitter's Scala School - Type & polymorphism basics](https://twitter.github.io/scala_school/type-basics.html)


記憶方法：

\\[
\forall T_1 \in +T, \text{then }T_1 \text{ is subclass of } T
\\]

\\[
\forall T_1 \in -T, \text{then }T_1 \text{ is superclass of } T
\\]


\\[
\begin{equation}
-T \\
\uparrow\\
T \\
\uparrow \\
+T
\end{equation}
\\]

ContraVariant 案例：`Function1[-T, +R]`

eg:

```
scala> class Test[+A] { def test(a: A): String = a.toString }
<console>:7: error: covariant type A occurs in contravariant position in type A of value a
       class Test[+A] { def test(a: A): String = a.toString }
```

fix: 

```
scala> class Test[+A] { def test[B >: A](b: B): String = b.toString }
defined class Test
```

## Bounds

### Lower Type Bound

`A >: B` => `A` is superclass of `B`. `B` is the lower bound.

### Upper Type Bound

`A <: B` => A is subclass of `B`. `B` is the upper bound.

參考：

* [Twitter's Scala School - Type & polymorphism basics](https://twitter.github.io/scala_school/type-basics.html)