---
title: A story about arrays and slices
date: 2023-09-27 18:33:10 -0
categories: [go,blog]
tags: [go,slice,array,blog]
---
## Intro
I wrote this piece quite a time ago when I started to learn go. Now its time has come to see the light.

## How to learn a new language
In general I see two kind of method to learn a programming language. You can go on a structured road (read a book, follow a course) or you can go on an unstructured way and explore/experiment on your own.  Probably you should use them parallel and let them complete each other. 
To continue learning go-lang now I will make a baby step on the latter path.  I chose an easy problem from HackerRank which we will solve together.

## The problem
Given an array “arr” and an integer “d”, rotate the elements of “arr” left by “d” steps and return the result. Easy enough problem to let us focus on the features of go and not on the algorithmic complexity. On HackerRank you can switch the code to go and you will receive this function to implement:

```go
func rotateLeft(d int32, arr []int32) []int32 {
 // TODO implement
}
```

Let see what we have here. `d int32` What is int32? As it turns out in go you have multiple integer types which all denoted by the memory size they can occupy: int8, int16, int32 and int64.

We have `arr []int32` so `[]int32` must be the notation for an array, right? (No.)

## The hardcoded solution 
I don’t want to deal with testing frameworks in go right now, so let’s start with a simple “test” before we code the solution:

```go
func main() {
        fmt.Println("exp: ", []int32{2, 3, 4, 5, 1})
        fmt.Println("act: ", rotateLeft(1, []int32{1, 2, 3, 4, 5}))
}
```

We print the expected output in the first line and print the actual result in the second line. How can we implement the rotation function in a test driven way? Of course by hard coding the response! Let’s see:

```go
func rotateLeft(d int32, arr []int32) []int32 {
   return []int32{2,3,4,5,1}
}
```

Let’s run this program, and the output is:

```go
exp:  [2 3 4 5 1]
act:  [2 3 4 5 1]
```

So far so good.  Our test passes and also you might think you already learnt how to instantiate an array in go! (You don’t.) 

# The algorithm
The next step in TDD is to remove duplication. For that we solve the problem of left rotation. If you are here to learn about go and not interested in the algorithm itself, feel free to jump to the next section.
It’s always a useful step to solve a simplified version first, so check the  `d = 1` case. We are rotating left by 1 step. Let’s populate a response array and as we populate it step by step always take the element in the next slot from the input array. Except for the last one since there is no next element after the last slot. We have to take the first element of the input array in the last slot. In pseudo code:

```go
var arrLength = the length of arr
var newArray = new array with arrLength
for i between 0 and arrLength // the upper bound is exlusive, so the last iteration is i = arrLength-1
  if(i == arrLength -1) newArray[i] = arr[0] // the last element in the response will be the first in the original
  else newArray[i] = arr[i+1] // every other element is copied from the next slot
```

How can we extend this logic to `d > 1` cases? As we reach forward `d` steps to grab a value in the input array at some point we will reach over the length of the input array. In that case we want to land on the beginning of the array. Just like a conveyor belt. As we reach the end magically we re-appear in the beginning. Whenever you face such a conveyor belt like issue the modulus operator is for the rescue. Taking the reminder by the length of the `arr` will just to do the trick and as you reach over the edge you will land on the beginning (e.g. the length is 9, 14%9 = 5, you reached over by 5 steps and you land on 5). In pseudo code:

```go
var arrLength = the length of arr
var newArray = new array with size of arr
for i between 0 and arrLength
  newArray[i] = arr[(i+d)%(arrLength)]
```

## Arrays and slices in Go
Let’s put this into practice in go:

```go
func rotateLeft(d int32, arr []int32) []int32 {
        var length = len(arr)
		  var response = [length]int32
        for i := 0; i < length; i++ {
                response[i] = arr[(i+d)%length]
        }
        return response
}
```

