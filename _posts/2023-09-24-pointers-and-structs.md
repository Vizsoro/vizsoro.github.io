---
title: Playing with structs and pointers
date: 2023-09-24 11:33:10 -0
categories: [puzzle,go]
tags: [go,puzzle,struct,pointer]
---

Today let's start with playing with this program:
```go
package main

import "fmt"

type student struct {
    name string
}

func (s student) update() {
    s.name = "unknown"
}

func main() {
    s := student{"J. Doe"}
    s.update()
    fmt.Println(s.name)
}
```

## Puzzle 1
What does this print when executed?

1. "J. Doe"
2. "unknown"
3. Nothing, this program has a compile error.
4. Nothing, this program has a runtime error.

<details >
<summary>Solution</summary>
1
</details>

<details markdown="1">
<summary>Explanation</summary>

The function receiver of `update` has the type of `student` and also `s` has the of type of `student` (not a pointer value). When the `update` function is executed, it uses the copy of the value it was invoked on. So it modifies the copy and original value remains unchanged.
</details>


## Puzzle 2

What will the program print if we change the `update` function to `func (s *student) update()`?

1. "J. Doe"
2. "unknown"
3. Nothing, this program has a compile error.
4. Nothing, this program has a runtime program.

<details markdown="1">
<summary>Solution</summary>
2
</details>

<details  markdown="1">
<summary>Explanation</summary>

Maybe the answer was obvious but there is a subtle detail here. The `s` in line 24 is still has the `student` type and it is not a pointer. So why the program didn't fail with a compilation error, since we were using a non-pointer type where the function is expecting `*student` (a pointer type)? It's a built in mechanism in the Go compiler:
> If x is addressable and &x's method set contains m, x.m() is shorthand for (&x).m().

[Source](https://go.dev/ref/spec#Calls)

To translate the English to English: if the function is defined on the pointer type of the receiver and it is possible to convert the non-pointer receiver to a pointer receiver (with the `&` operator), the compiler will automatically do this for you.
</details>

## Puzzle 3

Let's go back to the original program and now change line 16 to this: `fmt.Println(&s.name)`. What will this print?

1. "J. Doe"
2. "unknown"
3. A memory address.
4. Nothing, this program has a compile error.

<details markdown="1">
<summary>Solution</summary>
3
</details>

<details markdown="1">
<summary>Explanation</summary>

My naive 2 cent when I started with Go was option 4, since the `&` converts `s` to a pointer type and only the non-pointer type `student` has the `update` method. I was wrong on many levels. 
The first thing here is operator precedence. The `.name` is executed first, than the `&` so this example will print the memory address of the `name` string.
</details>
