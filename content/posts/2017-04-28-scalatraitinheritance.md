---
title: Scala - Trait - inheritance
date: 2017-04-28 09:00:00
tags:
    - Scala
categories:
    - scala programming 
keywords:
    - Scala
---
In this post we talk about the Scala traits and how to implement inheritance with classes. In some specifics scenarios like when you have to deal with base class with arguments it is beter to use abstract classes instead of traits.
Classes and objects can extends from traits which encapsulate methods and field defnitions.

A quite simple example below(base class with arguments) inherits(implements too) from abstract class:
```scala
abstract class Element {
  def contents: Array[String]
  def height = contents.length
  def width =
    if (height == 0) 0 else contents(0).length
} //defined class Element

// base class with arguments
class ArrayElement(aconts: Array[String]) extends Element {
  val contents: Array[String] = aconts
} //defined class ArrayElement

// base class with arguments
class LineElement(s: String) extends ArrayElement(Array(s)) {
  override  def width = s.length
  override def height = 1
} //defined class LineElement

val element: Element = new ArrayElement(Array("hello", "world"))
// element: Element = ArrayElement@1decbebb

element.width //res7: Int = 5
```

A better solution for multiple inheritance is a class mixin any number of traits.  

```scala
trait Printable{
  def print: Unit
}

class A6 extends Printable{
  def printA6: Unit = {
    println("Hello, this is A4 Sheet")
  }
}

      ^
error: class A6 needs to be abstract. Missing implementation for:
def print: Unit // inherited from trait Printable
```

Previous **error** because if a class inherits a trait it must implements all methods not implemented in the **trait**.

A proper solution:

```scala
trait Printable{
    def print: Unit
  }

trait Showable{
  def show
}

class A6 extends Printable with Showable{
  def print: Unit = {
    println("Hello, this is A6 Sheet")
  }

  def show: Unit = {
    println("This is showable")
  }
}
val paperSize = new A6 //val paperSize: A6 = A6@3f3b0b19
scala> paperSize.print //Hello, this is A6 Sheet
``` 





ref: [Scala standar library](https://www.scala-lang.org/files/archive/spec/2.13/public/images/classhierarchy.png)   