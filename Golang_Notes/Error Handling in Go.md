---
tags:
  - golang
  - hub
section: "Hub"
---
# Error Handling in Go

> [!info] Hub note
> Go has no exceptions. Errors are ordinary **values** that functions return, and you handle them right where they happen. The pattern `if err != nil { … }` appears in almost every video of the series from [[User Input and Comma Ok Syntax]] onwards, so this note explains it once, properly. Part of [[Golang Notes]].

## Overview

In languages like Java or JavaScript a failing function *throws* and some caller up the stack *catches*. Go made a deliberate design choice against that: a function that can fail returns an extra value of type **`error`**, and the caller checks it immediately. The result is more lines of code, but every failure point is visible in the source, and nothing can silently unwind past your code. The instructor calls the two-value return the **comma ok** / **comma error** syntax (see [[User Input and Comma Ok Syntax]]).

## The `error` type

`error` is a built-in **interface** with a single method:

```go
type error interface {
	Error() string
}
```

Anything with an `Error() string` method is an error. A `nil` error means "nothing went wrong", which is why the check is `err != nil`.

## Creating errors

| Way | When to use it |
|---|---|
| `errors.New("message")` | A fixed message, often stored in a package-level variable (a **sentinel** error) |
| `fmt.Errorf("context: %v", err)` | A formatted message (see [[fmt Package and Format Verbs]]) |
| `fmt.Errorf("context: %w", err)` | Same, but **wraps** `err` so callers can still detect it with `errors.Is` / `errors.As` |
| A custom struct with an `Error()` method | When the caller needs structured data (field name, status code …) |

## The idiom

```go
numRating, err := strconv.ParseFloat(strings.TrimSpace(input), 64)
if err != nil {
	fmt.Println(err)
	return
}
// from here on numRating is safe to use
```

Three rules make this work:

1. **Check immediately.** The check sits on the line after the call, so the "happy path" below it never sees a bad value.
2. **Handle once.** Either log it, return it (possibly wrapped with more context), or stop the program. Doing two of these for the same error produces duplicate logs.
3. **Keep the happy path un-indented.** Go code returns early on error and continues at the left margin, which is why Go functions read top to bottom without nesting.

## Comma ok everywhere

The same two-value shape shows up in places that are not errors at all. The second value is a `bool` that says whether the first one is meaningful:

| Expression | Second value means | Note |
|---|---|---|
| `v, ok := myMap["key"]` | the key existed | [[Maps]] |
| `v, ok := x.(string)` | the interface holds a `string` | [[Consuming JSON Data]] |
| `v, ok := <-ch` | the channel is still open | [[Channels and Deadlock]] |
| `v, err := strconv.ParseFloat(s, 64)` | an `error`, not a bool | [[Type Conversions]] |
| `input, err := reader.ReadString('\n')` | an `error` | [[User Input and Comma Ok Syntax]] |

## Ignoring errors with `_`

Go refuses to compile if you declare a variable and never use it, so early videos write `input, _ := reader.ReadString('\n')` to throw the error away. That is acceptable in a throwaway script and dangerous everywhere else: a failed `ReadString` returns whatever partial text was read, possibly nothing, and that silently flows into the rest of the program.

> [!warning] `_` is a promise that failure cannot matter
> Before writing `_` for an error, ask what the program would do with an empty or zero value. In [[Working with Files]] the instructor replaces `_` with a small `checkNilErr` helper that panics, which is the right instinct.

## `panic`, `recover` and `log.Fatal`

- **`panic(err)`** stops the current goroutine, runs all deferred calls (see [[Defer]]), and crashes the program with a stack trace. Use it for *programmer* errors or situations where continuing makes no sense (the file helper in [[Working with Files]], failed `http.Get` in [[Handling Web Requests]]).
- **`log.Fatal(err)`** prints the error and exits with status 1 **without** running deferred calls. The MongoDB section uses it when the database connection cannot be made (see [[Connecting to MongoDB]]).
- **`recover()`** inside a deferred function catches a panic and lets the program continue. It is rare in application code; web frameworks use it so one bad request does not kill the server.

## Sentinel errors, custom errors, `errors.Is` and `errors.As`

When a wrapped error travels up several layers, callers still need to ask *what kind* of failure it was:

- `errors.Is(err, ErrNotFound)` walks the wrap chain and reports whether `ErrNotFound` is anywhere in it.
- `errors.As(err, &target)` finds the first error in the chain that has `target`'s type and copies it out, so you can read its fields.

## Example: all of it in one program

