---
title: Type alias in action
date: 2023-09-27 18:33:10 -0
categories: [go,puzzle]
tags: [go,type,alias,puzzle]
---
## Puzzle 1

What will this program print?

```go
package main

import "fmt"

func main() {
	type MySuperPower string

	var s1 MySuperPower = "flying"
	s2 := &s1
	*s2 = "invisibility"

	fmt.Printf("%T %v %T %v", s1, s1, s2, s2)
}
```

1. It will print "string invisibility string invisibility".
2. It will print "main.MySuperPower flying main.MySuperPower invisibility".
3. It will print "main.MySuperPower invisibility *main.MySuperPower <some hexa code>".
4. It will print "string flying *string invisibility".

<details markdown="1">
<summary>Solution</summary>

3
</details>

<details markdown="1">
<summary>Explanation</summary>

Since we are assinging the pointer address to `s2` it will have the pointer type of `s1`. Since the `MySuperPower` type is defined in the `main` package, `s1`'s type will be `main.MySuperPower` therefore `s2` will have the type of `*main.MySuperPower`. Also the default value format for pointers is their address value,
</details>

## Puzzle 2

How does this program behaves?

```go
package main

import "fmt"

func main() {
	type WarningMessage string
	
	message := "caution"
	var warning, warning2 WarningMessage
	warning = message
	warning2 = "caution"
	
	fmt.Printf("%s %s", warning, warning2)
}
```

1. It has a compile error in line 10 and 11.
2. It has a compile error in line 10.
3. It has a compile error in line 11.
4. It will print "caution caution".

<details markdown="1">
<summary>Solution</summary>

2.
</details>

<details markdown="1">
<summary>Explanation</summary>

Line 10 will yield a compile error: `cannot use message (variable of type string) as WarningMessage value in assignment`. That's fair, `string` is not `WarningMessage` strictly speaking but why line 11 works then?
In line 11, `"caution"` is not a string. It's an untyped consant with the `string` underlying (or default) type. You can assign an untyped constant to a variable of any type until the consant's underlying type is compatible with the variable's type.
</details>


## Puzzle 3

Given this code:

```go
type SmallNumber int8

var a int8 = 1
var b SmallNumber = 2
```

Which line of code will not print 2?

1. `fmt.Print(b / a)`
2. `fmt.Println(b / 1)`
3. `fmt.Println(b / SmallNumb`
4. `fmt.Println(int8(b) / a)`

<details markdown="1">
<summary>Solution</summary>

1.
</details>

<details markdown="1">
<summary>Explanation</summary>

Go very strict about types. You cannot mix them even if they are compatible. In this case `b / a` will yield a compile error: `invalid operation: b / a (mismatched types SmallNumber and int8)`. If you want to use them together you have to explicitly cast them like in the other examples.
</details>