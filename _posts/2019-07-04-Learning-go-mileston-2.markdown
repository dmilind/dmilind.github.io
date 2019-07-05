---
title: "Learning Golang: Milestone 2"
layout: post
date: 2019-07-04
image: /assets/images/markdown.jpg
headerImage: false
tag:
- go
- golang
- go aggregate data types
- go's array, slice, maps, struct
category: blog
author: Milind
description: Learning Golang
---

Welcome to second part of learning go . . .

## Composite Data Types

Basic data types like int, bool, rune, string can be considered as an atom of go's data structure whereas Composite data type is higher level than basic data type.

Composite data types consist of array, struct , slice and map. So what exactly is that ?
When any data is processed in a program, that data can be in any format. This data should be injected to the program in a defined structure so that program can understand and act on it accordingly. To organize such data in proper format we use these data structures in go.

## Array
An array is aggregate data type which encapsulates <u><b>fixed length of same data</b></u>. It is kind of a list where are data elements have same data type and exact number of elements are specified. Array can not be shrink or expand once initialized.

Syntax to declare an Array

```
var item [10]int    // item is an array where 10 elements will be stored in a consecutive   memory location.  

var item [10]int = [10]int{1,2,3,4,5,6,7,8,9,10}
```

Sometime we dont know how many element will be initialized, in that case we can use <b>ellipsis `...` </b>

```
var item [...]int = [...]int{1,2,3}  // ellipsis appeared in the place of length.
```
#### Coding Fun: Implementing An Array
```
/*
Go program to get all months.
*/
package main
import (
	"fmt"
)
func main() {
	// initializing array of months
	months := [12]string{"January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"}
	// printing each of the month one by one
	for i, _ := range months {
		fmt.Println(months[i])
	}
}
```
<a href='https://play.golang.org/p/tK4a7cfjh_X'>Implementing An Array: Go Playground</a>

## Slices

Slices is modern data type form of array. Array has some limitations on elements expansion and shrinking.Elements from the slices can be shrink and expanded as per need.Actually slices are nothing but an array inside.
Slice can be declared as
```
var sliceName []int // slice containing all integers.
sliceName := []int{1,2,3,4}
```
When slice is initialized, under the hood an array is created and from that array slice will be referenced.`make()` function can be used to declare a slice.
```
sliceName := make([]int,10)
```
> NOTE: make() function allocates and initiliazes an object of type slice, map or chan only.

Example:

Lets take an example from array, creating a slice of all months.

```
// slices
package main

import (
	"fmt"
)

func main() {
	// initialiazing a slice
	months := make([]string, 0, 12) // hey compiler, make a slice where length of element is 0 and at max (cap) limit is 12.
	// here an underlying array has been created and slice,s pointer is set to 0th location.
	// this slice is empty now.
	// fmt.Println(months) // output: []
	// now lets fil this slice by the months, we can use append() function to fill the slice.

	months = append(months, "January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December")
	fmt.Println(months)
	for i, _ := range months {
		fmt.Printf("%v ", &months[i])
	}

	// finding length and capacity of the the slice.
	fmt.Println("\nLength of slice is", len(months), "and capacity is", cap(months))

	// trying to add new elements to the months
	months = append(months, "newMonth")
	fmt.Println(months)
	fmt.Println("\nLength of slice is", len(months), "and capacity is", cap(months))
	for i, _ := range months {
		fmt.Printf("%v ", &months[i])
	}


}

```
<a href='https://play.golang.org/p/e-KYCG_jZ17'>Implementing Slice: Go Playground</a>

## Builtin <i> append </i> Function
When slice is initialized , we might want to add new element to this slice. This addition can be done by using a builtin append function. Once appened function called , it takes arguments which can be added to the existing slice.
```
Syntax:

sliceName := append(sliceName, element) // element will be appended to sliceName
```
You can checkout the above example.

> If element is appended to a slice where capacity is filled, an extra underlying array will be initialized and slice of same capacity will be referenced.

