---
tags:
  - golang
  - hub
section: "Hub"
---
# fmt Package and Format Verbs

> [!info] Hub note
> `fmt` is the first package every Go program imports and it appears in every single video of the series, from [[Installation and Hello World]] onwards. This note collects everything the series uses from it in one place. Part of [[Golang Notes]].

## Overview

The **`fmt`** package (pronounced "fumpt" by the Go team, "format" by everyone else) implements formatted input and output. It is Go's equivalent of C's `printf` family, but it understands Go values natively: it can print a **struct** (see [[Structs]]), a **slice** (see [[Slices]]), a **map** (see [[Maps]]) or a **pointer** (see [[Pointers]]) without you writing any conversion code. Learning its handful of **verbs** (`%v`, `%T`, `%d`, `%s` …) early pays off for the whole series, because the instructor uses `fmt.Printf("%T", x)` constantly to reveal what type a value really has.

## The print family

The functions come in three groups that differ only in **where the text goes**:

| Goes to | Plain | With newline + spaces | With format string |
|---|---|---|---|
| Standard output | `fmt.Print` | `fmt.Println` | `fmt.Printf` |
| A `string` (returned) | `fmt.Sprint` | `fmt.Sprintln` | `fmt.Sprintf` |
| Any `io.Writer` (file, HTTP response, stderr) | `fmt.Fprint` | `fmt.Fprintln` | `fmt.Fprintf` |
| An `error` value | | | `fmt.Errorf` |

- **`Println`** writes its arguments separated by spaces and ends with a newline. This is the workhorse for quick debugging.
- **`Print`** inserts spaces only between operands when *neither* is a string, and adds no newline.
- **`Printf`** takes a **format string** with verbs and prints exactly what you describe. It does **not** add a newline, so you end the format with `\n` yourself.
- **`Sprintf`** builds a string instead of printing it. Use it whenever you need to compose text, for example an error message or a log line.
- **`Fprintf`** writes to any `io.Writer`. In the API section the `http.ResponseWriter` `w` is an `io.Writer`, so `fmt.Fprintf(w, "hello")` is a valid way to answer a request (the series mostly uses `w.Write([]byte(...))` and `json.NewEncoder(w)` instead, see [[Get All Courses Controller]]).
- **`Errorf`** formats and returns an `error`; with the `%w` verb it *wraps* another error (see [[Error Handling in Go]]).

## The verbs you will actually use

| Verb | Meaning | Example output |
|---|---|---|
| `%v` | default format of any value | `{Hitesh hitesh@go.dev 16}` |
| `%+v` | like `%v` but structs print **field names** | `{Name:Hitesh Email:hitesh@go.dev Age:16}` |
| `%#v` | Go-syntax representation (what you would type in source) | `main.User{Name:"Hitesh", Email:"hitesh@go.dev", Age:16}` |
| `%T` | the **type** of the value | `main.User`, `[]string`, `*int` |
| `%d` | integer, base 10 | `42` |
| `%b` `%o` `%x` `%X` | integer in binary / octal / hex | `101010`, `52`, `2a`, `2A` |
| `%c` | the character for a rune | `G` |
| `%f` `%.2f` | float, optionally with precision | `3.141590`, `3.14` |
| `%e` | scientific notation | `1.234567e+06` |
| `%s` | string (or `[]byte`) | `go` |
| `%q` | double-quoted, escaped string | `"go"` |
| `%t` | boolean | `true` |
| `%p` | pointer address | `0xc000012345` |
| `%%` | a literal percent sign | `%` |

**Width and flags** go between `%` and the verb: `%5d` pads to width 5 on the left, `%-5d` pads on the right, `%05d` pads with zeros, `%10s` right-aligns a string in 10 columns.

> [!tip] The instructor's favourite
> `fmt.Printf("Variable is of type: %T \n", x)` is used in [[Variables, Types and Constants]], [[User Input and Comma Ok Syntax]], [[Slices]] and many other videos. Whenever Go's type inference surprises you, print `%T`.

## Example: every verb in one program

