---
title: "Learning Golang: Milestone 1"
layout: post
date: 2019-06-28
image: /assets/images/markdown.jpg
headerImage: false
tag:
- go
- golang
category: blog
author: Milind
description: Learning Golang
---

## Objective:

Will start with what , when, where, how ?
I always starts my study with these questions. We will also start to understand the basics of golang by using these phrases.

## What is GoLang? :

GoLang is a computer program developed at google. Companies like google have very diverse set of engineering problems, to tackle those google engineers always work towards it. These was a time when some developers were working on some product written in C++, when they used to build the code, it used to take hours for compilation. Because of that developers had to wait for a long time. To address this issue, google engineers developed golang which is fast, flexible and concurrent language. Developers wrote go tool set, compiler and libraries.

Golang is not an object oriented language but it can partially comes under that category. It can be used as a compiled language or interpreted language.
Golang inherits the features form other languages like, C, ALGOL, PASCAL etc. This would be higher level which is not actually needed. So its better to summarize this portion now.

<li> Golang comes under compiled as well as interpreted language category </li>
<li> Golang is not a object oriented programming , but partial features can be derived. </li>

## When GoLang got developed ?

Go was designed in 2007 , but it got open sourced in 2016.

## Where GoLang can be implemented ?

Well, go can be implemented heavily anywhere now. It can be used as a scripting language also, or it can be used to develop web server application also. If you are familiar with Docker, kubernetes , gogs, all these big application are written in Go. So scope of this language is vast.

## How to write a go code ?

Now we will start digging deep into Go. Lets start learning it slowly. Well, user of Go can be called as a gopher.
Gophers ! Lets start a journey towards GoLang. Also If I missed anything or any suggestion please comment on the blogs.
To understand any language, grammar is always important. Grammar should be understood for learning any language.

## Go Basics:

There are two main things to understand here. First is package and second is main. Consider that you have written a code which says Hello to everyone. So you will write a simple code for it like, 

```
package main
 import “fmt”

 func main(){     
  fmt.Println(“Hello Person”)
}
```

As a programmer your tasks is to greet a person with this code. If a person comes then this code can be executed. If another person comes then again you will have to call this code by putting his name. Now this would be a bad idea to write whole lot of code which is doing the same thing. To address this issue we can use the concept of code reusability. That means single file of code can be called. So this file is called as a package. Package can be a collection of code files. Whenever you want to use same code to be executed, you can use the package written earlier. In order to use that package you will have to import that package. Once the package is imported, you can use the functions written in that package. 

## Go fundamentals:

Statements and Expressions: Every program is a collection of statements and expressions. Statement means instruction given to computer through program while expression is a code which computes the values during program execution. Expression always returns a value which will be used in later program.

## Go Doc:

go language is heavily and nicely documented. Once you install go on your machine. Go tools will get installed and go doc utility is ready to be used. If you want to get go documentation on local browser then just simple run `godoc` command. This command starts web server and documentations will be loaded on port 6060 by default. You can hit the `http://localhost:6060` and there you go. If you are working on terminal and want to see the documentation about specific go component then hit the command `go doc component/package name` e.g `go doc fmt` command gives all information about fat package. You can also dig deeper into the package. 

## Program Structure:

Before starting our journey last piece is to know how go code is structured and where it will be stored. When go is installed on the system, go sets its path with GOPATH variable. Under this structure you can create your workspace. As per the best practices, three directories should be created, pkg, src, and bin. All go code will be under src directory.

Go code can be structured as below.

```
package main

import (
  "package_name"
  )

var (
  variable_name variable_type
  )

func main(){
  code statements
}
```

## Variables And Basic Data Types

### Variables

Go is a strongly typed language, that means everything should be declared in go code. Likewise variables in go are statically type. Each variable is declared and type is predefined for that variable. Once variable is declared with type, same variable can not be assigned to other type. Variables can be declared in below format.

```
var name type // variable is initialized with default value.
var (
  var1 type
  var2 type
  []var 3 type
  ) // this block defines multiple variables
var1 := value // this is short declaration which initializes a value to variable.
var var1, var2, var3 type // multiple variable declaration with same data type.
var var1, var2, var3 int = 10,12,13 // tuple assignment.
var name string = "abcd"  // assigned a variable name with a literal string value.
```
Another way to create a variable is by using a built-in function `New()`. For example

```
variable_name := new(type)
num := new(int)  
```

When new() function is called with int as a parameter, `num` variable will be initialized with 0 value. But remember the value return by new() would be the address of variable. So `num` would contains the address of variable not a variable value.

### Scope

Once variable is declared in go, you can call this reference this variable to any where in the file depending on the scopt of the variable. Scope means where you can reference the variable. If variable is declared in the main file , not within any block then that variable can be accessed across the file, that means variable is having file scope. If variable is declared within a block then variable got block scope and that variable can not be called outside the block.

Scope of the variable and lifetime of the variable both are different concepts. Scope of a declaration is a region in a code file. Scope is deduced during compile time. Lifetime of a variable means until what point the variable can be referenced. This is run time property. So lifetime is a run-time property while scope is a compile time property.

## Data Types:

Every program takes input and provides some output.What would be the input to the program ? It would be a data always. Data can be of any type, numbers, words, sentence, etc etc. To organize these different type of data, data type (as an entity) is used in programming. So data type defines what type of data would be processed in the program.

In Go , data types comes under four categories.
* Basic Data Types
  * int
  * bool
  * string
  * float64
* Aggregate Data Types
  * array
  * struct
* Reference Data Types  
  * pointer
  * slice
  * map
  * function
  * channel
* Interface Data Type

Once variable is assigned with a specific data type, that variable can not be assigned to different data type. Since go is a strongly typed language, data type conversion should be done before.

## Type Conversion:

As we know that go is a strongly typed language. Once a variable is declared, then this can be assigned to different type of variable. This causes an compilation error. For example
```
var a int = 5         // type is int
vat b float64 = 2.5      //type is float64

c := a + b   // this is a compilation error.
```
But in programming you will get encountered where such things are necessary. In go , this can be implement using type conversion. Type will be converted to desired type in order to do the process.

Syntax:
```
type(variable)
```
Now implementing above example with type conversion.
```
c := float64(a) + b // result would be 7.5
// if we convert b into int then result would be 7 which is not correct.
```
So type conversion can be destructive also. This should be used carefully.

---

                                                          lets meet in next blog now . . .