## Maps
We have seen array and slices, which are a linear one dimensional data structure in go.They can be uses as we get data in linear fashio. What if the data is in a hash or dictionary format, like
```
key1: value1
key2: value2
```
If we get this kind of data to be processed through program then we can use map data structure.
maps stores the data in key value format. It is unordered collection of key value pairs.

* Keys are dictinct from each other.
* Key type is always identical
* Value type is always identical
* Key and Value type may be identical but not necessary.  

#### Declaring/Initializing a map
```
var variableName map[key_type]value_type
var mapItem map[string]int  // asking to declare a map (hash table or dictionary) which stores
                            // a key value pair. Key is string and value is int
```

`make()` function can also be used to declare a map.
```
mapItem := make(mapp[string]int)
```
Example to understand maps and actions on the map. Lets take months example where months will be stored with a key like
```
+----------------+
| Key   |  Value |
+----------------+
| 1     | Jan    |
| 2     | Feb    |
| .     |  .     |
+----------------+
```

> NOTE:
> * One reason that we can’t take the address of a map element is that growing a map might cause rehashing of existing elements into new storage locations, thus potentially invalidating the address.

#### Example:

```
// maps in go
package main

import "fmt"

func main() {
	// initialize a map
	months := map[int]string{
		1:  "Jan",
		2:  "Feb",
		3:  "March",
		4:  "April",
		5:  "May",
		6:  "June",
		7:  "July",
		8:  "August",
		9:  "Sept",
		10: "Octo",
		11: "Nov",
		12: "Dec",
	}
	fmt.Println(months)
	// retrieve months using a key.
	for i := 1; i <= len(months); i++ {
		fmt.Println(i, months[i])
	}
	// operations on map
	// deleting last 4 months from the map
	for i := 12; i >= 9; i-- {
		delete(months, i)
	}
	// printing map after deletion
	fmt.Println(months)
	// adding deleted months back to map
	addMonths := []string{"Sept", "Oct", "Nov", "Dec"}
	for i := 9; i <= 12; i++ {
		for j, _ := range addMonths {
			months[i] = addMonths[j]
		}
	}
	fmt.Println(months)
}

```

<a href='https://play.golang.org/p/zuMMi-P8it3'>Example on Map: Go Playground </a>

## Struct

map data structure is used to store key value pairs with some limitations like all key data type must be identical and all value data type as well. What if a data is a collection of all different types. For example a person's data would be name, age, address, contact etc. This data have string, int data types. If program wants to process such kind of data then there should be a data structure which can structure this kind of data.

So here struct can be used. `struct` is am aggregate data structure that gathers all type of data and refereces through a single entity.

```
Struct   |   Different Data Types
---------------------------------
Person <=| Name (string)
         |   Age (int)
         |   Address (string)
         |   contact No (int)
         |   contact email (string)
```
Now Person (struct) stores all data collectively and an object can be created from Person. For example Robert is an employee which can be an instance of this struct.Data can be feeded for Robert by dot notation.
```
var Robert Person
Robert.Name = Robert
Robert.Age = 30
Robert.Address = abc street
Robert.ContactNo = 1232341313
Robert.ContactEmail = email@mail.com
```
#### Example:
```
// Using struct
package main

import (
	"fmt"
)

type Employee struct {
	Name    string
	Id      int
	Age     int
	Contact int64
	Address string
}

func main() {
	// create an object of Employee
	var EmployeeOne Employee

	// feed data for EmployeeOne
	EmployeeOne = Employee{
		Name:    "Robert",
		Id:      555,
		Age:     30,
		Contact: 2341231345,
		Address: "4th Avenue, California",
	}

	// Feed data for EmployeeTwo
	var EmployeeTwo Employee
	EmployeeTwo = Employee{
		Name:    "Downy",
		Id:      777,
		Age:     31,
		Contact: 21231233,
		Address: "5th Avenue, California",
	}
	// printing data for EmployeeOne

	fmt.Println(EmployeeOne)

	// printing data for EmployeeTwo

	fmt.Println(EmployeeTwo)
}
```
<a href='https://play.golang.org/p/T-hv1JBbdZl'>Example on Struct: Go Playground </a>
