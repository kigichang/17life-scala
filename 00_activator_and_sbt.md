# Activator

Activator 在開發 Scala 會用到的專案管理工具。類似 Java 的 Ant, Maven, Gradle, Ivy 等工具。主要的功能有：

* 函式庫相依性管理
* Compile, Build
* Package，Depolyment

## Requirement

* Java 8 SDK

下載網址：[Activator](https://www.lightbend.com/activator/download)


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

## Forlds and Files in General Project

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
* `clean`: 
* ``
* ``

