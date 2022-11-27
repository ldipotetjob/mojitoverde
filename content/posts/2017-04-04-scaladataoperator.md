---
title: Scala in a nutshell - Data Types - Operators
date: 2017-04-04 09:00:00
tags:
    - Scala
categories:
    - scala programming
keywords:
    - Scala
    - Data Types
---

In this post we talk about Scala Data Types and its perspective in the easiest way.

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
Scala brings a mechanism for string interpolation, which allows to embed expressions within string literals. 
An example below:

```scala
val userName = "Luis" //val userName: String = Luis
println(s"Good morning, $userName!") //Good morning, Luis!

/* doing the calculations inside */
println(s"You have to check this value:${21%2}") //You have to check this value:1
```




