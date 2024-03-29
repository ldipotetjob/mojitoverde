---
title: Scala in a nutshell - Functions
date: 2017-03-28 09:00:00
tags:
    - scala
categories:
    - scala programming 
keywords:
    - scala
---
In this post, I will talk about the role of functions in scala, how to create them, and their implication on the scala language. Scala REPL (“**Read-Evaluate-Print-Loop**”),scala 2.x, will help us.

[Scala REPL page](https://docs.scala-lang.org/overviews/repl/overview.html):  

```
use :quit to exit from the scala REPL.
use tab for completion.
use :help for a list of commands.
use :paste to enter a class and object as companions.
```

**The telltale sign of a function with side effects is that its result type is Unit.** 

Defining simple functions in scala:

1. Defining a [Function1 (function of 1 parameter)](https://www.scala-lang.org/api/2.13.3/scala/Function1.html):

```scala
val f: Function1[Int,String] = myInt => "my Int value: " + myInt.toString
//val f: Int => String = $Lambda$1228/2144835163@9742012
f(20) //val res10: String = my Int value: 20
/* another way of Function1 */

val f1: Int => String = myInt => "my other Int value: " + myInt.toString
//val f1: Int => String = $Lambda$1230/2101422773@74ee07e
f1(20) //val res11: String = my other Int value: 20

/* another way of Function1 as an anonymous function */

val f2 = (x: Int) => "my new one Int value: " + x.toString
//val f2: Int => String = $Lambda$1232/711327915@1c724957
f2(20) //val res13: String = my new one Int value: 20

val formatArgs= (args: Array[String]) =>  args.mkString("\n") 
//val formatArgs: Array[String] => String = $Lambda$1235/699894580@3817d886

formatArgs(Array("functional","approach")) 
//val res7: String =
//functional
//approach

/* Defining function in a ay of <def> */

def formatArgs(args: Array[String]) = args.mkString("\n") 	//def formatArgs(args: Array[String]): String
formatArgs(Array("functional","approach")) 
//val res7: String =
//functional
//approach
```

2. Defining a [Function2 (function of 2 parameters)](https://www.scala-lang.org/api/2.13.3/scala/Function2.html) in this case with functional approach:

```scala
def max(x: Int, y: Int): Int = {
           if (x > y) x
else y } //def max(x: Int, y: Int): Int
max(10,20) //val res0: Int = 20

/* another way of Function2 as an anonymous function */
val max = (x: Int, y: Int) =>  if (x < y) y else x
//val max: (Int, Int) => Int = $Lambda$1254/801597836@5960fd98
max(10,20) //val res16: Int = 20
```

3. Function with [0 parameter](https://www.scala-lang.org/api/2.13.x/scala/Function0.html) as input and not returning value:

```scala
val name = "world"
val greeting = () => s"hello, $name" //val greeting: () => String = $Lambda$1259/1959562325@64b05928 

greeting() //val res18: String = hello, world
```

5. Defining closures

It is the finest [definition that I've found of closure](https://gist.github.com/ldipotetjob/08b41957431ec9014d02ac1c5e7eed23). It's a paradigm that can be applied to any language.

```scala
def closure_function(inc: Int) = (x: Int) => x + inc //def closure_function(inc: Int): Int => Int
val increment = closure_function(10) //val increment: Int => Int = $Lambda$1140/653292129@2b55ea4d
val increment5 = increment(5) //val increment5: Int = 15
scala> val increment5 = increment(10) //val increment5: Int = 20
```
Based on the above definition, you can use closure in scenarios where the value it returns depends on one or more variables(of any type) created or generated outside of it.

**Not purely functional**

```scala
def greet() = println("Hello, world!") //def greet(): Unit

/*Anonymous function*/
val greet = () => println("Hello, world!")

/* Other way of Function0 */
val greet: Function0[Unit] = () => println("Hello, world!")
```
In this post, you have seen different ways to create functions in Scala. We avoid the definition of Function 0..2 in Scala, mainly because It requires knowledge like **Trait** and **Object** as **Singleton**. We’ll address these concepts in later posts.




