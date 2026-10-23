---
tags:
  - golang
  - memory
section: "03 Memory and Data Types"
video: https://www.youtube.com/watch?v=_0R6H1m9o78
video_title: "Maps in golang"
order: 16
---
# Maps

> [!info] Source
> 🎬 [Maps in golang](https://www.youtube.com/watch?v=_0R6H1m9o78) · Section **03 Memory and Data Types** of [[Golang Notes]]
> ⬅️ [[Slices]] · ➡️ [[Structs]]

## Overview

A **map** is Go's built-in hash table: a collection of **key-value pairs** where you look values up by key instead of by position. Keys can be strings, integers or any other comparable type, values can be anything at all, and lookups are fast because the map knows exactly where each key lives. The instructor's example is a tiny dictionary of programming-language abbreviations (`JS` → `Javascript`). The video also sneaks in a preview of the `for range` loop, which gets its proper treatment in [[Loops, Break, Continue and Goto]].

## Creating a map

The type of a map is written `map[KeyType]ValueType`. As with slices, you create one with **`make`** from [[Memory Management]]: `new` would only give you a pointer to a nil map that cannot hold values, whereas `make` allocates *and* initialises the hash table so you can write to it immediately.

```go
languages := make(map[string]string)   // keys: string, values: string
```

A map literal does the same job and lets you provide the initial pairs:

```go
languages := map[string]string{
	"JS": "Javascript",
	"RB": "Ruby",
	"PY": "Python",
}
```

> [!warning] A `var m map[string]string` is nil
> Declaring a map without `make` or a literal gives you a **nil map**. Reading from it returns zero values, but writing panics with `panic: assignment to entry in nil map`. This is the single most common map mistake in Go.

## Adding, reading and deleting

| Operation | Syntax | Notes |
|---|---|---|
| Add or overwrite | `languages["JS"] = "Javascript"` | Same syntax for insert and update |
| Read | `languages["JS"]` | Returns `"Javascript"`; a **missing key returns the zero value** (`""` here) |
| Read with existence check | `value, ok := languages["GO"]` | `ok` is `false` when the key is absent: the comma ok syntax |
| Delete | `delete(languages, "RB")` | Built-in function; deleting a missing key is a no-op |
| Count | `len(languages)` | Number of pairs |

```go
languages["JS"] = "Javascript"
languages["RB"] = "Ruby"
languages["PY"] = "Python"

fmt.Println("List of all languages:", languages)   // map[JS:Javascript PY:Python RB:Ruby]
fmt.Println("JS shorts for:", languages["JS"])      // Javascript

delete(languages, "RB")
fmt.Println("List of all languages:", languages)   // map[JS:Javascript PY:Python]
```

Two details about the printed form: Go shows a map as `map[key:value key:value]` with **no commas**, the same space-separated style it uses for arrays and slices, and `fmt` prints the pairs **sorted by key** so the output is stable even though the map itself has no order.

### The comma ok syntax

Because a missing key silently returns the zero value, you cannot tell "the value is empty" apart from "the key does not exist" with a plain read. The two-value form fixes that, and it is the same **comma ok** pattern you met in [[User Input and Comma Ok Syntax]]:

```go
value, ok := languages["GO"]
if !ok {
	fmt.Println("GO is not in the map")
}
```

## Looping over a map

The instructor jumps ahead to loops here because iterating a map is so common. `for key, value := range languages` hands you each pair in turn, and the walrus operator `:=` declares both variables for the loop body:

```go
for key, value := range languages {
	fmt.Printf("For key %v, value is %v\n", key, value)
}
```

If you only need the values, replace the key with the blank identifier **`_`**, exactly as you do to ignore an error. Go refuses to compile an unused variable, so `_` is how you say "I know, I do not care about this one":

```go
for _, value := range languages {
	fmt.Printf("Value is %v\n", value)
}
```

> [!warning] Iteration order is random
> Go deliberately randomises the order in which `range` walks a map, so two runs of the same program can print the pairs in different orders. Never write code that depends on map order; if you need a stable order, sort the keys first (see the example below).

## Example: the instructor's maps program

The companion code for this video (`10mymaps/main.go`):

```go
package main

import "fmt"

func main() {
	fmt.Println("Maps in golang")

	languages := make(map[string]string)

	languages["JS"] = "Javascript"
	languages["RB"] = "Ruby"
	languages["PY"] = "Python"

	fmt.Println("List of all languages: ", languages)
	fmt.Println("JS shorts for: ", languages["JS"])

	delete(languages, "RB")
	fmt.Println("List of all languages: ", languages)

	// loops are interesting in golang
	for key, value := range languages {
		fmt.Printf("For key %v, value is %v\n", key, value)
	}

	// ignore the key with the blank identifier
	for _, value := range languages {
		fmt.Printf("Value is %v\n", value)
	}
}
```

**Output:** (the order of the loop lines can differ between runs)

```text
Maps in golang
List of all languages:  map[JS:Javascript PY:Python RB:Ruby]
JS shorts for:  Javascript
List of all languages:  map[JS:Javascript PY:Python]
For key JS, value is Javascript
For key PY, value is Python
Value is Javascript
Value is Python
```

## Example: existence checks and a stable order

Two things the video does not show but you will need almost immediately: distinguishing a missing key from an empty value, and iterating in a predictable order.

```go
package main

import (
	"fmt"
	"maps"
	"slices"
)

func main() {
	languages := map[string]string{"JS": "Javascript", "RB": "Ruby", "PY": "Python"}

	// comma ok: "missing" vs "present with a zero value"
	value, ok := languages["GO"]
	fmt.Printf("value=%q ok=%v\n", value, ok)

	if name, ok := languages["PY"]; ok {
		fmt.Println("PY is short for", name)
	}

	// deterministic iteration: sort the keys first
	for _, key := range slices.Sorted(maps.Keys(languages)) {
		fmt.Println(key, "=>", languages[key])
	}

	fmt.Println("len:", len(languages))
	clear(languages)
	fmt.Println("after clear:", languages, "| len:", len(languages))
}
```

**Output:**

```text
value="" ok=false
PY is short for Python
JS => Javascript
PY => Python
RB => Ruby
len: 3
after clear: map[] | len: 0
```

The `if name, ok := ...; ok` form declares the variables and tests them in one statement; that pattern is covered in [[If Else]].

## Gotchas and Best Practices

> [!warning] Keys must be comparable
> Strings, numbers, booleans, pointers and structs made of comparable fields all work as keys. Slices, maps and functions do not, because they cannot be compared with `==`. `map[[]string]int` is a compile error.

> [!note] Maps are reference-like
> Passing a map to a function does **not** copy its contents; the function modifies the same table. You rarely need a pointer to a map, unlike with [[Structs]].

> [!tip] Prefer a map when you look things up by key
> If you find yourself looping through a [[Slices|slice]] to find an element with a particular id or name, a map keyed by that id is usually the better structure. This becomes relevant in [[Get One Course Controller]], where the course list is searched by id.

> [!warning] Maps are not safe for concurrent writes
> Writing to a map from several goroutines at once crashes with `fatal error: concurrent map writes`. Guard it with a [[Mutex]] or use `sync.Map`.

## Modern Go Notes

- The built-in **`clear(m)`** (Go 1.21) removes every entry from a map.
- The **`maps`** package arrived in Go 1.21 with `maps.Clone`, `maps.Copy`, `maps.DeleteFunc`, `maps.Equal` and `maps.EqualFunc`. The iterator functions `maps.Keys` and `maps.Values` were added in Go 1.23 together with `slices.Sorted` and `slices.Collect`, which is why the example above pairs them (Go 1.23+).

## Video Timeline

| Time | What happens |
|---|---|
| [00:20](https://youtu.be/_0R6H1m9o78?t=20) | Maps are key-value pairs with fast retrieval |
| [01:43](https://youtu.be/_0R6H1m9o78?t=103) | Creating a map with `make(map[string]string)` and why not `new` |
| [02:36](https://youtu.be/_0R6H1m9o78?t=156) | Adding pairs and printing the whole map |
| [04:19](https://youtu.be/_0R6H1m9o78?t=259) | Reading one value by key |
| [04:54](https://youtu.be/_0R6H1m9o78?t=294) | Deleting a key with `delete` |
| [06:03](https://youtu.be/_0R6H1m9o78?t=363) | Looping with `for key, value := range`, then ignoring the key with `_` |

## Related Notes

- [[Slices]] — the other collection type; slices are ordered, maps are keyed.
- [[Memory Management]] — `make` vs `new`, and why a map needs `make`.
- [[User Input and Comma Ok Syntax]] — the comma ok pattern reused for map lookups.
- [[Loops, Break, Continue and Goto]] — the `for range` loop in full.
- [[If Else]] — the `if v, ok := m[k]; ok` form.
- [[Consuming JSON Data]] — decoding JSON into `map[string]interface{}` when the shape is unknown.
- [[Mutex]] — protecting a map that is shared between goroutines.
- [[Structs]] — next in the course: when fields have fixed names, use a struct instead of a map.
- [[Go Quick Reference]] — map syntax at a glance.