```go
package main

import (
	"errors"
	"fmt"
	"strconv"
)

// A sentinel error: a package-level value callers can compare against.
var ErrNotFound = errors.New("course not found")

// A custom error type carries structured data.
type ValidationError struct {
	Field string
	Msg   string
}

func (e *ValidationError) Error() string {
	return fmt.Sprintf("invalid %s: %s", e.Field, e.Msg)
}

func findCourse(id string) (string, error) {
	courses := map[string]string{"2": "ReactJS", "4": "MERN Stack"}
	name, ok := courses[id] // comma ok: ok is false when the key is missing
	if !ok {
		// %w wraps ErrNotFound so errors.Is can still recognise it
		return "", fmt.Errorf("findCourse(%s): %w", id, ErrNotFound)
	}
	return name, nil
}

func parseRating(input string) (float64, error) {
	rating, err := strconv.ParseFloat(input, 64)
	if err != nil {
		return 0, &ValidationError{Field: "rating", Msg: err.Error()}
	}
	if rating < 1 || rating > 5 {
		return 0, &ValidationError{Field: "rating", Msg: "must be between 1 and 5"}
	}
	return rating, nil
}

// recover turns a panic back into an ordinary error value.
func safeDivide(a, b int) (result int, err error) {
	defer func() {
		if r := recover(); r != nil {
			err = fmt.Errorf("recovered: %v", r)
		}
	}()
	return a / b, nil
}

func main() {
	// 1. The idiom: check err right after the call that produced it.
	if name, err := findCourse("2"); err != nil {
		fmt.Println("error:", err)
	} else {
		fmt.Println("found:", name)
	}

	// 2. Wrapped errors keep the original reachable via errors.Is.
	_, err := findCourse("9")
	fmt.Println("error:", err)
	fmt.Println("is ErrNotFound?", errors.Is(err, ErrNotFound))

	// 3. errors.As pulls a typed error out of the chain.
	_, err = parseRating("abc")
	var ve *ValidationError
	if errors.As(err, &ve) {
		fmt.Println("validation failed on field:", ve.Field)
	}
	fmt.Println("error:", err)

	// 4. panic + recover for truly exceptional situations.
	res, err := safeDivide(10, 0)
	fmt.Println("result:", res, "error:", err)
}
```

**Output:**

```text
found: ReactJS
error: findCourse(9): course not found
is ErrNotFound? true
validation failed on field: rating
error: invalid rating: strconv.ParseFloat: parsing "abc": invalid syntax
result: 0 error: recovered: runtime error: integer divide by zero
```

The last line shows the **named return value** trick: because `err` is named in the signature, the deferred function can assign to it after the panic.

## Where the series uses each technique

| Technique | Video / note |
|---|---|
| `input, _ :=` (ignoring) | [[User Input and Comma Ok Syntax]] |
| `if err != nil { fmt.Println(err) } else { … }` | [[Type Conversions]] |
| `panic(err)` inside a helper | [[Working with Files]], [[Handling Web Requests]] |
| `if err != nil { panic(err) }` after `http.Get` | [[Making GET Requests]] |
| `json.Valid` before `json.Unmarshal` | [[Consuming JSON Data]] |
| `log.Fatal(http.ListenAndServe(...))` | [[Routes and Testing the API]] |
| `log.Fatal(err)` on database failure | [[Connecting to MongoDB]] |
| `val, isChannelOpen := <-ch` | [[Channels and Deadlock]] |

## Gotchas and Best Practices

> [!tip] Add context when you return an error
> `return fmt.Errorf("loading config %s: %w", path, err)` tells the reader *what* was being attempted. Chains of such messages read like a stack trace written in English.

> [!warning] Do not compare error strings
> `err.Error() == "course not found"` breaks the moment the message changes. Use sentinel values with `errors.Is`, or custom types with `errors.As`.

> [!note] `nil` interface vs `nil` pointer
> Returning a typed `nil` pointer (`var e *ValidationError = nil; return e`) as an `error` produces a **non-nil** interface value. Always return the literal `nil` for "no error".

## Related Notes

- [[Golang Notes]] — map of content.
- [[Go Quick Reference]] — cheat sheet.
- [[fmt Package and Format Verbs]] — `Errorf` and the `%w` verb.
- [[User Input and Comma Ok Syntax]] — first appearance of the pattern.
- [[Defer]] — deferred calls run during a panic.
- [[Maps]] and [[Channels and Deadlock]] — comma-ok with a boolean.
- [[Connecting to MongoDB]] — `log.Fatal` on fatal startup errors.
