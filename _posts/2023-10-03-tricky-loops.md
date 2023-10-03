---
title: Tricky loops
date: 2023-10-03 18:33:10 -0
categories: [go,puzzle]
tags: [go,loop,puzzle]
---
## Puzzle 1

What does this program print?
```go
package main

import "fmt"

func main() {
	var i *int
	for j := 0; j < 5; j++ {
		if j%2 == 0 {
			i = &j
		}
	}

	fmt.Print(*i)
}
```
1. It will print a hexadecimal memory address.
2. It will print 0.
3. It will print 4.
4. It will print 5.

<details markdown="1">
<summary>Solution</summary>

4.
</details>

<details markdown="1">
<summary>Explanation</summary>

The `j` variable is reused in all the iterations. Once it's memory address is written into `i` in line 9, `i` will always refer to `j`'s address and when we dereference the memory address in line 13 we will got `j`'s last value which is 5 in this case.
</details>


## Puzzle 2

How can we change the program above to print 4?

1. Replace line 6 with `var i int`
2. Replace line 9 with `i = j`
3. Replace line 9 with `k := j; i = &k`
4. Replace line 13 with `fmt.Print(&i)`

<details markdown="1">
<summary>Solution</summary>

3.
</details>

<details markdown="1">
<summary>Explanation</summary>

The `k := j` will allocate a new variable, thus new memory slot and will copy `j`'s value into it. Now it is "safe" to assign `k`'s memory address since the loop won't touch that memory slot unless the `if` condition is met.
</details>

## Puzzle 3

What does this program print?
```go
package main

import "fmt"

func main() {
	var p []func()
	for _, s := range []string{"a", "b", "c"} {
		appendPrint(p, s)
	}

	for _, f := range p {
		f()
	}
}

func appendPrint(p []func(), s string) {
	p = append(p, func() { fmt.Println(s) })
}

```
1. It will print nothing.
2. It will print "abc".
3. It will print "ccc".
4. It will print "979899".

<details markdown="1">
<summary>Solution</summary>

1.
</details>

<details markdown="1">
<summary>Explanation</summary>

In this case the tricky part lies with the slices. When we pass down `p` to the `appendPrint`, we pass down a copy of `p` from main and when we call `p = append(p, ...` we store the return value in the variable `p` local to the `appendPrint` function. The result of `append` never reach the `p` in the main function. When we reach line 11, the length of `p` is still 0.
</details>