`len` is a built in function which will give the length of `arr`. In the second line we initialise an array with the required size. However if we try to compile this code we will receive a few error. The first one:
```bash
./main.go:14:20: non-constant array bound length
```

Hmm. That sounds bad. As it turns out arrays have to have compile time constant length. Which basically means you have to know the length when you write the code. But how the hell I should know the size when I write code if I only receive an `arr` when the program runs? Tough. In our simple test case we know the length so we might just go for it and use a constant length:

```go
var response = [5]int32
```

Let’s run it! Okay, at least the error message changed:
```go
./main.go:14:20: type [5]int32 is not an expression
```

Looks like `[5]int32` is not an expression but a type? Sure, then let’s use it as a type. Maybe it will work:

```go
var response [5]int32
```

Hurray, the error disappeared! But we have a new one (sometimes that’s a relief in itself, just let’s you google something else than you googled already 10 times):

```go
./main.go:18:9: cannot use response (type [5]int32) as type []int32 in return argument
```

What? How `type [5]int32` is not compatible with `type []int32`? If something can have whatever size why I can’t return it with the size of 5? 5 fits into the whatever, right? Wait a minute. Arrays have to have fix size by compile time. The type `[]int32` can have whatever size. Then `[]int32` cannot be an array! If arrays have fix size and the size is part of the type than something with a flexible size cannot be an array! In that case what is `[]int32`? **It’s a slice.**

