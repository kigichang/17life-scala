# Scala Introduction

## Introduction

### Author：Martin Odersky  
![artin Odersky](https://upload.wikimedia.org/wikipedia/commons/thumb/b/b7/Mark_Odersky_photo_by_Linda_Poeng.jpg/389px-Mark_Odersky_photo_by_Linda_Poeng.jpg)

from: [wikipedia](https://en.wikipedia.org/wiki/Martin_Odersky)

### History
* 2001 start design
* 2003 internal release
* 2004 release on Java Platform and .Net Framework
* 2006 2.0 release
* 2012 drop .NET support

from: [wikipedia](https://en.wikipedia.org/wiki/Scala_(programming_language)#History)

### Attribute
Scala is  
* a JVM (Java Virtual Machine) language, Scala program runs on JVM.
* a Strong type (strict type, Static type) lanaguage
* a Pure object-oriented language, every value is an object.
* a Functional language, every function is a value.

## Principle

### Java Introduction

* How to run an Java program  
![Run a Java Program](https://docs.oracle.com/javase/tutorial/figures/getStarted/getStarted-compiler.gif)

* How to run on different platform  
![Run on different platform](https://docs.oracle.com/javase/tutorial/figures/getStarted/helloWorld.gif)

* Platform Independent  
![Independent](https://docs.oracle.com/javase/tutorial/figures/getStarted/getStarted-jvm.gif)

from: [About the Java Technology](https://docs.oracle.com/javase/tutorial/getStarted/intro/definition.html)

### Scala on JVM
* Compile Scala source code (\*.scala) to Java bytecode
* Leverage Java **Garbage Collection** to manage object life cycle (Memory)
* Can use Java library
* Category of Scala data type is the same as Java. There are two category:
  * **Value Type** such as `Int`, `Double`
  * **Reference Type** such as `List`.
* Parameter passing is **by Value** for non-function value. The function value is **by Name**.

## Entry Point
