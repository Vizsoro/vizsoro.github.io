---
title: Numeric types
date: 2023-09-26 11:33:10 -0
categories: [puzzle,go]
tags: [go,puzzle,types,numbers]
---
## Puzzle 1

1. How does this program behave?

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	for i := 0; i < math.MaxInt64-1; i++ {
		fmt.Println(i)
	}
}

```

1. It will never stop.
2. It will print the integers until MaxInt64-1 and quit.
3. It will depend on where it is executed.
4. It will panic because of memory overflow.

<details markdown="1">
<summary>Solution</summary>
3
</details>

<details markdown="1">
<summary>Explanation</summary>

With the `i := 0` declaration, `i` will have the type of `int`. `int` is at least 32 bits in size but it can be 64 bits, 
depending on the machine. More details e.g.: [here](https://stackoverflow.com/questions/21491488/what-is-the-difference-between-int-and-int64-in-go#comment32446127_21491667).
If `int` has 32 bits size it will overflow before reaching `math.MaxInt64-1` and the program will never stop. If `int` has 64 bits (or more), the program will print the numbers until the limit than it will quit.
</details>


## Puzzle 2 - Farenheit conversion

1. What will this program print?

```go
package main

import "fmt"

func main() {
	fmt.Printf("%f", convertFahrenheitToCelsius(41))
}

func convertFahrenheitToCelsius(f float64) float64 {
	return (f - 32) * (5 / 9)
}
```

1. 0
2. 5
3. It won't compile.
4. It won't print anything.

<details markdown="1">
<summary>Solution</summary>
1
</details>

<details markdown="1">
<summary>Explanation</summary>

This [Stackoverflow answer](https://stackoverflow.com/a/32815507) is pretty good but the just of it:
The `5/9` division will yield an integer answer (since both operands are integer) and that integer answer will be `0` in this case.
So this `convertFahrenheitToCelsius` function will yield `0` no matter the input. To fix this, we need to convert at least one of the operands
of the division to float. E.g. `5.0/9` and `5/9.0` are both good.
</details>


## Puzzle 3

What will this program print?

```go
package main

import "fmt"

func main() {
	a := 1
	b := 2.5
	c := a / b
	fmt.Printf("%f", c)
}
```

1. 0
2. 0.400000
3. It won't compile.
4. It won't print anything.

<details markdown="1">
<summary>Solution</summary>
3
</details>

<details markdown="1">
<summary>Explanation</summary>

Line 8 will yield the following compile error: `Invalid operation: a / b (mismatched types int and float64)`.
Unlike C, Go <b>does not have automatic conversion between numeric types</b>. It was a deliberate choice from the creators of Go:

>The convenience of automatic conversion between numeric types in C is outweighed by the confusion it causes. When is an expression unsigned? How big is the value? Does it overflow? Is the result portable, independent of the machine on which it executes? It also complicates the compiler;

[Source](https://go.dev/doc/faq#conversions)

To make this work we need explicit type conversion, e.g. `c := float64(a) / b`.
</details>


