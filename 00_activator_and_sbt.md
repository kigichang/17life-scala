# Activator and SBT

## 開發環境設定

- JDK 8: [JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- Activator: [mini activator 1.3.12](https://downloads.typesafe.com/typesafe-activator/1.3.12/typesafe-activator-1.3.12-minimal.zip)
- scala: use [2.11.8](http://www.scala-lang.org/download/2.11.8.html)

## Scala REPL (Read Eval Print Loop)

Scala 有提供 REPL 環境，對於要測試簡單的程式作法或想法，非常方便，不用為了測試小程式就要開一個空白的專案。也可以利用這項功能，撰寫類似 script 的程式，處理日常的小工作。

- 進入 REPL 環境：執行 `Scala_Intall_Path\bin\scala` 即可進入 REPL 環境
- 離開 REPL 環境：輸入 `:quit` 即可離開。
- 提示：按 `tab` 鍵
- 清除畫面：按 `cmd` + `k`

REPL 的環境對於初學者來說，是一個很好的學習的工具，可以很快速了解一個程式語言。

註：Java 9 也會有 REPL 的功能。

## Activator (Deprecated)

Activator 在開發 Scala 會用到的專案管理工具。類似 Java 的 Ant, Maven, Gradle, Ivy 等工具。主要的功能有：

- 函式庫相依性管理
- Compile
- Test
- Package
- REPL (Read, Eval, Print, Loop)

### Generate Empty Project

指令：

`activator new PROJECT_NAME` eg: `activator new test_scala`

```text
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

- minimal-akka-java-seed: Akka 專案 template。使用 Java 開發
- minimal-akka-scala-seed: Akka 專案 template。使用 Scala 開發
- minimal-java: 基本 Java 專案。
- minimal-scala: 基本 Scala 專案。
- play-java: Play 專案，開發網站。使用 Java 開發。
- play-scala: Play 專案，開發網站。使用 Scala 開發。

### Folders and Files in General Project

```text
.
├── LICENSE
├── bin
│   ├── activator
│   └── activator.bat
├── build.sbt
├── libexec
│   └── activator-launch-1.3.10.jar
├── project
│   └── build.properties
└── src
    ├── main
    │   └── scala
    │       └── com
    │           └── example
    │               └── Hello.scala
    └── test
        └── scala
            └── HelloSpec.scala
```

說明：

- bin/activator: activator 執行檔
- build.sbt: SBT 專案管理檔，類似 maven 的 pom.xml
- libexec/activator-launch-x.x.x.jar: activator 的 JAR 檔
- project
  - build.properties
  - plugins.sbt: 如果有使用其他工具時，要寫在這。eg: Play 的 plugins
- src: 程式碼放的位置
  - main: 專案的程式碼
    - scala: Scala 的程式碼
    - java: 程式碼
    - resources: 設定檔
  - test: 專案 Unit Test 程式的位置
    - scala: Scala 程式碼
    - java: Java 程式碼
    - resources: Unit Test 設定檔，只有在 Unit Test 會用到

### Build Project

先進到目錄中，執行 `activator`，會進到互動的畫面。

- `compile`: 編譯
- `test`: Unit Test
- `clean`: 清除已編譯好的檔案 (\-.class)
- `reload`: 重新載入 build.sbt 設定。如果有修改過 build.sbt 就會需要用這個指令。
- `update`: 更新相依的函式庫檔案
- `run`: 執行開發的程式。也就是執行 `main` 程式。
- `package`: 專案打包成 JAR (\-.jar) 檔
- `console`: 進入 Scala 的 REPL (read-eval-print loop) 模式。Java 9 會支援 REPL 模式，會套用 build.sbt 設定的環境。
- `doc`: 產生 api 說明文件
- `exit`: 離開互動模式

#### Build life Cycle

- `test`: `compile` -> `test`
- `run`: `compile` -> `run`
- `package`: `compile` -> `package`
- `console`: `compile` -> `console`

### Package Dependency 管理

編輯 --build.sbt-- 中的 `libraryDependencies`。eg:

```scala
libraryDependencies += "com.typesafe.play.extras" %% "iteratees-extras" % "1.5.0"

libraryDependencies ++= Seq(
  jdbc,
  cache,
  ws,
  "org.scalatestplus.play" %% "scalatestplus-play" % "1.5.1" % Test
)
```

#### Library Dependency

`libraryDependencies += groupID % artifactID % revision % configuration`

or

`libraryDependencies += groupID %% artifactID % revision % configuration`

#### Dependency Scope

- Provided: 編譯時期也用到。打包程式時，不用包進來，而是執行的環境要提供。
- Compile: 預設。在編譯時期就會使用。打包程式時，要隨著出去。
- Runtime: 編譯時期不會使用到，但執行時會使用到。
- Test: 測試時期會用。

#### Scala Cross Version

Scala 開發的套件，都會指定是用那個 Scala 版本編譯，因此在使用 Scala 的函式庫時，會在 `groupdID` 後用 `%%`，eg: `libraryDependencies += groupID %% artifactID % revision % configuration`，此時會看 `build.sbt` 的 Scala 版本設定 (`scalaVersion := "2.11.7"`)，來決定要使用那個 Scala 版本的函式庫。

## 開發工具

- [Jet Brains IntelliJ Idea](https://www.jetbrains.com/idea/) \-
- [Eclipse with Scala Plugins](http://scala-ide.org/download/current.html)
- [ScalaIDE](http://scala-ide.org/)

## Hello World

- 產一個空白的專案 `activator new hello_world minimal-scala`
- 進入 `hello_world` 目錄，執行 `activator` 進入互動模式
  - `cd hello_world`
  - `activator`
- 編譯 `compile`
- 執行 `run`
- 打包 `package`，會產出 `hello_world/target/scala-2.11/hello_world_2.11-1.0.jar`
- 執行 JAR 檔 `scala -cp target/scala-2.11/hello_world_2.11-1.0.jar com.example.Hello`