## What is a slice?
Slice as it turns out a “pointer” to an array (I put it in quote before the computer science ninjas hunt me down). So there is an actual array somewhere stored  and we have an objects which references a part of that array. The object which describes exactly how the slice should behave (where does it start, where does it end, does it like Terry Pratchett books etc.) is called slice header. According to the go blog ([Arrays, slices (and strings): The mechanics of ‘append’ - The Go Programming Language](https://go.dev/blog/slices)) we can think about the slice header something like this:
```go
type slice header struct {
Length int // how long is the part we are referencing right now
Capacity int // it is at least the length but the original array can be longer than the part we are referencing now so there might be place to grow
ZerothElement *byte // points to the first element of this slice
}
```

This structure has serious consequences about how slices work in practice but now we want to solve our left rotate problem. We will deal with theory later. 
So let’s turn our response into a slice:
```go
var response []int32
```

At this point our program still doesn’t compile:

```go
./main.go:16:21: invalid operation: i + d (mismatched types int and int32)
```

Luckily, go has 4 different types for integer types so now we have to make sure they all match when we use them. I think the easiest thing to do is to convert `d` into an integer from `int32`. Our implementation will look like this:
```go
func rotateLeft(d int32, arr []int32) []int32 {
	var dInteger = int(d)
	var arrLength = len(arr)
	var response []int32
	for i := 0; i < arrLength; i++ {
		response[i] = arr[(i+dInteger)%arrLength]
	}
	return response
}
```

Try to run it. Drum roll, and … It compiled! It didn’t work but don’t be too greedy. Have to celebrate every little success:
```go
panic: runtime error: index out of range [0] with length 0
```

We got every type correct and our little child made their first steps in the fearsome runtime! It stumbled over the first tiny grain of sand but we will fix it. When we created our first slice with `var response []int32]`, go initialised it to the zero value of that type. By the way, stop here for a second to appreciate how cool is that! Did you notice we did not receive a `NullpointerException` nor a `TypeError` since go automatically initialise every declared variable. Adieu `NullpointerException`! (Just for the record: go is not completely free of NPE like problems) However the initialisation created a slice with zero length so when we tried to populate the slots, we failed. So we need to extend the slice. After a quick googling we can easily find the magic world: `append`. We can extend any slice with it:
```go
	for i := 0; i < arrLength; i++ {
		response = append(response, arr[(i+dInt)%length])
	}
```

And voilá, the output is:
```
exp:  [2 3 4 5 1]
act:  [2 3 4 5 1]
```

## Performance
I am happy we solved the problem but I have a lurking feeling that the solution is horribly expensive. I don’t know too much about the inner working of go (yet) but this solution cannot be really good. Remember the error message we received above: out of range. So we start a zero length array. Then comes `append` which will produce a new array with the length of one. In the next iteration a length of two. Then length of three and so on. These arrays cannot be the same. Maybe there is hidden magic below the shiny surface but my gut tells me `append` will copy the elements into a new, longer array and will use that new array to store the merged items. Copying elements over and over in every iteration means awful performance. How can we avoid that? We could use a single array in the background if we could initialise it immediately to the right size. Luckily you can create a slice with the specified length and capacity with the `make` built-in function:

```go
response := make([]int32 /* type */, len(arr) /* length */)
```

After this we won’t need the `append` since we could access the slots in the response by index:
```go
response[i] = arr[(i+dInt)%length]
```

Will this solution be better than the previous? Sounds reasonable but unfortunately not all reasonable thing is true. So let’s try to measure it. Performance measurement is it’s own art but we can start with something simple:

```go
package main

import (
	"log"
	"time"
)

func main() {
	start := time.Now()
	rotateLeftWithMake(1, make([]int32, 100000))
	log.Printf("Time elapsed with make: %s", time.Since(start))
	start = time.Now()
	rotateLeftWithAppend(1, make([]int32, 100000))
	log.Printf("Time elapsed with append: %s", time.Since(start))
}

func rotateLeftWithMake(d int32, arr []int32) []int32 {
	var dInt = int(d)
	var length = len(arr)
	response := make([]int32, len(arr))
	for i := 0; i < length; i++ {
		response[i] = arr[(i+dInt)%length]
	}
	return response
}

func rotateLeftWithAppend(d int32, arr []int32) []int32 {
	var dInt = int(d)
	var length = len(arr)
	var response []int32
	for i := 0; i < length; i++ {
		response = append(response, arr[(i+dInt)%length])
	}
	return response
}
```

The output:
```bash
2022/02/26 15:51:34 Time elapsed 528.11µs
2022/02/26 15:51:34 Time elapsed 2 1.586403ms

2022/02/26 15:51:53 Time elapsed with make: 533.785µs
2022/02/26 15:51:53 Time elapsed with append: 2.154747ms

2022/02/26 15:54:15 Time elapsed with make: 411.443µs
2022/02/26 15:54:15 Time elapsed with append: 1.55419ms

2022/02/26 15:59:08 Time elapsed with make: 401.264µs
2022/02/26 15:59:08 Time elapsed with append: 1.241734ms
```

As you can see, the solution with `make` is faster. I wouldn’t dare to say how much faster it is, since there is a whole lot more factor which we could incorporate in our performance test. E.g.: Is the result different if `rotateLeftWithAppend` runs first? And if the length of the input is 10? Or a million? What if the functions run multiple times not just once? Is there a compiler optimisation in go? I don’t know but I let you discover these questions. (Or maybe will return to this in a later episode)

## Summary
Hope you enjoyed this small journey with me as we discover the world of go-lang together. Here are the main points (according to my impartial opinion) from this article:

### Meta thoughts
* (In TDD) Always start with the test no matter how simple it is.
* Give a simple solution first which you can fine tune later.
* Solve algorithms for special cases first (if you have to) and generalise later.

### Go lang learnings
* Integers has multiple types based on the capacity
* Go initialise all the declared variables to the zero value of it’s type
* Arrays are useless for practical applications since the length is part of the type (e.g. `[15]int`)
* You should use slices instead which:
  * Points to an array behind the scenes
  * Can have variable length
* Although slice hides the actual array in the background, you are not free from the implications 

See you on the next trip!

### Further resources:
* [Arrays, slices (and strings): The mechanics of ‘append’ - The Go Programming Language](https://go.dev/blog/slices)
* [Slice tricks](https://github.com/golang/go/wiki/SliceTricks)
