---
title: "Learning Golang: Milestone 3"
layout: post
date: 2019-07-06
image: /assets/images/markdown.jpg
headerImage: false
tag:
- go
- golang
- go functions
category: blog
author: Milind
description: Learning Golang
---
## Functions
If you find a big code with thousands of lines of code, and you observe that few lines of code is getting  repeated across the whole program. Does that make sense ? This might be working but underneath it is using resources during runtime. So code might work but it may kill the performance of the application.

Repeatation of the code can be handled by using a `function`. Function is kind of a snippet of a program which performs some actions and returns the output. Once repeated code is written as a function then this function can be invoked at any point in our program. Function is a building block for a program.

## Function Declaration: Simple
```go
func FunctionName () {
  code statements
  ...
  ...
}
```
This is a simple function which neither takes any input from outside nor return any value for outside. When this function is called , control flow will be passed to this function and instruction will be executed within this function. For example
```go
package main
import "fmt"

func FunctionName () {   // declaring a function with code statements
  code statements
  ...
  ...
}                         // end function
...               // some code here
...

FunctionName()      // calling function to be executed.
}
```
## Function Declaration: Passing Arguments But No Return Value
```go
func FunctionName(var-list type) {
  statements
  ...
}
```
Sometimes, some data is required to be passed to the function as an argument. Once this data is available inside the function, then statements within function process on that data and gives output from function itself.

Example:
```go
package main
import "fmt"

func FunctionName (num int) {   // passing int type of argyment.
  code statements
  ...
  ...
}                         // end function
...               // some code here
...

FunctionName(5)      // invoking function by passing an argument 5 of type int  
}
```

## Function Declaration: Passing Arguments And Return Value
```go
func FunctionName(var_list type) (return_type) {
  code statements
  ...
  ...
}
```
When arguments are passed to the function, function will provide some output, now this output is necessary to be used in the upcoming program. In that case function is imposed to return a value.

Example:
```go
package main
import "fmt"

func FunctionName (num int) int { // passing int type of argyment and returning int type of data
  code statements
  ...
  ...
  return int_data
}                         // end function
...               // some code here
...

retVal := FunctionName(5)    // invoking function by passing an argument 5 of type int
                            // retVal stores the value returned by the function.
}

```

Program 1:

Program which finds the vowels and its count in the provided name by program arguments.
```go
/*
Program to find vowels and return the number of appearance from the provided string.
Program outline:
=> takes an arguments from the program.
  -> check only one argument is provided, error out if 2 arguments are provided
=> pass string to a function
  -> take first letter from the string and match with vowels
  -> if match found, store that chatacter in a slice increase the counter else move on to second character
  -> return slice and count to main function
=> display the result.
*/
package main

import (
	"fmt"
	"os"
)

// VowelsFinder finds appeared vowels,return them and their count
func VowelsFinder(word string ) (foundVowels []rune, count int) {
  fmt.Println("Your provided word is", word)
  // make an array of all vowels
  EngVowels := [5]rune{'a','e','i','o','u'}
  // take first letter from the word and compare it any one from EngVowels.
  count  = 0
  for i,_ := range word {
    for j, _ := range EngVowels {
      if rune(word[i]) == EngVowels[j] {
        foundVowels = append(foundVowels, rune(word[i]))
        count ++
      }
    }
  }
  return foundVowels,count
}

func main() {
	// getting argument from the user
	wordTaken := os.Args
	// check the length of the arguments
  if len(wordTaken) == 1 {
		fmt.Println("Provide a word to find the vowels")
		os.Exit(1)
	}
	if len(wordTaken) > 2 {
		fmt.Println("Only single argument is allowed")
		os.Exit(1)
	}

	// checks done: pass string to function
  Vowels, NumOfVowels :=	VowelsFinder(wordTaken[1])
  fmt.Println("Total number of vowels apreared in this word is", NumOfVowels)
  fmt.Printf("and that/those is/are[%s]\n", string(Vowels))
}
```
<a href='https://github.com/dmilind/golang/blob/master/codingFun/codefun01.go'><b>Find GitHub Code Here</b></a>

## Veriadic Functions
Usually function takes arguments and those arguments has to be declared, for that we should know how many arguments are being passed to a function. But what if we dont know the number of arguments beforehand or we are passing multiple arguments every time to the program. Here function will panic. To handle this behavior, veriadic function can be used.

In veriadic function number of arguments of same type can be passed.

Syntax:
```go
func VeriadicFunction (num ...int) int {
  .. function body   
}
```
In above function ```...``` is an ellipses which tells the function that number of arguments of ```int``` type can be passed and all those can be referred by ```num``` variable.
Lets take an example:
```go
package main

import (
  "fmt"

)
// function Add takes all passed arguments and calculates the addition of all of them. Also return the result
func Add(num ...int) int {
  Sum := 0
  for _, val := range num {
    Sum += val
  }
  return Sum
}

func main () {
  result := Add(1,1,2,2,3,4) // You can pass any num of arguments
  fmt.Println("Total =", result)    
}  

// output :
Total = 13
```
<a href='https://play.golang.org/p/K_Cc1lzoc5S'><b>Veriadic Function: Go Playground </b></a>

## Defer

## Panic And Recover
