---
tags:
  - golang
  - basics
section: "02 Basics"
video: https://www.youtube.com/watch?v=zYIZtbyUIDY
video_title: "Comma ok syntax and packages in golang"
order: 7
---
# User Input and Comma Ok Syntax

> [!info] Source
> 🎬 [Comma ok syntax and packages in golang](https://www.youtube.com/watch?v=zYIZtbyUIDY) · Section **02 Basics** of [[Golang Notes]]
> ⬅️ [[Variables, Types and Constants]] · ➡️ [[Type Conversions]]

## Overview

Reading input from the keyboard is a small task that teaches a lot: how to find and read a **standard library package** on `pkg.go.dev`, how `bufio` and `os` fit together, and, most importantly, the **comma ok syntax** (also called *comma error syntax*), which is how Go handles errors instead of `try`/`catch`. The result of this video is a tiny "rate our pizza" program that the next note, [[Type Conversions]], extends into something that can do arithmetic on the input.

## Project setup

```text
03userinput/
├── go.mod      ← go mod init userinput
└── main.go
```

## Finding the right package: `bufio` and `os`

Go's official package index is **https://pkg.go.dev**. Searching for `bufio` opens a page with the same layout every package has:

- **Overview**: what the package does. `bufio` implements *buffered* input/output: it wraps a reader or writer (keyboard, file, network) and reads in efficient chunks.
- **Index**: every constant, variable, function and type, with links.
- **Functions** and **Types**: the actual API. For `bufio` the interesting pieces are the `Reader` and `Scanner` types and the constructor `NewReader`.

![[07_userinput_bufio_docs_0255.png]]
*Figure: the bufio package page on pkg.go.dev with its Overview / Index / Functions / Types layout (video at 02:55)*

The second package is **`os`**, which exposes operating-system things such as files, environment variables and the three standard streams. We need `os.Stdin`, the standard input (your keyboard in a terminal).

> [!tip] Read docs in small doses
> The instructor's advice: do not try to understand the whole page. Open it, find the one function you need, read its signature, move on. Each video adds one more package this way. The habit is described in [[GOPATH and Go Docs]].

## Creating a reader

```go
reader := bufio.NewReader(os.Stdin)
```

- `bufio.NewReader` takes anything that can be read from and returns a `*bufio.Reader`. The asterisk means it is a **pointer** (explained in [[Pointers]]); for now, just know that `reader` refers to one buffered reader sitting on top of the keyboard.
- `:=` is used because, when working with library types, you often do not want to spell out the exact return type. The compiler infers it ([[Variables, Types and Constants]]).
- Note the capitals again: `bufio.NewReader`, `os.Stdin`. Both are exported names, as explained in [[Lexer and Types]]. The package names themselves are lowercase.
- You do not write the `import` lines; saving the file makes the Go extension add `bufio`, `fmt` and `os` to the import block.

## Reading a line: `ReadString`

The reader has several methods (`ReadByte`, `ReadRune`, `ReadLine`, `ReadString`, ...). `ReadString` is the simplest for line input:

```go
input, _ := reader.ReadString('\n')
```

`ReadString(delim byte)` keeps reading until it finds the delimiter you pass. `'\n'` means "read until the user presses Enter". It returns **two values**: the text (including the delimiter) and an `error`.

## The comma ok syntax

This is the first time the pattern appears, and it will appear in almost every program from now on. Go has no `try`/`catch`. Instead, a function that can fail **returns the error as an ordinary value**, next to its result:

```go
input, err := reader.ReadString('\n')
```

- `input` receives the data when everything went well.
- `err` receives the problem if something went wrong, otherwise it is `nil` (empty).

You must accept both values; taking one is a compile error:

```text
./main.go:11:11: assignment mismatch: 1 variable but reader.ReadString returns 2 values
```

If you genuinely do not care about one of them, assign it to the **blank identifier** `_`, which throws the value away and never triggers the "declared and not used" error:

| Form | Meaning |
|---|---|
| `input, err := reader.ReadString('\n')` | keep both; check `err` before using `input` (the idiomatic way) |
| `input, _ := reader.ReadString('\n')` | ignore the error (fine for a demo, risky in real code) |
| `_, err := reader.ReadString('\n')` | ignore the data, only care whether it failed (used with loops and side-effect calls) |

Think of the two halves as the `try` and the `catch`: the first value is what you get when the "try" succeeds; the second is what you would have "caught". The difference is that nothing is thrown, nothing unwinds the stack, and the check is explicit and local. The full pattern (`if err != nil { ... }`) is used in the next note, [[Type Conversions]], and the philosophy is collected in [[Error Handling in Go]].

> [!note] "Comma ok" versus "comma error"
> The same two-value shape appears elsewhere with a `bool` instead of an `error`: reading a key from a map (`value, ok := m["key"]`, see [[Maps]]) or receiving from a channel (`val, open := <-ch`, see [[Channels and Deadlock]]). Hence the two names for the same idea.

## Example: rate our pizza

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	welcome := "Welcome to user input"
	fmt.Println(welcome)

	reader := bufio.NewReader(os.Stdin)
	fmt.Println("Enter the rating for our Pizza:")

	// comma ok || err err
	input, _ := reader.ReadString('\n')
	fmt.Println("Thanks for rating, ", input)
	fmt.Printf("Type of this rating is %T", input)
}
```

Run it and type `5` followed by Enter:

**Output:**
```text
Welcome to user input
Enter the rating for our Pizza:
5
Thanks for rating,  5

