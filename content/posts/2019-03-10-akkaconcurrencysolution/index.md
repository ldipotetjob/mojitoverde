---
title: Akka an intelligent solution for concurrency
date: 2019-03-10 09:00:00
tags:
    - Scala
    - Akka
categories:
    - scala programming 
keywords:
    - Scala
    - Akka
---

We have a concurrency problem to solve: I need to access hundreds of thousands of websites, parse and process all of them asynchronously. I tried many options, always with a main idea: **Futures**.
So coding in that way I code something similar to the following:

```scala
//....................

//game_1: Int value indicating first game 

//game_n: Int value indicating game n   

val resultMatch:List[Future[List[Match]]] = (game_1 until game_n).map {
 matchnum =>
  Future{
   new ParserFootbalHTML("https://www.anyurl.com/..." + matchnum).finalMatch
    }}.toList
//..............
resultMatch foreach(future => future onComplete {
 Thread.sleep(1)
 doComplete
})
//...............
def doComplete: PartialFunction[Try[List[Match]],Unit] = { //
 case matches @ Success(_) =>{
  val resultMatch:List[Match] = matches.getOrElse(List.empty[Match])
    resultMatch.foreach(matches=>
//Do anything with matches
  )
 }
 case fail @ Failure(_) => println("error")
}
//....................
```

