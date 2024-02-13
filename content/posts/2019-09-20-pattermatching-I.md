---
title: Pattern Matching - I
date: 2019-09-20 09:00:00
tags:
    - scala
    - spec 2.12
    - pattern matching
categories:
    - scala programming
keywords:
    - scala
    - pattern matching
---

[Pattern matching specification](https://www.scala-lang.org/files/archive/spec/2.12/08-pattern-matching.html)

This post covers the following patern matching specifications:

1. Pattern Sequences.
2. Infix Operation Patterns.
3. Pattern Alternatives.
4. Irrefutable Patterns.

[For previuos kinds of Pattern matching specifications review this link]( {{< ref "2019-07-20-pattermatching" >}})

---
[Pattern Sequences:](https://www.scala-lang.org/files/archive/spec/2.12/08-pattern-matching.html#pattern-sequences)

>SimplePattern ::= StableId ‘(’ [Patterns ‘,’] [varid ‘@’] ‘_’ ‘*’ ‘)’

```scala
val testList:List[Int] = List(1,1)

testList match {
  case List(1, _*) => s"hello $testList"
  case _ => " nothing match"
}

// res23: String = hello List(1, 1)

val testListbinding:List[Int] = List(1,1)

testList match {
  case testList@List(1, _*) => "a list beginning with 1, binding"
  case _ => " nothing match  binding"
}

// testListbinding: List[Int] = List(1, 1)
// res24: String = a list beginning with 1, binding

testList match {
  case x @ List(1, _*) => s"$x"
  case _ => " nothing match  binding"
}

// res25: String = List(1, 1)
```

[Infix Operation Patterns:](https://www.scala-lang.org/files/archive/spec/2.12/08-pattern-matching.html#infix-operation-patterns)

>Pattern3  ::=  SimplePattern {id [nl] SimplePattern}

```scala

/**
  *   Infix operator is a shorthand for the constructor or extractor pattern
  *   examples infix operator:
  *   
  *   infix operator for syntactic sugar
  *   ref: https://docs.scala-lang.org/tour/operators.html.
  *   
  *   examples of infix operators:
  *   +, -, *, ++,::,:+, etc and are just methods.
  *
  */

//normal notation
10.+(20)

//infix notation
10 + 20

case class Vec(x: Double, y: Double) {
  def +(that: Vec) = Vec(this.x + that.x, this.y + that.y)
}

val vector1 = Vec(1.0, 1.0)
val vector2 = Vec(2.0, 2.0)

//normal notation
val vector3 = vector1.+(vector2)
vector3.x  // 3.0
vector3.y  // 3.0

//infix notation
val vector4 = vector1 + vector2
vector4.x  // 3.0
vector4.y  // 3.0

//::
val xs:List[Int] = List(20,2,3,4)

xs match {
  case first :: second :: _ => println(s"the first element: $first and the second: $second" )
  case _ => -1
}

// the first element: 20 and the second: 2
// res33: AnyVal = ()

xs match {
  case rest +: tail => println(s"tail elements: $tail" )
  case _ => -1
}

// tail elements: List(2, 3, 4) 
// res34: AnyVal = ()


/**
  *   infix notation like tuples
  *   remember e(p1, p2), where e is the extractor and p1 and p2
  *   
  *   x(𝑞1,𝑝1) were X is the extractor and q1 and p1 operand (only two operands)
  *
  */

case class Person[A,B](name: A, age: B)

val student: Person[String,Int] = Person("Luis", 20)

val student1: Person[String,Int] = Person("Peter", 23)

Seq(student, student1) foreach {
  case name Person age => println(s"$name with $age")
  case currentstudent => println(s"Unknown: $currentstudent")
}

// student: Person[String,Int] = Person(Luis,20)

// student1: Person[String,Int] = Person(Peter,23)

// Luis with 20
// Peter with 23
// res36: Unit = ()

```

[Pattern Alternatives:](https://www.scala-lang.org/files/archive/spec/2.12/08-pattern-matching.html#pattern-alternatives)

>Pattern   ::=  Pattern1 { ‘|’ Pattern1 }

```scala
def matchDifferentPatterns(x: Any): Any = x match {
  case 1 | 2 => "one or two"
  case "two" => 2
  case 3 | 4 => "three or four"  
  case y: Int => "scala.Int"
  case _ => "many"
}

matchDifferentPatterns(1) // res37: Any = one or two

matchDifferentPatterns(2) // res38: Any = one or two

matchDifferentPatterns(3) // res39: Any = three or four

matchDifferentPatterns(4) // res40: Any = three or four
```

[Irrefutable Patterns:](https://www.scala-lang.org/files/archive/spec/2.12/08-pattern-matching.html#irrefutable-patterns)

>SimplePattern   ::=  StableId   

```scala
//𝑝 is a typed pattern 𝑥:𝑇′, and 𝑇<:𝑇′,
abstract class Device
case class Phone(model: String) extends Device {
  def screenOff = "Turning screen off"
}
case class Computer(model: String) extends Device {
  def screenSaverOn = "Turning screen saver on..."
}

def goIdle(device: Device) = device match {
  case p: Phone => p.screenOff
  case c: Computer => c.screenSaverOn
}


//𝑝 is a constructor pattern 𝑐(𝑝1,…,𝑝𝑛)

sealed abstract class Furniture
case class Couch() extends Furniture
case class Chair() extends Furniture

def findPlaceToSit(piece: Furniture): String = piece match {
  case a: Couch => "Lie on the couch"
  case b: Chair => "Sit on the chair"
}
```