```go
package main

import (
	"fmt"
	"os"
)

type User struct {
	Name  string
	Email string
	Age   int
}

func main() {
	u := User{"Hitesh", "hitesh@go.dev", 16}
	nums := []int{1, 2, 3}
	langs := map[string]string{"JS": "JavaScript", "PY": "Python"}

	fmt.Println("Println adds spaces and a newline:", u.Name, u.Age)
	fmt.Print("Print adds nothing")
	fmt.Print("\n")

	fmt.Printf("%%v  -> %v\n", u)
	fmt.Printf("%%+v -> %+v\n", u)
	fmt.Printf("%%#v -> %#v\n", u)
	fmt.Printf("%%T  -> %T\n", u)
	fmt.Printf("%%v on a slice -> %v, on a map -> %v\n", nums, langs)

	fmt.Printf("%%d -> %d | %%5d -> |%5d| | %%-5d -> |%-5d| | %%05d -> %05d\n", 42, 42, 42, 42)
	fmt.Printf("%%f -> %f | %%.2f -> %.2f | %%e -> %e\n", 3.14159, 3.14159, 1234567.0)
	fmt.Printf("%%s -> %s | %%q -> %q | %%10s -> |%10s|\n", "go", "go", "go")
	fmt.Printf("%%t -> %t | %%c -> %c | %%x -> %x | %%b -> %b | %%%% -> %%\n", true, 'G', 255, 5)

	s := fmt.Sprintf("%s is %d years old", u.Name, u.Age)
	fmt.Println(s)

	fmt.Fprintln(os.Stderr, "this line goes to stderr")

	err := fmt.Errorf("user %q not found", u.Name)
	fmt.Println(err)
}
```

**Output:**

```text
Println adds spaces and a newline: Hitesh 16
Print adds nothing
%v  -> {Hitesh hitesh@go.dev 16}
%+v -> {Name:Hitesh Email:hitesh@go.dev Age:16}
%#v -> main.User{Name:"Hitesh", Email:"hitesh@go.dev", Age:16}
%T  -> main.User
%v on a slice -> [1 2 3], on a map -> map[JS:JavaScript PY:Python]
%d -> 42 | %5d -> |   42| | %-5d -> |42   | | %05d -> 00042
%f -> 3.141590 | %.2f -> 3.14 | %e -> 1.234567e+06
%s -> go | %q -> "go" | %10s -> |        go|
%t -> true | %c -> G | %x -> ff | %b -> 101 | %% -> %
Hitesh is 16 years old
this line goes to stderr
user "Hitesh" not found
```

Notice that maps print with their keys **sorted** (`JS` before `PY`); `fmt` sorts map keys so that output is deterministic even though map iteration order is random (see [[Maps]]).

## Reading input with fmt

`fmt` can also *read*: `fmt.Scan`, `fmt.Scanln` and `fmt.Scanf` parse whitespace-separated values from standard input into variables you pass **by pointer**:

```go
var rating float64
fmt.Print("Rate our pizza: ")
fmt.Scanln(&rating)
```

The series deliberately uses `bufio.NewReader(os.Stdin)` instead (see [[User Input and Comma Ok Syntax]]) because it reads a whole line including spaces (`fmt.Scan` stops at the first space) and hands you the raw text to clean up yourself; both return an `error` you can inspect. `fmt.Scan` is fine for quick experiments.

## Gotchas and Best Practices

> [!warning] A wrong verb does not crash, it prints garbage
> `fmt.Printf("%d", "hello")` compiles and prints `%!d(string=hello)`. Run `go vet` (editors backed by gopls show the same warning inline) and it will flag mismatched verbs for you.

> [!warning] `Printf` needs an explicit `\n`
> Forgetting it makes the next output appear on the same line. The instructor hits this several times in [[Defer]] when mixing `fmt.Print` and `fmt.Println`.

> [!tip] Use `%v` when unsure, `%+v` for structs, `%#v` when debugging
> `%#v` shows quotes around strings and the full type name, which makes it obvious whether a value is the string `"3"` or the number `3`, a classic confusion when decoding JSON (see [[Consuming JSON Data]]).

> [!note] `Println` and pointers
> Printing a pointer to a basic type with `Println` shows an address like `0xc000012345`, not the value; dereference with `*p` to see it (see [[Pointers]]). Pointers to structs, slices and maps are the exception: `fmt` prints them as `&{Hitesh hitesh@go.dev 16}` or `&[1 2 3]`.

## Related Notes

- [[Golang Notes]] — map of content.
- [[Go Quick Reference]] — the one-page cheat sheet.
- [[Error Handling in Go]] — `fmt.Errorf` and `%w`.
- [[Installation and Hello World]] — the first `fmt.Println`.
- [[Variables, Types and Constants]] — where `%T` is introduced.
- [[User Input and Comma Ok Syntax]] — reading input with `bufio` instead of `fmt.Scan`.
- [[Structs]] — `%v` vs `%+v` on struct values.
- [[Creating JSON Data]] — printing `[]byte` JSON with `%s`.
