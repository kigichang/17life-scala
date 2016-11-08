# 開發環境設定
* JDK 8: [JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
* Activator: [mini activator 1.3.12](https://downloads.typesafe.com/typesafe-activator/1.3.12/typesafe-activator-1.3.12-minimal.zip)
* scala: use [2.11.8](http://www.scala-lang.org/download/2.11.8.html)

# Activator

Activator 在開發 Scala 會用到的專案管理工具。類似 Java 的 Ant, Maven, Gradle, Ivy 等工具。主要的功能有：

* 函式庫相依性管理
* Compile
* Test
* Package
* REPL (Read, Eval, Print, Loop)

## Generate Empty Project

指令：

`activator new PROJECT_NAME` ex: `activator new test_scala`

```
activator new test_scala

Fetching the latest list of templates...

Browse the list of templates: http://lightbend.com/activator/templates
Choose from these featured templates or enter a template name:
  1) minimal-akka-java-seed
  2) minimal-akka-scala-seed
  3) minimal-java
  4) minimal-scala
  5) play-java
  6) play-scala
(hit tab to see a list of all templates)
>
```

選擇專案的 template. Template 說明：

* minimal-akka-java-seed: Akka 專案 template。使用 Java 開發
* minimal-akka-scala-seed: Akka 專案 template。使用 Scala 開發
* minimal-java: 基本 Java 專案。
* minimal-scala: 基本 Scala 專案。
* play-java: Play 專案，開發網站。使用 Java 開發。
* play-scala: Play 專案，開發網站。使用 Scala 開發。

## Folders and Files in General Project

```
.
├── LICENSE
├── bin
│   ├── activator
│   └── activator.bat
├── build.sbt
├── libexec
│   └── activator-launch-1.3.10.jar
├── project
│   └── build.properties
└── src
    ├── main
    │   └── scala
    │       └── com
    │           └── example
    │               └── Hello.scala
    └── test
        └── scala
            └── HelloSpec.scala
```

說明：

* bin/activator: activator 執行檔
* build.sbt: SBT 專案管理檔，類似 maven 的 pom.xml
* libexec/activator-launch-x.x.x.jar: activator 的 JAR 檔
* project
	* build.properties
	* plugins.sbt: 如果有使用其他工具時，要寫在這。eg: Play 的 plugins
* src: 程式碼放的位置
	* main: 專案的程式碼
		* scala: Scala 的程式碼
		* java: 程式碼
		* resources: 設定檔
	* test: 專案 Unit Test 程式的位置
		* scala: Scala 程式碼
		* java: Java 程式碼
		* resources: Unit Test 設定檔，只有在 Unit Test 會用到

## Build Project

先進到目錄中，執行 `activator`，會進到互動的畫面。

* `compile`: 編譯
* `test`: Unit Test
* `clean`: 清除已編譯好的檔案 (\*.class)
* `reload`: 重新載入 build.sbt 設定。如果有修改過 build.sbt 就會需要用這個指令。
* `update`: 更新相依的函式庫檔案
* `run`: 執行開發的程式。也就是執行 `main` 程式。
* `package`: 專案打包成 JAR (\*.jar) 檔
* `console`: 進入 Scala 的 REPL (read-eval-print loop) 模式。Java 9 會支援 REPL 模式，會套用 build.sbt 設定的環境。
* `doc`: 產生 api 說明文件
* `exit`: 離開互動模式

### Build life Cycle
* `test`: `compile` -> `test`
* `run`: `compile` -> `run`
* `package`: `compile` -> `package`
* `console`: `compile` -> `console`

## Package Dependency 管理

編輯 **build.sbt** 中的 `libraryDependencies`。eg:

```
libraryDependencies += "com.typesafe.play.extras" %% "iteratees-extras" % "1.5.0"

libraryDependencies ++= Seq(
  jdbc,
  cache,
  ws,
  "org.scalatestplus.play" %% "scalatestplus-play" % "1.5.1" % Test
)
```

### Dependency Scope

* compile
* required
* test



### Scala Cross Version

## 開發工具
* [Jet Brains IntelliJ Idea](https://www.jetbrains.com/idea/) \*
* [Eclipse with Scala Plugins](http://scala-ide.org/download/current.html)
* [ScalaIDE](http://scala-ide.org/)

## Hello World
* 產一個空白的專案 `activator new hello_world minimal-scala`
* 進入 `hello_world` 目錄，執行 `activator` 進入互動模式
  * `cd hello_world`
  * `activator`
* 編譯 `compile`
* 執行 `run`
* 打包 `package`，會產出 `hello_world/target/scala-2.11/hello_world_2.11-1.0.jar`
* 執行 JAR 檔 `scala -cp target/scala-2.11/hello_world_2.11-1.0.jar com.example.Hello`
