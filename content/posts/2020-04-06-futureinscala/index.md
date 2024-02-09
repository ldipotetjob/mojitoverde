---
title: With Futures in Scala who brings the core, Java Concurrency api?
date: 2020-04-06 09:00:00
tags:
    - scala
categories:
    - scala 
keywords:
    - scala
    - future
    - sbt
---

I will not talk about the syntax of **Scala's Future**. You can find everything at [Scala Official documentation](https://docs.scala-lang.org/overviews/core/futures.html). 

It is clear what Scala Futures is for. I think that it is the best solution for concurrency and any other asynchronous processing.

My challenge today will be to try to explain the core of **Scala Futures** and who provides it. I will attempt to give you an idea in relation to  the link between the **Scala Concurrency package** and the **Java Concurrency package**. Like in too many other Scala modules in concurrency, Scala constructs its core over Java concurrency pillars.

I guess you know the following concepts, but I want to mention some of them:

* A **Runnable** is a type of class ([Runnable](https://docs.oracle.com/javase/7/docs/api/java/lang/Runnable.html) is an Interface) that can be put into a thread as is explained in the java spec [Should be implemented by any class whose instances are intended to be executed by a thread].
* A **Thread** is a thread of execution in a program where run lightweight processes.

![concurrentprogramming_img](/mojitoverde/images/concurrentprogramming.png#floatcenter)

There is a connection between the task a new thread executes(defined by its Runnable object) and  the thread itself(defined by a Thread object) in every **small aplication**. 
In large applications, it makes sense to separate thread management and creation from the rest of the application so in these cases. 
The **Executors** are the objects that encapsulate these tasks(**creation of threads**, **task execution in these threads**, etc.)

![executors_img](/mojitoverde/images/executors.png#floatcenter)

The strategy used in Scala to deal with the Futures will be the **Executor**, who is responsible for executing computations. The [Executor](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executor.html) comes from the java concurrency api. Futures in Scala revolve around **ExecutionContexts**, but this is nearly a copy of  java **Executor**. The Scala ExecutionContext implements the Executor java interfaces, so behind the scecene all  computations are executed in a wrapper of previosly mentioned java **Executor**.

```scala
.........

/**
 * An [[ExecutionContext]] that is also a
 * Java [[http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executor.html Executor]].
 */
trait ExecutionContextExecutor extends ExecutionContext with Executor

/**
 * An [[ExecutionContext]] that is also a
 * Java [[http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html ExecutorService]].
 */
trait ExecutionContextExecutorService extends ExecutionContextExecutor with ExecutorService

.........
```

The snapshot source code above is **ExecutionContext.scala**. The main job here is done by java concurrency api, once again. Execution context executes Runnable(the task that must be to executed).

In Java concurrency Api, the most of the executor implementations use **thread pools**, which consist of **worker threads**. It exists separately from the Runnable and Callable tasks it executes and is often used to execute multiple tasks.

![threadpools_img](/mojitoverde/images/threadpools.png#floatcenter)

Using worker threads minimizes the overhead due to thread creation. 
The thread objects use a significant amount of memory, and in a large-scale application, allocating and deallocating many thread objects creates a significant memory management overhead.

![threadspooltree_img](/mojitoverde/images/threadspooltree.png#floatcenter)

One common type of thread pool is the fixed thread pool, which has a specified number of threads running. If a thread somehow terminates while It is in use,  a new thread will replace it . An internal queue submits the tasks to the pool via, which holds extra tasks whenever there are more active tasks than threads.

One of the Cores in our ExecutionContext that is behind the scenes is the Java Fork/Join framework.
Fork/join lets us take advantage of multiple processors when we work in this kind of environment. It ditribute tasks to **worker threads**(explained above) in a thread pool. The core of this Framework is the **ForkJoinPool** class.

The **ForkJoinPool**'s goal is to use all the available processing power to enhance the performance of your application. There are some aspects that you can get as default values. You can set another properties for a custom performance as [available processors](https://docs.oracle.com/javase/7/docs/api/java/lang/Runtime.html#availableProcessors).

We can create our own ExecutionContext, but the language recommendation is to use the **ExecutionContext.global** in **scala.concurrent** package. The **Global Execution Context** is backed by the **ForkJoinPool** aforementioned. The java concurrency api does the main job once again.

The **ExecutionContext.global** sets the parallelism level of its underlying fork-join pool to the number of available processors by default, but you can set this properties and others like :

* scala.concurrent.context.minThreads - defaults to Runtime.availableProcessors
* scala.concurrent.context.numThreads - can be a number or a multiplier (N) in the form ‘xN’; defaults to Runtime.availableProcessors
* scala.concurrent.context.maxThreads - defaults to Runtime.availableProcessors

I wanted to give you an idea about the Scala Future behind the scenes. In my opinion, everything is a wrapper of java.concurrency api.
