---
tags:
  - golang
  - basics
section: "02 Basics"
video: https://www.youtube.com/watch?v=9fYqg6uo-UU
video_title: "Variables, types and constants"
order: 6
---
# Variables, Types and Constants

> [!info] Source
> 🎬 [Variables, types and constants](https://www.youtube.com/watch?v=9fYqg6uo-UU) · Section **02 Basics** of [[Golang Notes]]
> ⬅️ [[Lexer and Types]] · ➡️ [[User Input and Comma Ok Syntax]]

## Overview

The first real coding video. It shows every way to declare a **variable** in Go (`var` with a type, `var` with an inferred type, the short `:=` form), what happens when a value does not fit its type, what a variable contains before you assign anything (**zero values**), and how **constants** differ. It also introduces `fmt.Printf` with the `%T` verb to print a variable's type (see [[fmt Package and Format Verbs]]). The types themselves were introduced in [[Lexer and Types]]; this note is about *using* them.

## Project setup

Create `02variables/main.go` next to `01hello` and initialise a module, exactly as in [[Installation and Hello World]]:

```bash
go mod init variables
```

> [!note] Why `go mod init` every time?
> Technically a lone `main.go` runs fine with `go run main.go` and no module. The instructor insists on initialising a module in every folder anyway because that is how production code is structured; the habit pays off in [[Go Modules]].

## Declaring a variable with `var`

The full form is `var <name> <type> = <value>`:

```go
var username string = "hitesh"
fmt.Println(username)
fmt.Printf("Variable is of type: %T \n", username)
```

**Output:**
```text
hitesh
Variable is of type: string
```

Two things happen on save that are worth noticing:

- `fmt.Printf` works like C's `printf`: a format string with **placeholders** (verbs) filled from the arguments. `%T` prints the *type* of the value and `\n` ends the line. `fmt.Println` adds the newline for you; `Printf` does not. Full list of verbs in [[fmt Package and Format Verbs]].
- If you declare `username` and never use it, the program does not compile:
  ```text
  ./main.go:4:6: declared and not used: username
  ```
  Go treats an unused variable as an error, not a warning. The `fmt` import also vanished on its own while nothing used it: that is the VS Code Go extension (gopls with goimports) tidying imports on save, not the compiler or the lexer from [[Lexer and Types]].

### Booleans

```go
var isLoggedIn bool = false
fmt.Println(isLoggedIn)
fmt.Printf("Variable is of type: %T \n", isLoggedIn)
```

**Output:**
```text
false
Variable is of type: bool
```

Only `true` and `false` are allowed. The instructor's naming habit, worth copying: prefix booleans with `is` (`isLoggedIn`, `isVerified`) so the name reads as a question.

### Integers and their limits

```go
var smallVal uint8 = 255
fmt.Println(smallVal)
fmt.Printf("Variable is of type: %T \n", smallVal)
```

**Output:**
```text
255
Variable is of type: uint8
```

Change `255` to `256` and the compiler refuses:

```text
./main.go:6:23: cannot use 256 (untyped int constant) as uint8 value in variable declaration (overflows)
```

`uint8` is 8 bits, so 0 to 255 is all it can hold. The ranges come straight from the spec's *Numeric types* section:

| Type | Range |
|---|---|
| `uint8` | 0 to 255 |
| `uint16` | 0 to 65 535 |
| `uint32` | 0 to 4 294 967 295 |
| `uint64` | 0 to 18 446 744 073 709 551 615 |
| `int8` | -128 to 127 |
| `int16` | -32 768 to 32 767 |
| `int32` | -2 147 483 648 to 2 147 483 647 |
| `int64` | -9 223 372 036 854 775 808 to 9 223 372 036 854 775 807 |
| `int`, `uint` | same size as the platform word: 64 bits on a 64-bit machine |

![[06_variables_numeric_types_spec_0735.png]]
*Figure: the Numeric types table in the Go specification (video at 07:35)*

> [!tip] Use plain `int` unless you have a reason not to
> The sized types exist for binary formats, hardware registers and memory-tight structures. For counters, lengths and everyday arithmetic, `int` is the idiomatic choice; you do not need to memorise the table.

### Floats and precision

```go
var smallFloat float64 = 255.45544511254451885
fmt.Println(smallFloat)
fmt.Printf("Variable is of type: %T \n", smallFloat)
```

**Output:**
```text
255.4554451125445
Variable is of type: float64
```

The same literal stored in a `float32` prints `255.45544`. A `float32` keeps roughly 6 to 7 significant decimal digits, a `float64` roughly 15 to 16; the rest of the literal is silently rounded away. That is why `float64` is the default choice (and why `strconv.ParseFloat` in [[Type Conversions]] takes a bit size of 64).

## Zero values: no garbage, ever

Declare a variable without assigning anything and Go initialises it to the **zero value** of its type. There are no uninitialised, garbage-filled variables as in C.

```go
var anotherVariable int
fmt.Println(anotherVariable)
fmt.Printf("Variable is of type: %T \n", anotherVariable)
```

**Output:**
```text
0
Variable is of type: int
```

| Type | Zero value |
|---|---|
| numbers (`int`, `float64`, ...) | `0` |
| `string` | `""` (empty string, zero length) |
| `bool` | `false` |
| pointers, slices, maps, channels, functions, interfaces | `nil` |

The instructor's exercise ("what is inside an empty string?"): it is `""`, which `fmt.Printf("%q", s)` prints as two quote marks. The `nil` row becomes important in [[Pointers]], [[Slices]] and [[Maps]].

## Implicit typing: let the compiler infer

You can leave the type out and keep `var`; the compiler infers it from the value:

```go
var website = "learncodeonline.in"
fmt.Println(website)
```

The variable is still **statically typed** as `string`; inference only saves typing. Trying to assign a different kind of value later fails:

```go
website = 3
```

```text
./main.go:7:12: cannot use 3 (untyped int constant) as string value in assignment
```

## The short declaration `:=` (walrus operator)

The most common syntax in real Go code drops `var` entirely:

```go
numberOfUser := 300000.0
fmt.Println(numberOfUser)
```

**Output:**
```text
300000
```

`:=` **declares and initialises** in one step and infers the type (`float64` here because of the `.0`). You can reassign it later with plain `=` as long as the type matches.

> [!warning] `:=` only works inside functions
> Move the same line to package level (outside `main`) and the compiler stops you:
> ```go
> jwtToken := 300000   // at package level
> ```
> ```text
> ./main.go:5:1: syntax error: non-declaration statement outside function body
> ```
> At package level you must use `var jwtToken = 300000` or `var jwtToken int = 300000`. Inside any function, `:=` is fine and preferred.

## Constants

A **constant** is a value that can never change during the run of the program. Declare it with `const`; you cannot use `:=` for constants.

```go
const LoginToken string = "ghabbhhjd" // Public
```

The capital **L** is not decoration. As explained in [[Lexer and Types]], an identifier whose first letter is capitalised is **exported**: `LoginToken` can be used from any other file and any other package that imports this one. A lowercase `loginToken` would be private to the package. Go has no `public` keyword; the letter case is the keyword.

> [!example] Extra: constant groups and `iota`
> Not in the video, but you will see it in real code. `iota` numbers the constants of a group starting at 0:
> ```go
> const (
> 	Sunday = iota // 0
> 	Monday        // 1
> 	Tuesday       // 2
> )
> ```

## Example: the complete variables program

```go
package main

import "fmt"

const LoginToken string = "ghabbhhjd" // Public

func main() {
	var username string = "hitesh"
	fmt.Println(username)
	fmt.Printf("Variable is of type: %T \n", username)

	var isLoggedIn bool = false
	fmt.Println(isLoggedIn)
	fmt.Printf("Variable is of type: %T \n", isLoggedIn)

	var smallVal uint8 = 255
	fmt.Println(smallVal)
	fmt.Printf("Variable is of type: %T \n", smallVal)

	var smallFloat float64 = 255.45544511254451885
	fmt.Println(smallFloat)
	fmt.Printf("Variable is of type: %T \n", smallFloat)

	// default values and some aliases
	var anotherVariable int
	fmt.Println(anotherVariable)
	fmt.Printf("Variable is of type: %T \n", anotherVariable)

	// implicit type
	var website = "learncodeonline.in"
	fmt.Println(website)

	// no var style
	numberOfUser := 300000.0
	fmt.Println(numberOfUser)

	fmt.Println(LoginToken)
	fmt.Printf("Variable is of type: %T \n", LoginToken)
}
```

**Output:**
```text
hitesh
Variable is of type: string 
false
Variable is of type: bool 
255
Variable is of type: uint8 
255.4554451125445
Variable is of type: float64 
0
Variable is of type: int 
learncodeonline.in
300000
ghabbhhjd
Variable is of type: string 
```

## Gotchas and Best Practices

> [!warning] Unused variables and imports stop the build
> This is the strictness that makes Go code clean. Either use the variable or delete it; the blank identifier `_` (introduced in [[User Input and Comma Ok Syntax]]) is the escape hatch for values you must receive but do not need.

> [!warning] Overflow is a compile error for constants but wraps at run time
> `var x uint8 = 256` is rejected. But `var x uint8 = 255; x++` compiles and `x` silently becomes `0`, because arithmetic on fixed-size integers wraps around. Pick a type large enough for the values you will compute, not only the ones you type.

> [!tip] Prefer `:=` inside functions, `var` at package level
> `:=` is the idiomatic choice inside functions. Use `var` when you need a zero value without an initial value, when you want to state the type explicitly, or when declaring at package level.

> [!tip] Capital letter means "part of my API"
> Decide the case of every constant, function and struct field on purpose. Exported names are a promise to other packages.

## Video Timeline

| Time | What happens |
|---|---|
| [00:53](https://youtu.be/9fYqg6uo-UU?t=53) | New folder `02variables`, `go mod init variables` |
| [02:10](https://youtu.be/9fYqg6uo-UU?t=130) | First `var username string`; the "declared and not used" error |
| [03:26](https://youtu.be/9fYqg6uo-UU?t=206) | `fmt.Printf` with `%T` and `\n` |
| [04:44](https://youtu.be/9fYqg6uo-UU?t=284) | Booleans and the `is` naming habit |
| [06:00](https://youtu.be/9fYqg6uo-UU?t=360) | `uint8` holds 255 but not 256; reading *Numeric types* in the spec |
| [08:33](https://youtu.be/9fYqg6uo-UU?t=513) | `float32` vs `float64` precision |
| [10:15](https://youtu.be/9fYqg6uo-UU?t=615) | Zero values: a declared `int` is `0`, never garbage |
| [11:31](https://youtu.be/9fYqg6uo-UU?t=691) | Implicit typing with `var website = ...`; type cannot change later |
| [12:48](https://youtu.be/9fYqg6uo-UU?t=768) | The walrus operator `:=` |
| [14:04](https://youtu.be/9fYqg6uo-UU?t=844) | `:=` is not allowed outside functions |
| [14:54](https://youtu.be/9fYqg6uo-UU?t=894) | Constants, and why `LoginToken` starts with a capital letter |

## Related Notes

- [[Lexer and Types]] — the type catalogue and the exported-name rule used here.
- [[fmt Package and Format Verbs]] — `Println` vs `Printf`, the `%T` verb and friends.
- [[User Input and Comma Ok Syntax]] — next: reading values from the keyboard, and the blank identifier.
- [[Type Conversions]] — turning strings into numbers with `strconv`.
- [[Memory Management]] and [[Pointers]] — what a variable *is* in memory.
- [[Functions]] — scope: why `:=` lives inside function bodies.
- [[Go Quick Reference]] — declaration forms at a glance.
