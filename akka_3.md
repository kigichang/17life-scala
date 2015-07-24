# Introduction of Akka Routing

當產生多個 actor 來處理相同事件時， Akka 提供 `Router` 的機制，來有效分配 message 給 actor 來完成工作。 在 Akka 中，受 `Router` 管理的 actor 稱作 `Routee`。

Akka 提供以下 routing 的管理方式：

* `akka.routing.RoundRobinRoutingLogic`
* `akka.routing.RandomRoutingLogic`
* `akka.routing.SmallestMailboxRoutingLogic`
* `akka.routing.BroadcastRoutingLogic`
* `akka.routing.ScatterGatherFirstCompletedRoutingLogic`
* `akka.routing.ConsistentHashingRoutingLogic`

參考資料：

[Akka Routing](http://doc.akka.io/docs/akka/2.3.12/scala/routing.html)


實作：

**RouterDemo** in [Sample Code](https://github.com/kigichang/akka-sample)

延用 __吃飯、睡覺、打東東__ 的範列，由 `PenguinKing` 產生 `Penguin`。`PenguinManager` 擔任 Router 角色，分配工作。`Reporter` 會透過 `PenguinManager` 詢問所有的企鵝，共 100 次，每隻企鵝會記錄被問了幾次，並回報給記者。

如何使用 `Router`

```
var router = {
  val routees = Vector.fill(5) {
    val r = context.actorOf(Props[Worker])
    context watch r
    ActorRefRoutee(r)
  }
  Router(RoundRobinRoutingLogic(), routees)
}

```

使用 Router 很簡單，首先將產生的 `ActorRef` 利用 `ActorRefRoutee`，再加入 `Router` 並設定此 `Router` 要使用那種管理方式，如上例使用 `RoundRobinRoutingLogic`。

使用 Router 時，記得把變數設成 `var`，因為當要加入/移除 Actor 時，Akka 會重新產生新的 `Router`，此時必須 re-assign 變數。


`PenguinManager`:

```
/**
 * 企鵝總管
 */
class PenguinManager extends Actor {
  var router = Router(RoundRobinRoutingLogic())

  def receive: Actor.Receive = {
    case PenguinReady(actor) =>
      context watch actor
      router = router.addRoutee(actor)
      println("manager watch " + actor.path)

    case Terminated(actor) =>
      println("manager remove " + actor.path)
      router = router.removeRoutee(actor)

    case Interest =>
      router.route(Interest, sender)

    case QueryCount =>
      router.routees foreach { actor =>
        actor.send(QueryCount, sender)
      }

    case Hit =>
      router.route(Hit, sender)

    case KillOne =>
      router.routees(0).send(PoisonPill, self)

    case _ =>
  }
}

object PenguinManager {
  val props = Props[PenguinManager]
}

```

如果要進一步監控 actor，可以使用 `watch` ，如上例 `context watch actor`，當被監控 actor 中止時，監控者會收到 `Terminated` 的訊息，其中會指名那個 actor 已經中止。如下：

```
case Terminated(actor) =>
  println("manager remove " + actor.path)
  router = router.removeRoutee(actor)
      
```

如果要測試不同的 routing 的方式，則修改 `var router = Router(RoundRobinRoutingLogic())` 即可。

