---
title: Scala - Classes - Objects
date: 2017-04-12 09:00:00
tags:
    - Scala
category: programming
keywords:
    - Scala
---
In this post we talk about Scala Class, Objects, Traits and Interfaces. Its relationships between different components and inheritance. We are assuming **that classes, methods, and polymorphism are familiar to the reader** and we will only delve into those terms that are new to the scenario at hand.

The next example or its idea have been taken from [Programming in scala, 3rd edition](https://www.amazon.co.uk/Programming-Scala-3rd-Martin-Odersky/dp/0981531687)

Defining classes in scala like in any other language  is a blueprint for creating objects.

Let's define a very simple implementation of the Rational numbers, just a few simple operations:

```scala
// Modeling Rational numbers 
class Rational(n: Int, d: Int) {
  // rise java.lang.IllegalArgumentException
  require(d != 0)
  private val g = gcd(n.abs, d.abs)
  val numer = n / g
  val denom = d / g
  def this(n: Int) = this(n, 1)

  // addition  operation between rational numbers
  def + (that: Rational): Rational =
    new Rational(
      numer * that.denom + that.numer * denom,
      denom * that.denom
    )
  // addition  operation between rational number and integer number
  def + (i: Int): Rational =
    new Rational(numer + i * denom, denom)

  // subtraction operation between rational numbers
  def - (that: Rational): Rational =
    new Rational(
      numer * that.denom - that.numer * denom,
      denom * that.denom
    )
  // subtraction operation between rational number and integer number
  def - (i: Int): Rational =
    new Rational(numer - i * denom, denom)

  // multiplication between rational numbers
  def * (that: Rational): Rational =
    new Rational(numer * that.numer, denom * that.denom)
  // multiplication between rational num and int number
  def * (i: Int): Rational =
    new Rational(numer * i, denom)

  //  division between rational numbers
  def / (that: Rational): Rational =
    new Rational(numer * that.denom, denom * that.numer)
  //  division between rational number and integer number
  def / (i: Int): Rational =
    new Rational(numer, denom * i)

  // we override here the default definition of toString method
  override def toString = s"$numer/$denom"

  // gcd: greatest common divisor
  // because "private" it can only be call inside the class
  private def gcd(a: Int, b: Int): Int =
    if (b == 0) a else gcd(b, a % b)
}

val rational: Rational = new Rational(3,4) //rational: Rational = 3/4
val rational1: Rational = new Rational(5,6) //rational1: Rational = 5/6
// Normal scenario
rational + rational1 //res0: Rational = 19/12
rational - rational1 //res1: Rational = -1/12

rational*(2) //res2: Rational = 3/2
rational+(3) res3: Rational = 15/4

```
Based in the revious scenario are operation between a cardinal numbers and rational numbers. Basically because the scala standar libraries do not contain operation with rational number. 
```scala
2*rational
        ^
       error: overloaded method * with alternatives:
         (x: Double)Double <and>
         (x: Float)Float <and>
         (x: Long)Long <and>
         (x: Int)Int <and>
         (x: Char)Int <and>
         (x: Short)Int <and>
         (x: Byte)Int
        cannot be applied to (Rational)

```

So basically a solution can be **implicit convertion**. Implicit definitions are those that the compiler is allowed to insert into a program in order to fix any of its type errors. For example, if x * Rational_number does not type check, then the compiler might change it
to convert(x) * Rational_number, where convert is some available implicit conversion. If convert changes x into something that has a * method that accept Rational Number, then this change might fix a program so that it type checks and runs correctly. 

Places where implicit convertions are implemented:

1. Conversions to an expected type. It lets you use one type in a context where a different type is expected
2. Conversions of the receiver of a selection. It lets you adapt the receiver of a method call (i.e., the object on which a method is invoked), if the method is not applicable on the original type
3. Implicit parameters. They are usually used to provide more information to the called function about what the caller wants. Useful with generic functions. 

So the solution to the previous compilation error can be the next line of code:

```scala
implicit def intToRational(x: Int) = new Rational(x)
                    ^
       warning: implicit conversion method intToRational should be enabled
       by making the implicit value scala.language.implicitConversions visible.
       This can be achieved by adding the import clause 'import scala.language.implicitConversions'
       or by setting the compiler option -language:implicitConversions.
       See the Scaladoc for value scala.language.implicitConversions for a discussion
       why the feature should be explicitly enabled.
def intToRational(x: Int): Rational

2*rational //val res6: Rational = 3/2
```
In the previous piece of code when we execute 2*rational what's really happen is **intToRational**(2)***rational**

The  warning on the previous piece of code can be solved with the following import on the scope of code:\
**scala.language.implicitConversions**

Implicits conversion can complicate the code and made it confuse so use it just when it's needed.

An **object** is a class that has exactly one instance. It is created lazily when it is referenced, like a lazy val. In scala doesn't exist static values but we have singleton objects which can be imported from everywhere in our code(**objects**). Every NOT private method can be used for any instant that imports the object previously. 

```scala
// Singleton object
object Person {
  import java.time.LocalDate
  val year = LocalDate.now.getYear
  private def calculateYearOfBirth(yearOfBirth:Int): Int =  year - yearOfBirth
}
```

If there is a class with the same name as our objects the class is named **companion class** and the object is named **companion object**. **class or object can access the private members of its companion**. Take a look at the code below:

```scala 
// Companion class
class Person(val firstName: String, val lastName: String, val yearOfbirth: Int, val address: String) {
   import Person.calculateAge
  val fullName = firstName + " " + lastName
  def this(firstName: String, lastName: String, year: Int) = this(firstName, lastName, year ,"")
  def printFullName = println(fullName)
  def age: Int = calculateAge(yearOfbirth)
}

// Companion object
object Person {
  import java.time.LocalDate
  val year = LocalDate.now.getYear
  private def calculateAge(yearOfBirth:Int): Int =  year - yearOfBirth
}
``` 

**Objects** can have constructors(apply) which take argument and create objects or extractors(unapply), which extracts object values(arguments) and give them back.   

```scala 
class PersonObj(firstName: String, lastName: String, yearOfbirth: Int, address: String){
  override def toString = s"Person $firstName $lastName born $yearOfbirth and live on $address"
}
// defined class PersonObj

object PersonObj {
 def apply (firstName: String, lastName: String, yearOfbirth: Int, address: String) =
   new PersonObj(firstName: String, lastName: String, yearOfbirth: Int, address: String)

  def unapply (personObj: PersonObj): Option[(PersonObj)] = Some(personObj)
}
// defined object PersonObj

val person = PersonObj("Peter", "Gabriel", 1970, "")

//person: PersonObj = Person Peter Gabriel born 1970 and live on

PersonObj.unapply(person)
//res5: Option[PersonObj] = Some(Person Peter Gabriel born 1970 and live on )
```

On code above, the unapply method returns Some(our_object) and  our_object will return the default method, toString, which has been overridden.

A better example is the scala **List object**, so let's check apply and unapply methods:

```scala
// List constructor
def apply[A](elems: A*): List[A]

// List extractor 
final def unapplySeq[A](x: List[A]): UnapplySeqWrapper[A]

// UnapplySeqWrapper class 
def apply(i: Int): A
```
So based on previous code, let's create and inspect the following List:

```scala
// list constructor 
List.apply(1, 2, 3)
val res0: List[Int] = List(1, 2, 3)

//list extractor 
res0.unapply(0)
val res13: Option[Int] = Some(1)

//list extractor 
res0.unapply(1)
val res14: Option[Int] = Some(2)

//list extractor 
res0.unapply(4)
val res15: Option[Int] = None
```

In the above code, pay attention to how unapply is invoking UnapplySeqWrapper.apply getting to the indexed value of the collection. As a proper extractor in case of List, its extractor extracts an specific index of a value introduced as an argument.

In this post we have talked about the objects as a new element to model our data in scala and its relationship with classes. I assume that abstract class and final class are familiar for you because they are not new in this programming language.