With minimal changes the main idea was to launch as many "Futures" as web sites we need to parse. Then after every computation finishes(onComplete) I will do "anything" with  the result. So after several tests in different escenarios checking how long it takes I decided to explore Akka, to improve performance and reduce time of calculation.
The Core of Akka libraries is [the Actor Model](https://en.wikipedia.org/wiki/Actor_model), I recommend understanding it because it is the base of these libraries. Akka has made its own implementation of this model.

![akka_actormodel_eng_img](/mojitoverde/images/akka_actormodel_eng.png#floatcenter)

We have an idea about the **Actor Model**, let's have a look about the actor model Akka implementation.

![akka_mailbox_img](/mojitoverde/images/akka_mailbox.png#floatleft)
**Actor and Actor-1** communicate exclusively by exchanging messages asynchronously which are placed into the recipient’s MailBox. \
**Actor and Actor 1** should avoid hogging resources. \
Message Should Not be mutable. \
Something to know from the official documentation: \
**The only meaningful way for a sender to know whether an interaction was successful is by receiving a business-level acknowledgement message, which is not something Akka could make up on its own (neither are we writing a “do what I mean” framework nor would you want us to).** \
So as we can see this is the real concept of encapsulation. **Everything happens inside the actor** and if you want to delegate a task then it sends a message to the actor and keeps working. For that reason you ought to granulate every task in the actor as simple as possible and you should have at the end that each actor processing be as simple as possible.

Split up the task and delegate it until they become small enough to be handled in One piece!

As you can see in the picture below: when you invoke the ActorSystem several actors have already been created.
Actor1 is a **Supervisor** of Actor1.1 and Actor 1.2

### Actor system heritage

![akka_actor_heritage_img](/mojitoverde/images/akka_actor_heritage.png#floatcenter)

Create only **One ActorSystem per application** is considered a **BestPractice** because it is a very heavy object and can be create by [different ways](https://doc.akka.io/api/akka/current/akka/actor/ActorSystem$.html):
* [ActorSystem.apply](https://doc.akka.io/api/akka/current/akka/actor/ActorSystem$.html#apply(name:String,config:Option[com.typesafe.config.Config],classLoader:Option[ClassLoader],defaultExecutionContext:Option[scala.concurrent.ExecutionContext]):akka.actor.ActorSystem)
* [ActorSystem.create](https://doc.akka.io/api/akka/current/akka/actor/ActorSystem$.html#create(name:String,config:com.typesafe.config.Config,classLoader:ClassLoader,defaultExecutionContext:scala.concurrent.ExecutionContext):akka.actor.ActorSystem)

The **ActorSystem** created can be in the same context or in any other **execution context created** by us and declared implicitly.
As we can see in [the actor system heritage figure](#actor-system-heritage) there are different ways of creating actors:
* Actors that are children of user "guardian" are created by: **system.actorOf**. I don't create too many actor of this type because this actors will be the own root actor of my app.
* Actor that will be child of my own actors(created by us) are created by: **context.actorOf**.The context is an **ActorContext**, an **implicit val** in the Actor trait.

The **ActorContext** exposes contextual info for the Actor:
* [self()](https://doc.akka.io/api/akka/current/akka/actor/ActorContext.html#self:akka.actor.ActorRef)
* [sender()](https://doc.akka.io/api/akka/current/akka/actor/ActorContext.html#sender():akka.actor.ActorRef)
* [watch()](https://doc.akka.io/api/akka/current/akka/actor/ActorContext.html#watch(subject:akka.actor.ActorRef):akka.actor.ActorRef): used for track actor activities and know when it stopped. Expected a "[Terminated](https://doc.akka.io/api/akka/current/akka/actor/Terminated.html)"
* [unwatch()](https://doc.akka.io/api/akka/current/akka/actor/ActorContext.html#unwatch(subject:akka.actor.ActorRef):akka.actor.ActorRef)
* [actorOf()](https://doc.akka.io/api/akka/current/akka/actor/ActorContext.html#actorOf(props:akka.actor.Props):akka.actor.ActorRef) 
* [become()](https://doc.akka.io/api/akka/current/akka/actor/ActorContext.html#become(behavior:akka.actor.Actor.Receive,discardOld:Boolean):Unit): change the actor behavior. We decide what message we processed and how. An actor can start processing some type of message and the change its behavior for any reason.
* [actorSelection](https://doc.akka.io/api/akka/current/akka/actor/ActorContext.html#actorSelection(path:akka.actor.ActorPath):akka.actor.ActorSelection)(actor path): We can select actor for its path (ref. [the actor system heritage figure](#actor-system-heritage) , user/Actor1/Actor1.2)

When we create actors then we get an ActorRef, this is an immutable reference and identifies the actor until it terminates his life. If an actor restarts its ActorRef will change.

Terminate(ActorRef  that is being watched, ref [watch](https://doc.akka.io/api/akka/current/akka/actor/ActorContext.html#watch(subject:akka.actor.ActorRef):akka.actor.ActorRef)). When it happens the Actor frees up its resources.

When we have an ActorRef:
[! or tell](https://doc.akka.io/api/akka/current/akka/actor/ActorRef.html#!(message:Any)(implicitsender:akka.actor.ActorRef):Unit): fire a message and forget.
[? or ask](https://doc.akka.io/api/akka/current/akka/pattern/AskSupport.html#ask(actorSelection:akka.actor.ActorSelection,message:Any,sender:akka.actor.ActorRef)(implicittimeout:akka.util.Timeout):scala.concurrent.Future[Any]): send a message asynchronously and return a future.  
As best practice: send a message via ! or tell.

We indicate below the best way to create and invoke different types of actors[^1]:

```scala
............

object BootNotScheduled extends App with ScalaLogger{

  val moviePageBoundaries: List[Int] = Try(List(args(0).toInt, args(1).toInt)) match {
    case Success(expectedList) => expectedList
    case  Failure(e) => log.error("Error with the boundaries of your page numbers,reviews your parameters {}",e.toString)
      System.exit(1)
      Nil
  }

  // This actor will control the whole System ("IoTDaemon_Not_Scheduled")
  val system = ActorSystem(ConstantsObject.NotScheduledIoTDaemon)

// creating ioTAdmin actor  
val ioTAdmin = system.actorOf(IoTAdmin.props, ConstantsObject.NotScheduledIoTDaemonProcessing)
  val filteredAndOrderedList = moviePageBoundaries.toSet.toList.sorted
  val filteredAndOrderedListGamesBoundaries = (filteredAndOrderedList.head to filteredAndOrderedList.last).toList

// Sending a message to ioTAdmin actor
ioTAdmin!ParseUrlFilms(filteredAndOrderedListGamesBoundaries,Option(ioTAdmin))
}
```

Props are config classes to specify options when you create actors. You can find below a snapshot configuring the actor ioTAdmin[^2] previously created in the above code:

```scala
........

object IoTAdmin {
  case class ParseUrlFilms(listTitles: List[moviePage], actorRef: Option[ActorRef]=None)
  case object ErrorStopActor
  case object StopProcess
  def props: Props = Props(new IoTAdmin)
}

...

}

```

The way we face configuration files solutions is like most libraries or frameworks or even in java language. I read the config info from application.conf but there are several way for do  that you can find it in [Akka config information](https://doc.akka.io/docs/akka/current/general/configuration.html#where-configuration-is-read-from), it does not explain at all how deal with it.

What do you have to consider when deal with configuration files in akka:

* Akka configuration values. We will use default values for the moment.
* Where the configuration file should be saved and how to name it.   
* How the configuration files are loaded or read. [related class/interfaces: [ConfigFactory](https://www.javadoc.io/doc/com.typesafe/config/1.3.3/com/typesafe/config/ConfigFactory.html), [Config](https://www.javadoc.io/doc/com.typesafe/config/1.3.3/com/typesafe/config/Config.html)] 

The main akka configuration files are usually **application.conf** and placed in the **"resource"** folder.
In our real life we usually need to work in different environments and sometimes we create our **application.conf** during the compilation process  other times we include in our specific configurations depending on the environments in which we are working. We have created a real world config example in which you can configure  a production(**production.conf**) or development(**development.conf**) environment. This is not the focus of this post so if you want learn how work, setting and manage config file you can go to my post about how [generating distributions for different environments](https://mojitoverdeintw.blogspot.com/2019/02/generating-scala-app-distribution-with.html).

We already have minimal information to create an Akka App only using the actor module anyway you can have a look to my repository [akka-quickstart-scala](https://github.com/ldipotetjob/akka-quickstart-scala) and check about how install and launching it(try the first option in your terminal[**sbt** "AkkaSchedulerPoc/runMain com.ldg.BootNotScheduled 1 5")]

We gave you here a first glance about how to solve concurrency problems in scala using akka libraries.


[^1]: BootNotScheduled code in [my github](https://raw.githubusercontent.com/ldipotetjob/akka-quickstart-scala/master/modules/AkkaScheduler/src/main/scala/com/ldg/BootNotScheduled.scala)
[^2]: ref. to IoTAdmin code in [my github](https://raw.githubusercontent.com/ldipotetjob/akka-quickstart-scala/master/modules/AkkaScheduler/src/main/scala/com/ldg/actors/IoTAdmin.scala)