---
title: Scala in a nutshell - Collections - List
date: 2017-03-21 09:00:00
tags:
    - scala
categories:
    - scala programming
keywords:
    - scala
---
In this post I'm going to explain one of the main collections in Scala, **List**. The performance in this inmutable collection for some scenarios is great with optimal features however, it doesn't work as well in others scenarios because this performance is proportional to the size of the collection.\
You can find the performance of List and other collections 
[here](https://docs.scala-lang.org/overviews/collections-2.13/performance-characteristics.html).\
Specific for **List**, this is what we have: 

| head  | tail | apply | update | prepend | append | insert |
| :---  | :--- | :---  | :---   | :---    | :---   | :---   |
| C     | C    | L     | L      | C       | L	   | -      |

**C**	The operation takes (fast) constant time.\
**L**	The operation is linear, meaning it takes time proportional to the collection size.


Based in the table above, you shouldn't for example **Append** elements to the list because the time it takes to append to a list grows linearly with the size of the list, whereas **Prepending** takes constant time. 

Alternative to Append:
1. Prepend + reverse
2. [ListBuffer](https://www.scala-lang.org/api/2.13.6/scala/collection/mutable/ListBuffer.html)

**Something you should know:** 
```
val: makes a variable immutable 
var: makes a variable mutable
```
Creating different types of Lists:
```scala
/** creating list */
val numbers: List[Int] = List(2, 3, 4, 5, 8, 100, 200, 300)
val cities: List[String] = List("Havana","Madrid","Oporto")
val listOfList: List[List[Int]] = List(List(1,2,3),List(0,1,3),List(4,2,1))
val emptyList: List[Nothing] = List()
val newList: List[Int] = Nil
```
Checking if the list is empty:
```scala
val emptyList = List() //val emptyList: List[Nothing] = List()
emptyList == Nil //val res7: Boolean = true
emptyList.isEmpty //val res8: Boolean = true
```

In previous sections of code we have seen the simplest way to generate a list as well as how to check if a list is empty.
A Scala list is composed of 2 parts:
* Block of Elements
* Nil

Adding elements to an empty List:
```scala
val newList :List[Int]= Nil
1::3::4::newList // res1: List[Int] = List(1, 3, 4)
newList.concat(List(1,2,3)) // res1: List[Int] = List(1, 3, 4)
```
The **filter** method of a collection, as its name suggests, given **a boolean expression** will return all those elements of the collection that meet that expression. An important detail of this method is that **if it finds elements that satisfy this expression, those elements will be returned in a collection of the same type before we performed the filtering. In case no element meets the expression, the **filter** method will return an empty collection of the type queried.**

Filtering List:
```scala
def filter(p: (A) ⇒ Boolean): List[A]
```
Let's see the following example where there is a list that satisfies the filtered condition(**line 1**) and another one(**line 2**) in which it does not.**Take a look at the use of the placeholder**(**\_**), although we will see it more in depth in future posts:
```scala
val cities: List[String] = List("Havana","Madrid","Oporto","Malaga","Matanza")
cities.filter(city=>city.startsWith("M")) == cities.filter(_.startsWith("M")) //res2: List[String] = List(Madrid, Malaga, Matanza)
cities.filter(city=>city.startsWith("x")) //val res0: List[String] = List()
```

Many of the collection methods return a standard **Scala type for optional values**, which are implemented by the Option type:

When a value is of type Option it indicates that it can be of two different types:
* Some (x) where x is the current value.
* None that means that it has no value or has a null value.

The **headOption** method returns the first element of the collection in a **Some(first_element)** or a **None** if no items meets the requirements.
```scala
def headOption: Option[A]
```

Filtering collections that fulfill a certain condition and some methods that help us chaining solutions. Check examples below:  

```scala
List(10,30,45,76,66,20).filter(_ > 30) //res45: List[Int] = List(45, 76, 66)

List(10,30,45,76,66,20).filter(_ > 30).headOption //res44: Option[Int] = Some(45)

List(10,30,45,76,66,20).filter(_ > 130).headOption //res47: Option[Int] = None
```

A solution similar to the previous code but more complete comes with the **find** method.

```scala
def find(p: (A) ⇒ Boolean): Option[A]
```

**Find** the first element that satisfies the Boolean condition. Our previous combination **(filter + headOption)** is done in a single method(**Find**), achieving the same result and the consequent advantage of its use due to doing almost everything  in a line.

For each element in the collections, as detailed below, it will be asked if it meets the established condition. The first element that meets this condition will be returned in a Some (the_element_searching). In case no element meets the condition, it will return None.

```scala
List(10,30,45,76,66,20).find(_ > 30) //val res51: Option[Int] = Some(45)
List("Havana","Madrid","Oporto").find(city=>city.contentEquals("Havana")) //val res9: Option[String] = Some(Havana)
List("Havana","Madrid","Oporto").find(city=>city.contentEquals("NotHavana")) //val res10: Option[String] = None
```

Creating a new collection by applying a specific function to each element that we need to make the map. **The elements will be returned in a collection of the same type as the collection involved in the operation.**
```scala
def map[B](f: (A) ⇒ B): List[B]
```
In code segment below, a new list is created by applying a function to each item in the collection. The two lines return the same collection, so let's see the utility of the placeholder(**\_**) in this scenario:

```scala
List(10,20,30,40,50).map(_ * 2) //res13: List[Int] = List(20, 40, 60, 80, 100)
List(10,20,30,40,50).map(element => element * 2) //res14: List[Int] = List(20, 40, 60, 80, 100)
```
```scala
def flong(x:Int): Int= x*2 // def flong(x: Int): Int 
def otherwayOFflong: Int => Int = x => x*2 // def otherwayOFflong: Int => Int

List(10,20,30,40,50).map(element => flong(element)) //res7: List[Int] = List(20, 40, 60, 80, 100)
List(10,20,30,40,50).map(flong(_)) //res8: List[Int] = List(20, 40, 60, 80, 100)
List(10,20,30,40,50).map(otherwayOFflong(_)) //res1: List[Int] = List(20, 40, 60, 80, 100)
```

Collections can be grouped following different constraints or conditions.

```scala
def groupBy[K](f: (A) ⇒ K): Map[K, List[A]]
```
So this method **splits** the collection into a map of lists according to a particular function. For example, if we group a list of String by its length, all those elements of the list with the same length will be in the same key K and will belong to the same List [A].

```scala
/** grouping by length */
List("test", "food", "trick", "deal", "canada", "drawing").groupBy(_.length)
//res9: scala.collection.immutable.Map[Int,List[String]] = HashMap(5 -> List(trick), 6 -> List(canada), 7 -> List(drawing), 4 -> List(test, food, deal))

/** grouping by number size */
List(1,2,34,33,56,56,32,20).groupBy(_<20)
//res12: scala.collection.immutable.Map[Boolean,List[Int]] = HashMap(false -> List(34, 33, 56, 56, 32, 20), true -> List(1, 2))
```

In a **partition**, all members of the collection that meet a boolean condition are grouped on the left side of the tuple, and those that do not fulfill it go to the right side.

```scala
def partition(p: (A) ⇒ Boolean): (List[A], List[A])
```
Let’s see the example below about a list of integers that we want to divide into **the integers > 15** and **the rest**. Integers that satisfy the conditions are grouped at the left side and the rest at the right side.
```scala
val numberList: List[Int] = List(1,3,4,5,8,20,28,14,12)
numberList.partition(_>15) //res12: (List[Int], List[Int]) = (List(20, 28),List(1, 3, 4, 5, 8, 14, 12))

val (elementGreaterThan10,elementLessThan10) = List(1,3,4,5,8,20,28,14,12).partition(number=>number>10)
val elementGreaterThan10: List[Int] = List(20, 28, 14, 12)
val elementLessThan10: List[Int] = List(1, 3, 4, 5, 8)

```

The Span method of a collection, returns a **(prefix,suffix)** tuple, from the first element of the collection that does not meet the condition will conform the second part of the tuple, no matter that there are other remaining elements that meet the condition.

```scala
final def span(p: (A) ⇒ Boolean): (List[A], List[A])
```

Below, we have applied the span method to an integer collection. It divides our collection starting from the number 20(not < 20), the first element that didn't meet the condition. **No matter that there are other elements after the number 20 that meet the condition**, the first one that fails and the other elements at its right side in the collection will form the other part of the tuple.
```scala
List(15, 10, 5, 20, 12, 8).span(_ < 20) //res1: (List[Int], List[Int]) = (List(15, 10, 5),List(20, 12, 8))
List(15, 10,45, 5, 20, 12, 8).span(_ < 20) //res2: (List[Int], List[Int]) = (List(15, 10),List(45, 5, 20, 12, 8))
```
The flatMap method constructs a new collection by applying a function to all items in the list and using the elements of the resulting collection. In general it makes sense in nested collections.
```scala
def flatMap[B](f: (A) ⇒ GenTraversableOnce[B]): List[B]
```
**flatMap()** method is identical to the map() method, but the only difference is that in flatMap the inner grouping of an item is removed and a sequence is generated. [**It can be defined as a blend of map method and flatten method**](https://www.geeksforgeeks.org/scala-flatmap-method/)
```scala
val cities: List[List[String]]=List(List("madrid","barcelona"),List("havana","cienfuegos"),List("london","manchester"))
cities.flatMap(x=>x.map(cities=>cities.toUpperCase)) //res12: List[String] = List(MADRID, BARCELONA, HAVANA, CIENFUEGOS, LONDON, MANCHESTER)
List(Option(10), Option(20), None).flatMap{(op: Option[Int])=> op} //val res42: List[Int] = List(10, 20)
List("madrid","londres","paris").flatMap(_.toUpperCase) //val res46: List[Char] = List(M, A, D, R, I, D, L, O, N, D, R, E, S, P, A, R, I, S)
List(Map("Six" -> 6, "five" -> 5, "four" -> 4),Map("Nine" -> 9, "eight" -> 8, "seven" -> 7)).flatMap(x=>x)
//val res49: List[(String, Int)] = List((Six,6), (five,5), (four,4), (Nine,9), (eight,8), (seven,7))
```

```scala
List(Some(1),Some(3),Some(10),None).flatMap(x=>x) //res41: List[Int] = List(1, 3, 10)
List(Some(1),Some(3),Some(10),None).flatten //res42: List[Int] = List(1, 3, 10)
```

Let’s see the operation of the fold method, which **“merges or synthesizes”** the elements of the collection using an associative operator (which can be executed in an arbitrary order [sum, multiplication]). **Always have an initial value or accumulator.** In the fold method, we talk about an associative binary operation which means the order of operations is irrelevant. **The fold method** is a monoid, have a look at our [source examples](https://github.com/ldipotetjob/functionalprog/blob/main/src/main/scala/functionalProg-monoids.sc)

```scala
def fold[A1 >: A](z: A1)(op: (A1, A1) ⇒ A1): A1
```

It is important to emphasize that in the fold method, the operations on the items in the collection do not have an order, the accumulator (z) can be applied as a parameter to the operator (op) an indeterminate amount of times.

```scala
List(1,2,3).fold("gdg")( _.toString + _.toString) //res0: Any = gdg123
// 1st iteration -> "gdg" + "1" -> "gdg1"
// 2nd iteration -> "gdg1" + "2" -> "gdg12"
// 3rd iteration -> "gdg12" + "3" -> "gdg123"
```

```scala
/** Integer acumulator */
List (1,2,3,4,5).fold (0)(_ * _) //res2: Int = 0

List (1,2,3,4,5).fold (0)(_ + _) //res3: Int = 15

List (1,2,3,4,5).fold (10)(_ + _) //res4: Int = 25

/** String acumulator */
List (1,2,3,4,5).fold ("")((s1, s2) => s"$s1 - $s2") //res5: Any = " - 1 - 2 - 3 - 4 - 5"

List (1,2,3,4,5).fold ("acumulator")((s1, s2) => s"$s1 - $s2") //res6: Any = acumulator - 1 - 2 - 3 - 4 - 5
```

There is difference when using the foldLeft method instead of fold. 
The following operation will generate an error because it is an operation in which **the order of the operations to be performed is decisive**:
```scala
List("1","2","3").fold(0)(_ + _.toInt)
<console> error: value toInt is not a member of Any
```

It is essential to consider no use the fold method in scenarios **where the order of the operations is important**.
Based on the above line of code would be **an error in the following analysis**:

1st iteration -> op (0, "1") -> 0 + "1" .toInt -> 1\
2nd iteration -> op (1, "2") -> 1 + "2" .toInt -> 3\
3rd iteration -> op (3, "3") -> 3 + "3" .toInt -> 6

Starting from the fact that the fold method nature itself indicates there is no set order for operations, nothing points out that the first iteration is op (0, “1”) -> 0 + “1” .toInt -> 1 could be any other combination in which for example is **op (0, “1”) -> 1 + 0.toInt -> 1** and we could have our first error.

A crucial detail of the functionality of the fold method is that depending on the object that applies this method, the concept of **“synthesize or merge”** will not be the same in all types of collections or other iterables on which **this method can be applied**, so we will assume here that the concept of **polymorphism** is known by all.
```scala
def foldLeft[B](z: B)(op: (B, A) ⇒ B): B
```
As noted in the specification op (... op (z, x_1), x_2, ..., x_n) we will apply the operator in a direction from **LEFt** -> **RIGht**

```scala
/** this scenario fail*/
List("1","2","3").fold(0)(_ + _.toInt) //<console> error: value toInt is not a member of Any

/** this scenario succeed */
List("1","2","3").foldLeft(0)(_ + _.toInt) //res11: Int = 6
```
So from **LEFt** to **RIGht** Sequence order of the operations executed here is as follows:

1st iteration (0 + "1" .toInt) -> 1\
2nd iteration (1 + "2" .toInt) -> 3\
3rd iteration (3 + "3" .toInt) -> 6 [the last result]

The same point of view is applied to foldRight.

```scala
def foldRight[B](z: B)(op: (A, B) ⇒ B): B
```

As you can appreciate syntax of the method is the same even if the way to do the iterations is **RIGht -> LEFt**

Below, simple examples how foldRight works:

```scala
List(1,2,3,4,5,6,7,8,9,10).foldRight(1)((a,b) => {println(s"$a,$b");a})
10,1
9,10
8,9
7,8
6,7
5,6
4,5
3,4
2,3
1,2
//val res0: Int = 1
```

```scala
List("1","2","3").foldRight(0)(  _.toInt + _) //val res31: Int = 6
```

We see how it has been necessary to change the order of the parameters. So let's see the order of the operations.

**Important: See the order or position that the accumulator (on the right) occupies in our case.**


1st iteration op (first_element_element, accumulator) -> Result_1\
2nd iteration op (second_element_element, Result_1) -> Result_2\
3rd iteration op (third_element_element_, Result_2) -> Result_3

The Common operation in fold methods is when the collection is empty, this cases return the accumulator, in general, it must be **neutral**.

What does neutral means:

If it's a list -> Nil\
If it is a multiplication operator -> 1\
If it is a String operator -> ""\
And so on.

We can adapt our code to use the fold method. Many times it won't be able. If **foldRight** and **foldLeft**  are equivalent, choose foldLeft by default.

In the **reduce** method, whose main difference from the fold method is that it does not have an initial value, the collection is "reduced or merged" using associative operators.
Make sure to use this method in the not-empty collection.

```scala
def reduce[A1 >: A](op: (A1, A1) ⇒ A1): A1
```

The **reduce method** is quite similar to the fold method with two main differences, **do not include any acumulator** also **you must guarantee do not traverse an empty collection**(list in this case). 
```scala
List (1,2,3,4,5).fold ("")((s1, s2) => s"$s1 - $s2") //res20: Any = " - 1 - 2 - 3 - 4 - 5"
/** check difference between the leftmost char comparing against the above line of code */
List ("1","2","3","4","5").reduce((s1, s2) => s"$s1 - $s2") //res22: String = 1 - 2 - 3 - 4 - 5
```

```scala
List(10,30,45,76,66,20).reduce(_ min _) //res23: Int = 10
List(10,30,45,76,66,20).min ////res23: Int = 10
List(10,30).reduce(_ - _) //res27: Int = -20

List(10,30,45,76,66,20).reduceLeft(_ min _) //res28: Int = 10
List(10,30,45,76,66,20).reduceRight(_ min _) //res29: Int = 10

val lines: List[String] = List("firsts_string_value","second_string_value", "third_string_value")
val longestLine = lines.reduceLeft((a, b) => if (a.length > b.length) a else b)
//val longestLine: String = second_string_value
```

In the previous use case better to use the **reduce** method. But suppose our op is a **division** instead of being **max** or **min**; then we must care about the order of the operations and decide what **reduces method** (left or right) to use.


&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;

ref: https://twitter.github.io/effectivescala/ \
&nbsp; &nbsp; &nbsp;[reference for scala api  collection list](https://www.scala-lang.org/api/2.13.3/scala/collection/immutable/List.html) \
&nbsp; &nbsp;&nbsp; [Monoids](https://www.baeldung.com/scala/monoids-semigroups#monoids)
