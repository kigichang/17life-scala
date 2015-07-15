# Introduction of Akka Remote

Akka 允許遠端執行 Actor, 而且內建小型的 message server，在開發時，完全不用去理會底層網路的建置。要使用 Akka Remote 功能，要特別注意每台機器要彼此看得到。也就是說在網段的安排上，每台機器要彼此可以互連，因為 Akka Remote 會在每台機器，啟一個小 server，而且會彼此互連。

Akka 遠端執行，分成兩種模式如下：

## 模式

### Lookup

Lookup 是在遠端 Server，先啟動 Actor，再讓 Client 連線使用。這種模式很類似傳統的 Client/Server 架構。


### Deployment

由 Master，指定遠端的 Node 要執行那些Actor。在佈署環境時，還是需要將 Actor 放到 Node。Node 啟動時，並不會去預載要執行的 Actor，而是等待 Master 端來指定。


## Actor 基本運作
在使用 Remote 功能前，先要了解 Actor 基本運作。

### Actor 階層關係

每一個 Actor 都有一個 Supervisor，包含系統內建及自建的 Actor。如下圖：

![The Top-Level Supervisors](http://doc.akka.io/docs/akka/2.3.11/_images/guardians.png)

__Image from: [Supervision and Monitoring](http://doc.akka.io/docs/akka/2.3.11/general/supervision.html)__

我們自己產生的 Actor 會被歸類在 `user` 這一群。請特別注意 __user__。__user__ 在後面的談到的 Actor Path 會利用到。

在容錯的機制中，Supervisor 需要扮演監控及重啟 Child Actor 的角色，Akka 本身已有內建監控與重啟的 `Strategy`。如果沒有特別去 orverride，就會套用預設的機制。


### Actor Path

每一個 Actor 都會有一個 Actor Path，如下：

```
"akka://my-sys/user/service-a/worker1"                   // purely local
"akka.tcp://my-sys@host.example.com:5678/user/service-b" // remote
```
![The Interplay with Remote Deployment](http://doc.akka.io/docs/akka/2.3.11/_images/RemoteDeployment.png)

__Code and image from [Actor References, Paths and Addresses](http://doc.akka.io/docs/akka/2.3.11/general/addressing.html)__

簡單來說，在單機時：

```
val system = ActorSystem("HelloWorld")
val actor = system.actorOf(Props[MyActor], "MyHelloWorld")
```
上面 `actor` 的 path 會是 `akka://HelloWorld/user/MyHelloWorld` 也就是 `akka://System_Name/user/Actor_Name`，其中要特別注意 __user__，所以自己產生的 actor 都會被歸到 __user__ 路徑下。


同理在遠端上 Actor path 就會像 `akka.tcp://HelloWorld@host:port/user/MyHelloWorld`。

如果某個 actor 是被另一個 actor 產生時，則 path 會像：

```
akka://HelloWorld/user/parent/MyHelloWorld
```

Actor path 在系統中，是不能重覆的，因此在為 actor 命名時，要小心。


## 實作 - 把企鵝丟到遠處
延用上一篇 __吃飯、睡覺、打東東__ 的梗，將企鵝的 Actor 放在遠端的機器上執行。[Source code](https://github.com/kigichang/akka-sample) 放在我的 GitHub 上。


### 前置工作
* 首先要修改 `libraryDependencies` 原本是用 ~~`"com.typesafe.akka" %% "akka-actor" % "2.3.4"`~~ `"com.typesafe.akka" %% "akka-actor" % "2.3.11"`, 改成 Remote 版本 ~~`"com.typesafe.akka" %% "akka-remote" % "2.3.4"`~~ `"com.typesafe.akka" %% "akka-remote" % "2.3.11"`

* 在專案的目錄，新加 `src/main/resources`~~，並且加到 eclipse 專案的 `Source PATH`。這個目錄等一下要放設定檔。~~

### 問題

```
package com.example

sealed trait Question
object Interest extends Serializable with Question
object Why extends Serializable with Question

```

Singleton 要繼承 **Serializable** 否則訊息會傳不出去。

### 回答

新增 `PenguinReady`

```
package com.example

sealed trait Answer
case class Three(name: String, a: String, b: String, c: String) extends Answer
case class Two(name: String, a: String, b: String) extends Answer
case class Because(name: String, msg: String) extends Answer

case class PenguinReady(path: String)

```

### 角色

原本的角色，再新增 `LookupReporter`，`PenguinKing`，`DepolyReporter`。

```
package com.example

import akka.actor.Actor
import akka.actor.Props
import akka.actor.Identify
import scala.concurrent.duration.DurationInt
import akka.actor.ActorIdentity
import scala.concurrent.duration.Duration
import akka.actor.ActorRef

/**
 * 企鵝
 */
class Penguin(val name: String) extends Actor {
  def receive: Actor.Receive = {
    case Interest =>
      println(s"$name got Interest")
      sender() ! Three(name, "吃飯", "睡覺", "打東東")

    case Why =>

    case _ =>
  }

  override def preStart() = {
    println(s"$name start at ${self.path}")
  }
}

object Penguin {
  def apply(name: String): Props = Props(classOf[Penguin], name)
}

/**
 * 叫東東的企鵝
 */
class DongDong extends Penguin("東東") {

  override def receive = {
    case Interest =>
      println(s"$name got Interest")
      sender() ! Two(name, "吃飯", "睡覺")

    case Why =>
      println(s"$name got Why")
      sender() ! Because(name, s"我就是【${name}】")

    case _ =>
  }
}

object DongDong {
  val props = Props[DongDong]
}

/**
 * 記者
 */
class Reporter extends Actor {
  def receive: Actor.Receive = {

    /* 有三個興趣的回覆 */
    case Three(name, a, b, c) =>
      println(s"${name}: ${a}, ${b}, ${c}")

    /* 只有二個興趣的回覆，反問 why */
    case Two(name, a, b) =>
      println(s"${name}: ${a}, ${b}")
      sender ! Why

    /* 接到 why 的回覆 */
    case Because(name, msg) =>
      println(s"${name}: ${msg}")

    case _ =>
  }
}

object Reporter {
  val props = Props[Reporter]
}

/**
 * Lookup 版的記者
 */
class LookupReporter(penguins: Array[String]) extends Reporter {

  var count = 0

  def sendIdentifyRequest() {
    penguins foreach { path => println(path); context.actorSelection(path) ! Identify(path) }

    context.setReceiveTimeout(5 seconds)
  }

  sendIdentifyRequest

  def myReceive: Actor.Receive = {
    case ActorIdentity(path, Some(actor)) =>
      count += 1

      if (count == penguins.length) {
        context.setReceiveTimeout(Duration.Undefined)
      }

      println(s"${path} found")

      actor ! Interest

    case ActorIdentity(path, None) =>
      println(s"${path} not found")
  }

  override def receive: Actor.Receive = myReceive orElse super.receive

  override def preStart() = {
    println("Lookup Reporter start")
  }
}

object LookupReporter {

  def apply(penguins: Array[String]) = Props(classOf[LookupReporter], penguins)
}

/**
 * 企鵝王
 */
class PenguinKing(count: Int, reporter: ActorRef) extends Actor {

  def depoly = {
    for (i <- 0 until count - 1) {
      val penguin = context.actorOf(Penguin(s"penguin-$i"))

      penguin ! Identify(penguin.path.toString)

    }

    val dongdong = context.actorOf(DongDong.props)
    
    dongdong ! Identify(dongdong.path.toString)
  }

  depoly
  
  def receive: Actor.Receive = {
    case ActorIdentity(path, Some(actor)) =>
      println(s"$path found")
      reporter ! PenguinReady(path.toString)
      
    case ActorIdentity(path, None) =>
      println(s"$path not found")
  }
}

object PenguinKing {
  def apply(count: Int, reporter: ActorRef) = Props(classOf[PenguinKing], count, reporter)
}

/**
 * Depolyment 版記者
 */
class DepolyReporter extends Reporter {

  def myReceive: Actor.Receive = {
    case PenguinReady(path) =>
      val actor = context.actorSelection(path)
      actor ! Interest
  }

  override def receive = myReceive orElse super.receive
}

object DepolyReporter {
  val props = Props[DepolyReporter]
}

```


### Lookup 模式
Lookup 模式像 Client/Server，比較好理解。程式分成 `LookupServer` 及 `LookupClient`。執行順序：`LookupServer` -> `LookupClient`. 

* Lookup Server

```
package com.example

import com.typesafe.config.ConfigFactory

import akka.actor.ActorRef
import akka.actor.ActorSystem

object LookupServer {

  def main(args: Array[String]) {

    val system = ActorSystem("LookupServer", ConfigFactory.load("lookup-server"))

    val penguins = new Array[ActorRef](10)

    for (i <- 0 to 8) {
      penguins(i) = system.actorOf(Penguin(s"Penguin-$i"), s"penguin-$i")
      println(penguins(i).path)
    }

    penguins(9) = system.actorOf(DongDong.props, "dongdong")
    println(penguins(9).path)
  }
}

```

程式跟先前的差不多，主要差別在產生 ActorSystem 時，多了一個 `ConfigFactory.load("lookup-server")`，這個就是先前提到的設定檔，在開發環境下，會去 src/main/resources/ 目錄下找 `lookup-server.conf`

* lookup-server.conf

```
akka {
  actor {
    provider = "akka.remote.RemoteActorRefProvider"
  }

  remote {
    netty.tcp {
      hostname = "127.0.0.1"
      port = 2552
    }
  }
}

```

原則上，只要寫好設定檔，在程式開好 Actor，就完成一個可以被遠端呼叫的 Server。

* Lookup Client

```
package com.example

import akka.actor.ActorSystem
import com.typesafe.config.ConfigFactory

object LookupClient {

  def main(args: Array[String]) {
    val system = ActorSystem("LookupClient", ConfigFactory.load("lookup-client"))

    val remotePath = "akka.tcp://LookupServer@127.0.0.1:2552/user/"

    val penguins = new Array[String](10)

    for (i <- 0 to 8) {
      penguins(i) = s"${remotePath}penguin-${i}"
    }

    penguins(9) = s"${remotePath}dongdong"

    val reporter = system.actorOf(LookupReporter(penguins), "reporter")
    println(reporter.path)

    /* 主程式等一下，要不然上面都是 non-blocking call，會直接結束程式 */
    Thread.sleep(5 * 1000)
    system.shutdown
    println("end")
  }
}

```
	
* lookup-client.conf

```
akka {
  actor {
    provider = "akka.remote.RemoteActorRefProvider"
  }

  remote {
    netty.tcp {
      hostname = "127.0.0.1"
      port = 2554
    }
  }
}

```

Client 程式，先準備好遠端 Actor Path，然後傳給 `LookupReporter` 去呼叫遠端的 `Penguin`。

* LookupReporter

```
class LookupReporter(penguins: Array[String]) extends Reporter {

  var count = 0

  def sendIdentifyRequest() {
    penguins foreach { path => println(path); context.actorSelection(path) ! Identify(path) }

    context.setReceiveTimeout(5 seconds)
  }

  sendIdentifyRequest

  
  def myReceive: Actor.Receive = {
    case ActorIdentity(path, Some(actor)) =>
      count += 1

      if (count == penguins.length) {
        context.setReceiveTimeout(Duration.Undefined)
      }

      println(s"${path} found")

      actor ! Interest

    case ActorIdentity(path, None) =>
      println(s"${path} not found")
  }
  
  override def receive: Actor.Receive = myReceive orElse super.receive

  override def preStart() = {
    println("Lookup Reporter start")
  }
}

object LookupReporter {

  def apply(penguins: Array[String]) = Props(classOf[LookupReporter], penguins)
}

```

在 `LookupReporter` 中，我們使用 Actor 內建的 `context`，來查詢遠端的 actor： `context.actorSelection(path)`。

`LookupReporter` 跟先前一樣，但多了一個 `sendIdentifyRequest` 的函式，主要的功能是查詢遠端的 actor，是否已經 ready。在 Akka Actor 內建處理 `Identify` 的功能，我們可以對某個 Actor 傳送 `Identify`, `Identify` 的參數可以自定，用來辨識。當 Actor 是 OK 的話，則會收到 `ActorIdentity(id, Some(actor))`，其中的 `id` 就是先前在 `Identify` 加入的辨識字串。如果 Actor 沒有 Ready 好的話，則會收到 `ActorIdentity(id, None)`, 這時候，我們就可以知道那個 Actor 掛了。

執行起來的結果，你可以在 Server 的 console 上看到 `Penguin` output 的訊息，在 Client 端這邊，看到 `LookupReporter` 的 output。


### Deployment 模式
Deployment 模式，Master 可以指定 Actor 在那個遠端的 Node 來執行。這個範例純用設定檔的方式，來佈署 Actor。Akka 也允許在程式內，自行動態佈署。

執行順序：`DepolyNode` -> `DepolyMaster`

* Deploy Node - 用來執行被佈署的 Actor

```
package com.example

import akka.actor.ActorSystem
import com.typesafe.config.ConfigFactory

object DepolynNode {

  def main(args: Array[String]) {
    val system = ActorSystem("DepolyNode", ConfigFactory.load("depoly-node"))
  }
}
	
```
	
由上的範例，其實我們只要啟一個 `ActorSystem` 即可。

* deploy-node.conf

```
akka {
	actor {
		provider = "akka.remote.RemoteActorRefProvider"
	}
	  
	remote {
		netty.tcp {
			hostname = "127.0.0.1"
			port = 2551
		}
	}
}
	
```
	
其實跟上面的 `lookup-server.conf` 相似。

* deploy-master.conf
	
```	
akka {
	actor {
		provider = "akka.remote.RemoteActorRefProvider"

		deployment {
			"/penguin-king/*" {
			remote = "akka.tcp://DepolyNode@127.0.0.1:2551"
		}
	}
}

	remote {
		netty.tcp {
			hostname = "127.0.0.1"
			port = 2553
		}
	}
}
	
```

在 client 程式前，我們先看一下 client 的 conf 檔。跟先前的 `lookup-client.conf` 差不多，但多了一個 `deployment` 設定。上面的設定，是說明要將 actor path 是 `/penguin-king/*` 送到遠端 Node 執行。

* Deploy Master - 指定 Actor 在那些 Node 執行

```
package com.example

import com.typesafe.config.ConfigFactory
import akka.actor.ActorSystem

object DepolyMaster {

  def main(args: Array[String]) {
    val system = ActorSystem("DepolyMaster", ConfigFactory.load("depoly-master"))
    
    val reporter = system.actorOf(DepolyReporter.props)
    
    val king = system.actorOf(PenguinKing(10, reporter), "penguin-king")
    
    
    Thread.sleep(10 * 1000)
    
    system.shutdown()
    println("end")
  }
}

```

`DepolyMaster` 開啟一個 `PenguinKing` 的 actor，且註冊 actor name 是 `penguin-king`。`PenguinKing` 用來產生 `Penguin` 及 `DongDong`。如下：

```
def depoly = {
  for (i <- 0 until count - 1) {
    val penguin = context.actorOf(Penguin(s"penguin-$i"))

    penguin ! Identify(penguin.path.toString)

  }

  val dongdong = context.actorOf(DongDong.props)
    
  dongdong ! Identify(dongdong.path.toString)
}

```

在 `Actor` 中有 `context` 可以用來產生子 actor。由於 `PenguinKing` 的 actor name 是 `penguin-king`，這些子 `Penguin` 的 path 就會變成 `/penguin-king/xx`，就符合我們在 `deploy-master.conf` 設定檔上設定的 deployment 路徑 `/penguin-king/*`，佈署後，一樣使用 `Identify` 來確認是否完成佈署。當完成佈署後，通知 `DepolyReporter` 可以問題。

`PenguinKing` 的 `receive` 如下：

```
def receive: Actor.Receive = {
  case ActorIdentity(path, Some(actor)) =>
    println(s"$path found")
    reporter ! PenguinReady(path.toString)
  
  case ActorIdentity(path, None) =>
    println(s"$path not found")
}

```


`DepolyReporter` 的 `receive` 如下：

```
def myReceive: Actor.Receive = {
  case PenguinReady(path) =>
  val actor = context.actorSelection(path)
  actor ! Interest
}

override def receive = myReceive orElse super.receive

```


執行結果會像 lookup 模式，在 Server 端看到 `Penguin` 的 output，在 Client 端看到 `DepolyReporter` 的 output。