Type of this rating is string
```

Two details hide in that output:

1. There is a **blank line** after `Thanks for rating, 5`. `ReadString` returns the text *including* the trailing `\n` the user typed, and `Println` adds another newline after it.
2. `%T` reports **`string`**. Even though you typed a number, what you have is text. You cannot add 1 to it, compare it numerically, or store it in an `int` without converting it first. That is the whole topic of [[Type Conversions]].

## Other ways to read input

Not in the video, but useful to know:

```go
// Scanner: line-oriented, strips the newline for you
scanner := bufio.NewScanner(os.Stdin)
scanner.Scan()
line := scanner.Text()

// fmt.Scanln: reads space-separated values straight into variables
var rating int
fmt.Scanln(&rating)
```

`bufio.Scanner` is what you want for reading many lines (for example a file, see [[Working with Files]]). `fmt.Scanln` is convenient but stops at the first space and is strict about types. The `&rating` argument is a pointer, which is explained in [[Pointers]].

## Gotchas and Best Practices

> [!warning] The newline comes with the input
> `ReadString('\n')` includes the delimiter. On Windows terminals you may even get `\r\n`. Trim it before using the value (`strings.TrimSpace`, shown in [[Type Conversions]]).

> [!warning] Ignoring errors with `_` is a shortcut, not a habit
> For the demo it keeps the code short. In anything that matters, keep the `err` and check it; see [[Error Handling in Go]].

> [!tip] When does `ReadString` return an error?
> Only when the data it returns does **not** end in the delimiter, typically `io.EOF` when input is closed (for example when input is piped from a file that lacks a final newline). You still get the partial text.

> [!tip] Hover in VS Code
> Hovering `reader` shows `*bufio.Reader`; hovering `ReadString` shows its signature `(string, error)`. The editor tells you how many values to expect before you compile.

## Video Timeline

| Time | What happens |
|---|---|
| [00:28](https://youtu.be/zYIZtbyUIDY?t=28) | New folder `03userinput`, `go mod init userinput` |
| [02:12](https://youtu.be/zYIZtbyUIDY?t=132) | Two things to learn: a package for input, and the comma ok syntax |
| [02:37](https://youtu.be/zYIZtbyUIDY?t=157) | Reading the `bufio` page on pkg.go.dev: overview, index, functions, types |
| [04:44](https://youtu.be/zYIZtbyUIDY?t=284) | `bufio.NewReader(os.Stdin)`; imports added on save |
| [06:26](https://youtu.be/zYIZtbyUIDY?t=386) | Hovering shows the reader is a pointer |
| [06:51](https://youtu.be/zYIZtbyUIDY?t=411) | Comma ok syntax explained: no try/catch, errors as values, `_` to discard |
| [08:33](https://youtu.be/zYIZtbyUIDY?t=513) | Running the program and rating the pizza |
| [09:24](https://youtu.be/zYIZtbyUIDY?t=564) | `%T` shows the input is a `string` |
| [10:16](https://youtu.be/zYIZtbyUIDY?t=616) | The two values as try and catch; `_, err` and `input, _` variants |

## Related Notes

- [[Error Handling in Go]] — the full story of `error` values, `if err != nil` and `panic`.
- [[Type Conversions]] — next: turning the string into a number with `strconv`, and trimming the newline.
- [[Variables, Types and Constants]] — `:=` and the "declared and not used" rule that `_` works around.
- [[Lexer and Types]] — why `NewReader` and `Stdin` are capitalised.
- [[Pointers]] — what the `*` in `*bufio.Reader` means.
- [[Maps]] and [[Channels and Deadlock]] — the comma ok form with a boolean.
- [[Working with Files]] — the same reader/scanner ideas applied to files.
- [[fmt Package and Format Verbs]] — `Println` vs `Printf` and `%T`.
