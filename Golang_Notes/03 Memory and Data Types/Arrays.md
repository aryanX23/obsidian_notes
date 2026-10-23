---
tags:
  - golang
  - memory
section: "03 Memory and Data Types"
video: https://www.youtube.com/watch?v=JoUSa8jtadE
video_title: "Array in golang"
order: 13
---
# Arrays

> [!info] Source
> 🎬 [Array in golang](https://www.youtube.com/watch?v=JoUSa8jtadE) · Section **03 Memory and Data Types** of [[Golang Notes]]
> ⬅️ [[Pointers]] · ➡️ [[Slices]]

## Overview

In most languages the array is the most-used data structure. In Go it is **rarely used directly**, because the language specification "holds your hand": an array has a **fixed length that is part of its type**, it cannot grow, and there is no sorting or appending built in. Under the hood, though, arrays power the data type you *will* use everywhere, the slice (see [[Slices]]), so understanding arrays first makes slices obvious. This note covers how to declare an array, how zero values show up, what `len` really reports, and the short list of things you can do with one.

## Declaring an array

An array is written as `[N]T`: square brackets holding the **number of elements**, followed by the element type. The size is compulsory, and it is checked at compile time.

```go
var fruitList [4]string   // room for exactly 4 strings, all "" for now
```

Elements are accessed by index starting at **zero**, and you fill them with plain assignment:

```go
fruitList[0] = "Apple"
fruitList[1] = "Tomato"
fruitList[3] = "Peach"    // index 2 deliberately left empty
```

Printing the array shows all four slots. Look carefully at the output: there is a **double space** between `Tomato` and `Peach`. That is Go printing the empty string in slot 2, a small visual hint that your array has an unused element.

```text
Fruit list is:  [Apple Tomato  Peach]
```

### `len` reports the declared size, not the filled count

`len(fruitList)` is **4** even though only three values were stored. The length of an array is fixed by its declaration; Go does not count "how many did you actually put in". The instructor calls this the strangest thing about Go arrays, and it is the first reason people move to slices.

### Declaring and initialising in one line

You can provide the values in curly braces. Any slots you do not fill keep their zero value, and `len` still reports the declared size:

```go
var vegList = [5]string{"potato", "beans", "mushroom"}
fmt.Println(len(vegList))   // 5, not 3
```

> [!tip] Let the compiler count with `[...]`
> If you want the size to match the literal exactly, write `[...]T{...}` and Go counts the elements for you: `auto := [...]int{10, 20, 30, 40}` has type `[4]int`.

## Example: the instructor's array program

The companion code for this video (`08myarray/main.go`):

```go
package main

import "fmt"

func main() {
	fmt.Println("Welcome to array in golang")

	var fruitList [4]string

	fruitList[0] = "Apple"
	fruitList[1] = "Tomato"
	fruitList[3] = "Peach" // slot 2 stays ""

	fmt.Println("Fruit list is: ", fruitList)
	fmt.Println("Fruit list is: ", len(fruitList))

	var vegList = [5]string{"potato", "beans", "mushroom"}
	fmt.Println("Vegy list is: ", vegList)
	fmt.Println("Vegy list is: ", len(vegList))
}
```

**Output:**

```text
Welcome to array in golang
Fruit list is:  [Apple Tomato  Peach]
Fruit list is:  4
Vegy list is:  [potato beans mushroom  ]
Vegy list is:  5
```

Notice the two trailing spaces inside the vegetable list: two empty strings at the end.

## Arrays are values, and the size is part of the type

Two properties that the video only hints at but that explain Go's behaviour completely:

1. **Assigning an array copies it.** `copied := original` makes a full, independent copy (contrast this with [[Pointers]] and with slices, which share memory).
2. **`[3]string` and `[4]string` are different types.** You cannot assign one to the other or pass a `[4]string` to a function expecting `[3]string`.

```go
package main

import "fmt"

func main() {
	original := [3]string{"go", "rust", "zig"}
	copied := original // arrays are copied on assignment
	copied[0] = "python"

	fmt.Println("original:", original)
	fmt.Println("copied:  ", copied)

	// the size is part of the type
	fmt.Printf("%T vs %T\n", original, [4]string{})

	// let the compiler count the elements
	auto := [...]int{10, 20, 30, 40}
	fmt.Println("auto has", len(auto), "elements:", auto)
}
```

**Output:**

```text
original: [go rust zig]
copied:   [python rust zig]
[3]string vs [4]string
auto has 4 elements: [10 20 30 40]
```

The `%T` verb that prints a value's type is covered in [[fmt Package and Format Verbs]].

## Arrays vs slices at a glance

| | Array `[4]string` | Slice `[]string` |
|---|---|---|
| Length | Fixed at compile time, part of the type | Dynamic; grows with `append` |
| `len()` | Always the declared size | Number of elements currently held |
| Assignment | Copies all elements | Copies a small header; both share the data |
| Sorting, appending | Not built in | `append`, `sort`, `slices` package |
| Typical use in Go | Rare; fixed-size data such as a SHA-256 digest `[32]byte` | Almost everywhere |

The next note, [[Slices]], is where the course spends its real effort.

## Gotchas and Best Practices

> [!warning] Out-of-range indexes
> `fruitList[4]` on a `[4]string` is a **compile-time error** when the index is a constant, and a runtime `panic: index out of range [4] with length 4` when the index is a variable. The same panic appears with slices, see [[Slices]].

> [!note] Zero values fill the gaps
> An array of strings starts as all `""`, an array of ints as all `0`, an array of bools as all `false`. This is the same zero-value rule you met in [[Variables, Types and Constants]].

> [!tip] Reach for a slice by default
> Use an array only when the size is genuinely fixed and known up front. Everything else, including every list that comes from user input, a file or a database, is a slice.

## Video Timeline

| Time | What happens |
|---|---|
| [00:20](https://youtu.be/JoUSa8jtadE?t=20) | Arrays are rarely used directly in Go; the spec "holds your hand" |
| [01:49](https://youtu.be/JoUSa8jtadE?t=109) | Declaring `var fruitList [4]string`: the size is compulsory |
| [03:31](https://youtu.be/JoUSa8jtadE?t=211) | Running it: the double space reveals the empty slot |
| [04:09](https://youtu.be/JoUSa8jtadE?t=249) | `len()` returns 4 even with three values stored |
| [04:53](https://youtu.be/JoUSa8jtadE?t=293) | Declaring and filling `vegList` in one line |
| [06:19](https://youtu.be/JoUSa8jtadE?t=379) | No sorting, nothing fancy: that is what slices are for |

## Related Notes

- [[Slices]] — the dynamic, far more common data type built on top of arrays.
- [[Pointers]] — arrays copy on assignment; pointers are how you avoid the copy.
- [[Memory Management]] — fixed-size arrays are allocated once; `make` is for slices and maps.
- [[Variables, Types and Constants]] — zero values and the `var` declaration syntax.
- [[Loops, Break, Continue and Goto]] — iterating over arrays and slices with `for range`.
- [[fmt Package and Format Verbs]] — `%T` to print the array's type.
- [[Go Quick Reference]] — array syntax at a glance.
