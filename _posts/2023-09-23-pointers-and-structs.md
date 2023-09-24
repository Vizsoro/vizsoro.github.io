---
title: Playing with structs
date: 2023-09-22 19:33:10 -0
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
    fmt.Println(s)
}
```

## Puzzle 1
What does this print when executed?

1. "J. Doe"
2. "unknown"
3. Nothing, this program has a compile error.
4. Nothing, this program has a runtime program.

<details>
<summary>Solution</summary>
1
</details>

<details>
<summary>Explanation</summary>

The function receiver of `update` has the type of `student` (it is not a pointer value) and also `s` has the of `student`. When the `update`` function is executed, it uses the copy of the value it was invoked on. So it modifies the copy and original value remains unchanged.
</details>


## Puzzle 2

What will the program pint if we change the `update` function to `func (s *student) update()`?

1. "J. Doe"
2. "unknown"
3. Nothing, this program has a compile error.
4. Nothing, this program has a runtime program.

<details>
<summary>Solution</summary>
2
</details>

<details  markdown="1">
<summary>Explanation</summary>

Maybe the answer was obvious but there is a subtle detail here. The `s` in line 24 is still has the `student` type and it is not a pointer. So why the program didn't fail with a compilation error, since we were using a non-pointer type where the function is expecting `*student` (a pointer type)? It's a built in mechanism in the Go compiler:
> If x is addressable and &x's method set contains m, x.m() is shorthand for (&x).m().


</details>


```
func (s *student) update() {
    s = &student{"unknown", 0}
}

```

```
package main

func main() {
    // a simple variable
    role := "Admin"

    // get the memory address of the `role` variable
    roleMemAddr := &role

    // assign the value of `Employee` to the `role` variable
    // using the `roleMemAddr` pointer variable and dereferencing
    // it using the `*` symbol to assign the new value
    *roleMemAddr = "Employee"
}


package main

import "fmt"

func main() {
    // a simple variable
    role := "Admin"

    // get the memory address of the `role` variable
    roleMemAddr := &role

    // assign the value of `Employee` to the `role` variable
    // using the `roleMemAddr` pointer variable and dereferencing
    // it using the `*` symbol to assign the new value
    *roleMemAddr = "Employee"

    // log the value of `role` variable to console
    fmt.Println(role) // Employee ✅
}
```