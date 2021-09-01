---
title: Scala in a nutshell I - Collections - List
date: 2017-03-21 09:00:00
tags:
    - Scala
category: programming
keywords:
    - Scala
---
In this post I am gonna explain ONE of the main collection in Scala, List. The performance in this inmutable collection for some scenarios is great with optimal features and not so good in others because this performance is proportional to the size of the collection.\
You can find the performance of List and other collections 
[here](https://docs.scala-lang.org/overviews/collections-2.13/performance-characteristics.html).\
Especific for List this is what we have: 

| head  | tail | apply | update | prepend | append | insert |
| :---  | :--- | :---  | :---   | :---    | :---   | :---   |
| C     | C    | L     | L      | C       | L	   | -      |

Differents way to create Lists:
```scala
/** creating list */
val numbers: List[Int] = List(2, 3, 4, 5, 8, 100, 200, 300)
val cities: List[String] = List("Havana","Madrid","Oporto")
val listOfList: List[List[Int]] = List(List(1,2,3),List(0,1,3),List(4,2,1))
val emptyList: List[Nothing] = List()
val newList: List[Int] = Nil
```

Adding element to an empty List:
```scala
val newList :List[Int]= Nil
1::3::4::newList // res1: List[Int] = List(1, 3, 4)
```

Filtering List and have a look about the use of **\_**:
```scala
val cities: List[String] = List("Havana","Madrid","Oporto","Malaga","Matanza")
//val res2: List[String] = List(Madrid, Malaga, Matanza)
cities.filter(city=>city.startsWith("M")) == cities.filter(_.startsWith("M"))
```

Creating a new collection by applying an specific function to each of the elements of the collection on 
which we want to make the map. **The elements will be returned in a collection of the same type that the collection involved in the operation.**
```scala
List(10,20,30,40,50).map(_ * 2) //res13: List[Int] = List(20, 40, 60, 80, 100)
List(10,20,30,40,50)..map(element => element * 2) //res14: List[Int] = List(20, 40, 60, 80, 100)
```
```scala
List(10,20,30,40,50).map(_ * 2) //res5: List[Int] = List(20, 40, 60, 80, 100)

def flong(x:Int): Int= x*2
def  otherwayOFflong: Int => Int = x  => x*2

List(10,20,30,40,50).map(element => flong(element)) //res7: List[Int] = List(20, 40, 60, 80, 100)
List(10,20,30,40,50).map(flong(_)) //res8: List[Int] = List(20, 40, 60, 80, 100)
```


 flatMap() method is identical to the map() method, but the only difference is that in flatMap the inner grouping of an item is removed and a sequence is generated. [**It can be defined as a blend of map method and flatten method**](https://www.geeksforgeeks.org/scala-flatmap-method/)
```scala
val ciudades: List[List[String]]=List(List("madrid","barcelona"),List("havana","cienfuegos"),List("london","manchester"))
ciudades.flatMap(x=>x.map(cities=>cities.toUpperCase)) //res12: List[String] = List(MADRID, BARCELONA, HAVANA, CIENFUEGOS, LONDON, MANCHESTER)
```

```scala
List(Some(1),Some(3),Some(10),None).flatMap(x=>x) //res41: List[Int] = List(1, 3, 10)
List(Some(1),Some(3),Some(10),None).flatten //res42: List[Int] = List(1, 3, 10)
```

Filtering collections and some methods that help us to chaining solutions. Check examples below.  

```scala
List(10,30,45,76,66,20).filter(_ > 30) //res45: List[Int] = List(45, 76, 66)

List(10,30,45,76,66,20).filter(_ > 30).headOption //res44: Option[Int] = Some(45)

List(10,30,45,76,66,20).filter(_ > 130).headOption //res47: Option[Int] = None
```

Find the first element, if it exists and satisfies the Boolean condition. Our previous combination (filter + headOption) is done in a single method, achieving the same result and the consequent advantage of its use, in a line do almost everything.

```scala
List(10,30,45,76,66,20).find(_ > 30)
val res51: Option[Int] = Some(45)
```

Let's see an example about a list of integers that we want to divide in 2 groups. The elements > 15 and the rest. Pay attention that **_>15** has the same implication that **elem=>elem>15**.
Remember that firt will go all element that satisfy the conditions and the other groups those element that don't 
```scala
val numberList: List[Int] = List(1,3,4,5,8,20,28,14,12)
numberList.partition(_>15) //res12: (List[Int], List[Int]) = (List(20, 28),List(1, 3, 4, 5, 8, 14, 12))
```

As we can see at the below code segment, we have made a span with those elements less than 20 and has divided our list starting from the number 20(is not less than 20), the first element that did not comply with the condition. No matter that there are other elements after the number 20 that meet the condition, from the first one that fails will form the other part of the tuple. **Once again the elements will be returned in a collection of the same type that the collection involved in the operation.**
```scala
List(15, 10, 5, 20, 12, 8).span(_ < 20) //res1: (List[Int], List[Int]) = (List(15, 10, 5),List(20, 12, 8))
List(15, 10,45, 5, 20, 12, 8).span(_ < 20) //res2: (List[Int], List[Int]) = (List(15, 10),List(45, 5, 20, 12, 8))
```

Let's see the operation of the fold method which "merge or synthesize" the elements of the collection using an associative operator (which can be executed in an arbitrary order [sum, multiplication]). Always have an initial value or accumulator.
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

Understanding difference when foldLeft instead of fold.
The following operation will generate an error because it is an operation in which **the order of the operations to be performed is decisive**:
```scala
List("1","2","3").fold(0)(_ + _.toInt)
<console> error: value toInt is not a member of Any
```

It is important to note the previous behaviour in situations like this (**where the order of operations is important**) we can not consider the use of fold. Based on the previous line of code would be **an error the following analysis**:

1st iteration -> op (0, "1") -> 0 + "1" .toInt -> 1\
2nd iteration -> op (1, "2") -> 1 + "2" .toInt -> 3\
3rd iteration -> op (3, "3") -> 3 + "3" .toInt -> 6

Starting from the fact that the fold nature itself indicates that there is no set order for operations then **NOTHING** indicates that the first iteration is op (0, "1") -> 0 + "1" .toInt -> 1 could just be any other combination in which for example is **op (0, "1") -> 1 + 0.toInt -> 1** and we would already have our first error.

We must know an important detail of the functionality of fold, depending the object that apply this method the concept of **"synthesize or merge"** will not be the same in all types of collections or objects on which **fold method can be applied**, so we will assume here that the concept of **polymorphism** is known by all.

```scala
/** this scenario fail*/
List("1","2","3").fold(0)(_ + _.toInt) //<console> error: value toInt is not a member of Any

/** this scenario succeed */
List("1","2","3").foldLeft(0)(_ + _.toInt) //res11: Int = 6
```
The sequence order of the operations executed here is as follows:

1st iteration (0 + "1" .toInt) -> 1\
2nd iteration (1 + "2" .toInt) -> 3\
3rd iteration (3 + "3" .toInt) -> 6 [the last result]

The same point of view is applied to foldRight.

**fold**, it is a monoids, have a look to our [source examples](https://github.com/ldipotetjob/functionalprog/blob/main/src/main/scala/functionalProg-monoids.sc)

**reduce** is quite similar to fold but don't include acumulator and **the ONLY** aspect that you have to guarantee is just 
don't traverse an empty collection(list in this case) 
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
```

The previous use case would be better to use **reduce**. But imagine that our op instead of being **max** or **min**, is a **division**, in this case we must care about the order of the parameters and we would then have to see what **reduces method** (left or right) to use.

&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;

ref: https://twitter.github.io/effectivescala/


