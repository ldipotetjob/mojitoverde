---
title: Scala in a nutshell - Data Types - Operators
date: 2017-04-04 09:00:00
tags:
    - scala
categories:
    - scala programming
keywords:
    - scala
    - Data Types
---

In this post, we talk about Scala Data Types and their perspective in the easiest way.

```
Basic type    Range

Byte          8-bit signed two's complement integer (-27 to 27 - 1, inclusive)
Short         16-bit signed two's complement integer (-215 to 215 - 1, inclusive)
Int           32-bit signed two's complement integer (-231 to 231 - 1, inclusive)
Long          64-bit signed two's complement integer (-263 to 263 - 1, inclusive)
Char          16-bit unsigned Unicode character (0 to 216 - 1, inclusive)
String        a sequence of Chars
Float         32-bit IEEE 754 single-precision float
Double        64-bit IEEE 754 double-precision float
Boolean       true or false
```
Scala brings a mechanism for string interpolation, which allows embedding expressions within string literals:

```scala
val userName = "Luis" //val userName: String = Luis
println(s"Good morning, $userName!") //Good morning, Luis!

/* doing the calculations inside */
println(s"You have to check this value:${21%2}") //You have to check this value:1
```

Repeated params, It implies that a param in a function can include one or more elements of the specific type:
```scala
def echo(args: String*) =
            for (arg <- args) println(arg)
// def echo(args: String*): Unit

echo("hello") // hello
echo("hello","i am louis")
// hello
// i am louis

echo("hello","i am louis","and you")
// hello
// i am louis
// and you
```

Even when the parameter is declared as an array you can't pass the collection like array, list, sets or other type of collection as a parameter:

```scala
// the same scenario when using arrays 
echo(List("hello","i am louis","and you ?"))
                ^
       error: type mismatch;
        found   : List[String]
        required: String
```

To solve the previous error, we need to tell the compiler to pass each item in the list as its parameter rather than all of it(the collection) as a single argument. The items of the list are considered a single argument: _*.

\
Check below code:

```scala 
val myechoList = List("hello","i am louis","and you ?")
// val myechoList: List[String] = List(hello, i am louis, and you ?)

echo(myechoList: _*)
hello
i am louis
and you ?
```

The idea of this post is an overview of data types in Scala and somehow pass repeated parameters to function and handle it that takes as parameter repeated parameters or collections too.