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

Filtering List and have a look to the use of **\_**:
```scala
val cities:List[String] = List("Havana","Madrid","Oporto","Malaga","Matanza")
//val res2: List[String] = List(Madrid, Malaga, Matanza)
cities.filter(city=>city.startsWith("M")) == cities.filter(_.startsWith("M"))
```

Let's see an example about a list of integers that we want to divide in 2 groups. The elements > 15 and the rest. Pay attention that **_>15** has the same implication that **elem=>elem>15**.
Remember that firt will go all element that satisfy the conditions and the other groups those element that don't 
```scala
val numberList:List[Int] = List(1,3,4,5,8,20,28,14,12)
numberList.partition(_>15)
//val res12: (List[Int], List[Int]) = (List(20, 28),List(1, 3, 4, 5, 8, 14, 12))
```



&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;



ref: https://twitter.github.io/effectivescala/
