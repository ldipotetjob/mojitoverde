---
title: Scala - Abstrac  - Generic Type
date: 2018-06-13 09:00:00
tags:
    - Scala
categories:
    - scala programming 
keywords:
    - Scala
---
In this post we will talk about **Generic Elements**. One of the most useful mechanisms in the Scala programming language. This mechanism is quite useful when a member of a class, feature or even a method definition does not have a complete definition in its body and can also be usable by different implementations.

The following example is quite simple but jut to give an idea about using inheritance and polymorphism:

```scala
trait AddOper {
  type T
  def processElemnt(x: T, y: T): T
}

class ConcatString extends AddOper{
  type T = String
  override def processElemnt (x: String, y: String): String =
    x + y
}

class ConcatInt extends AddOper{
  type T = Int
  override def processElemnt (x: Int, y: Int): Int =
    x + y
}
``` 

In the same way, parameterization allows writing generic classes and traits. 
A simple example: with **List[T]** you can define List[String], List[Int], List[Any] 

```scala 
class Box[A](val content: A)

val intBox: Box[Int] = new Box[Int](20) //val intBox: Box[Int] = Box@11267e87
val listBox = new Box[List[Int]](List(12,30)) //val listBox: Box[List[Int]] = Box@469caf69
```

Another example is the next code is an extractor of any colection, in the next piece of code we deal with type erasure, I asume that you are familiar with it, quickly it refers to the runtime encoding of parameterized classes performed by Scala compiler in which it removes all the generic type information after compilation[^1].

```scala
import scala.reflect.ClassTag

def fNone[T]: Option[T] = None

def extractRightCollection[T](list: Seq[Any])(implicit tag: ClassTag[T]): Option[T] =
  list.headOption match {
    case Some(element: T) => Some(element)
    case _ => fNone[T]
  }

extractRightCollection[String](List("Elem1", "Elem2", "Elem3")) // val res25: Option[String] = Some(Elem1)

scala> extractRightCollection[Int](List(20, 10, 5)) // val res26: Option[Int] = Some(20)

scala> extractRightCollection[String](List(20, 10, 5)) // val res27: Option[String] = None
```

The **variance**, another way to defines inheritance relationships of parameterized types.


| Meaning       | Scala                          | notation |
| :---          | :-----------                   | :----    |
| covariant     | C[T’] is a subclass of C[T]    | [+T]     |
| contravariant | C[T] is a subclass of C[T’]    | [-T]     |
| invariant     | C[T] and C[T’] are not related | [T]      |


You can find some references on twitter documentation[^2]. 

Take a look to the following code where we will show you why you need here a **covariant** scenario:

```scala

trait Person{
  val name: String
  val age: Int
  val isStudent: Boolean
}

case class Student(name: String, age: Int, subjects: List[String]) extends Person{
  val isStudent: Boolean = !subjects.isEmpty
}

case class Worker(name: String, age: Int, subjects: List[String]) extends Person{
  val isStudent: Boolean = subjects.isEmpty
}

class PersonRegister[A](content: A) // class PersonRegister
val ps: PersonRegister[Student] = new PersonRegister[Student](Student("Louise", 18, List("Math", "Chemestry", "History")))
// val ps: PersonRegister[Student] = PersonRegister@57f5be22

val registerStudentAsPerson: PersonRegister[Person] = ps
                                                             ^
       error: type mismatch;
        found   : PersonRegister[Student]
        required: PersonRegister[Person]
       Note: Student <: Person, but class PersonRegister is invariant in type A.
       You may wish to define A as +A instead. (SLS 4.5)
```

Thus making the previous modification, indicated by the compiler we get the right compilation and create a **covariant** scope:

```scala 
// covariant: Student is a subtype of Person

class PersonRegister[+A](content: A)
val ps: PersonRegister[Student] = new PersonRegister[Student](Student("Louise", 18, List("Math", "Chemestry", "History")))
val registerStudentAsPerson: PersonRegister[Person] = ps // registerStudentAsPerson: PersonRegister[Person] = PersonRegister@ed52b9a
``` 

The next piece of code is the **contravariant scenario** where we assign to a subtype to its parent type. Keep in mind that Student is a subclass of Person:

```scala
// Student is a subtype of Person

class PersonRegister[-A](content: A) // defined class PersonRegister

val ps: PersonRegister[Person] = new PersonRegister[Person](
    new Person{
      val name: String = "Louise"
      val age: Int = 18
      val isStudent: Boolean = List("Math", "Chemestry", "History").isEmpty}
) // ps: PersonRegister[Person] = PersonRegister@1f171255

val registerStudentAsPerson: PersonRegister[Person] = ps // registerStudentAsPerson: PersonRegister[Person] = PersonRegister@1f171255
```

There are scenarios where we need **the utility of variances** but they can not be applied. For instance in implementations where there are **Reassignable fields**. The following piece of code is an example:

```scala
trait PersonRegister[-A]{
   val content: A
}
 val content: A
             ^
On line 2: error: contravariant type A occurs in covariant position in type A of value content

 trait PersonRegister[+A]{
   def content(element: A): A
 }
       
 def content(element: A): A
             ^
On line 2: error: covariant type A occurs in contravariant position in type A of value element
```

There is a solution for the above scenario, **Lower Bounds** and **Upper Bounds**

| Meaning       | Scala                             | notation |
| :---          | :-----------                      | :----    |
| Upper Bounds  | T refers to a subtype of type A.  | [T <: A] |
| Lower Bounds  | T refers to a supertype of type A.| [T >: A ]|



TODO !!! Examples y resumen.



[^1]:https://docs.oracle.com/javase/tutorial/java/generics/index.html
[^2]:https://twitter.github.io/scala_school/type-basics.html








