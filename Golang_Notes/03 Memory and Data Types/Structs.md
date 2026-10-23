---
tags:
  - golang
  - memory
section: "03 Memory and Data Types"
video: https://www.youtube.com/watch?v=NMTN543WVQY
video_title: "Structs in golang"
order: 17
---
# Structs

> [!info] Source
> 🎬 [Structs in golang](https://www.youtube.com/watch?v=NMTN543WVQY) · Section **03 Memory and Data Types** of [[Golang Notes]]
> ⬅️ [[Maps]] · ➡️ [[If Else]]

## Overview

Go has **no classes**. The closest thing is the **struct**: a type that groups named fields of different types into one value, such as a `User` with a name, an email, a verification status and an age. Structs are the last data type in this section and they are the backbone of everything that follows: [[Methods]] attach behaviour to them, [[Creating JSON Data]] converts them to and from JSON, and every model in [[Building an API - Models]] and [[Defining Models for MongoDB]] is a struct. The instructor is explicit that this video is only the basics and that structs are revisited twice more, once functions are known and again in the API section.

## No inheritance

The most important fact in the video: **there is no inheritance in Go**. No `extends`, no `super`, no parent and child classes. The Go designers believe inheritance hierarchies make code harder to read, because understanding one file means hunting through several others. Go's philosophy is that what is on the screen should be understandable almost completely on its own. Code is still organised and reused, but through **composition** (putting one struct inside another) and interfaces, not through class hierarchies.

> [!note]
> If you come from Java or Python, resist the urge to look for the base class. A Go struct is a plain value with fields; behaviour is added separately with methods, and shared behaviour is expressed by interfaces.

## Defining a struct

A struct type is declared with the **`type`** keyword, a name, and the keyword `struct` followed by the fields in curly braces, one per line with its type:

```go
type User struct {
	Name   string
	Email  string
	Status bool
	Age    int
}
```

`gofmt` (which VS Code runs on save) aligns the field types into a column automatically, which is why the fields in the video "line up" after saving.

### Capital letters mean exported

Look at the capitalisation: `User`, `Name`, `Email`, `Status`, `Age`. In Go an identifier that starts with an **uppercase letter is exported**, meaning other packages can see it; a lowercase identifier is private to its package. The instructor calls this "the most common thing to miss": the struct is a template that other code should be able to use, so the type *and* its fields are capitalised. This matters enormously later, because `encoding/json` can only see exported fields (see [[Creating JSON Data]]).

## Creating a struct value

With the type defined, you create a value with the type name and the field values in curly braces. In the **positional** form the values must appear in the same order as the fields:

```go
hitesh := User{"Hitesh", "hitesh@go.dev", true, 16}
```

The **named-field** form is longer but safer: the order does not matter and any field you leave out gets its zero value. This is the form you will see in real code and in the API section:

```go
hitesh := User{Name: "Hitesh", Email: "hitesh@go.dev", Status: true, Age: 16}
```

A struct declared without a value is its **zero value**: every field is zeroed (`""`, `false`, `0`), exactly like the zero values in [[Variables, Types and Constants]].

## Printing a struct

The format verbs from [[fmt Package and Format Verbs]] make a real difference with structs:

| Verb | Prints | Example |
|---|---|---|
| `%v` (or `Println`) | values only | `{Hitesh hitesh@go.dev true 16}` |
| `%+v` | field names **and** values | `{Name:Hitesh Email:hitesh@go.dev Status:true Age:16}` |
| `%#v` | Go-syntax representation | `main.User{Name:"Hitesh", Email:"hitesh@go.dev", Status:true, Age:16}` |

`%+v` is the one the instructor recommends for debugging, because you can see which value belongs to which field.

## Accessing fields

Individual fields are read and written with the dot: `hitesh.Name`, `hitesh.Email`. Remember the capital letters, since the fields were declared exported.

```go
fmt.Printf("Name is %v and email is %v.\n", hitesh.Name, hitesh.Email)
hitesh.Age = 17
```

## Example: the instructor's structs program

The companion code for this video (`11mystructs/main.go`), with the `%#v` form added:

```go
package main

import "fmt"

func main() {
	fmt.Println("Structs in golang")
	// no inheritance in golang; no super or parent

	hitesh := User{"Hitesh", "hitesh@go.dev", true, 16}
	fmt.Println(hitesh)
	fmt.Printf("hitesh details are: %+v\n", hitesh)
	fmt.Printf("hitesh in Go syntax: %#v\n", hitesh)
	fmt.Printf("Name is %v and email is %v.\n", hitesh.Name, hitesh.Email)
}

type User struct {
	Name   string
	Email  string
	Status bool
	Age    int
}
```

**Output:**

```text
Structs in golang
{Hitesh hitesh@go.dev true 16}
hitesh details are: {Name:Hitesh Email:hitesh@go.dev Status:true Age:16}
hitesh in Go syntax: main.User{Name:"Hitesh", Email:"hitesh@go.dev", Status:true, Age:16}
Name is Hitesh and email is hitesh@go.dev.
```

Note that the `type User struct` declaration sits *below* `main` and still works: top-level declarations in a Go file can appear in any order.

## Example: named fields, zero values, pointers and composition

A second program covering the things you will reach for right after the basics: named-field literals, the zero value, pointers to structs (from [[Pointers]]), value semantics, and composition in place of inheritance.

```go
package main

import "fmt"

type Address struct {
	City    string
	Country string
}

type User struct {
	Name    string
	Email   string
	Status  bool
	Age     int
	Address Address // composition: a struct inside a struct
}

func main() {
	// named fields: any order, omitted fields get their zero value
	asha := User{Name: "Asha", Email: "asha@go.dev", Address: Address{City: "Pune", Country: "India"}}
	fmt.Printf("%+v\n", asha)

	// the zero value of a struct zeroes every field
	var nobody User
	fmt.Printf("%+v\n", nobody)

	// pointer to a struct: Go dereferences automatically for field access
	p := &asha
	p.Age = 25
	fmt.Println("age via pointer:", asha.Age, "| city:", p.Address.City)

	// structs are values: assignment copies the whole thing
	copyOfAsha := asha
	copyOfAsha.Name = "Someone else"
	fmt.Println(asha.Name, "|", copyOfAsha.Name)

	// structs whose fields are all comparable can be compared with ==
	fmt.Println(asha.Address == Address{City: "Pune", Country: "India"})
}
```

**Output:**

```text
{Name:Asha Email:asha@go.dev Status:false Age:0 Address:{City:Pune Country:India}}
{Name: Email: Status:false Age:0 Address:{City: Country:}}
age via pointer: 25 | city: Pune
Asha | Someone else
true
```

## What comes later: methods and struct tags

The instructor flags two things that need foundations the course has not built yet:

- **Methods**: a function attached to a struct, written `func (u User) GetStatus()`. Covered in [[Methods]], including why `hitesh.NewMail()` with a value receiver does *not* change the original.
- **Struct tags**: the "fancy stuff with backticks" after a field, such as ``Name string `json:"coursename"` ``, which tells `encoding/json` what key to use. Covered in [[Creating JSON Data]] and used in every model of [[Building an API - Models]] and [[Defining Models for MongoDB]].

```go
type Course struct {
	Name  string `json:"coursename"`
	Price int    `json:"price"`
}
```

## Gotchas and Best Practices

> [!warning] Lowercase fields are invisible outside the package
> A field named `email` cannot be read by other packages and is silently skipped by `encoding/json`. If a JSON field keeps coming out empty, check the capital letter first.

> [!warning] Positional literals break when the struct changes
> `User{"Hitesh", "hitesh@go.dev", true, 16}` stops compiling (or, worse, silently shuffles values) the moment a field is added or reordered. Use named fields in anything beyond a throwaway example.

> [!tip] Pass large structs by pointer
> Structs are copied on assignment and when passed to functions. For small structs that is fine; for large ones, or when the function must modify the struct, pass `&user` as shown in [[Pointers]].

> [!note] Anonymous structs
> For one-off data you can skip the `type` declaration: `point := struct{ X, Y int }{1, 2}`. Handy in tests and quick scripts.

## Video Timeline

| Time | What happens |
|---|---|
| [00:16](https://youtu.be/NMTN543WVQY?t=16) | Structs are Go's alternative to classes; revisited later in the course |
| [01:50](https://youtu.be/NMTN543WVQY?t=110) | No inheritance, no `super`, no parent and child |
| [02:41](https://youtu.be/NMTN543WVQY?t=161) | Defining `type User struct` with Name, Email, Status, Age |
| [03:41](https://youtu.be/NMTN543WVQY?t=221) | Capital first letters: exported type and exported fields |
| [04:19](https://youtu.be/NMTN543WVQY?t=259) | Creating a value with `User{...}` and printing it |
| [05:33](https://youtu.be/NMTN543WVQY?t=333) | `%+v` shows field names; `hitesh.Name` reads one field |

## Related Notes

- [[Methods]] — attaching behaviour to a struct; value vs pointer receivers.
- [[Pointers]] — `&user` and automatic dereferencing of struct pointers.
- [[Maps]] — use a map for dynamic keys, a struct for a fixed set of named fields.
- [[Creating JSON Data]] and [[Consuming JSON Data]] — struct tags and marshalling structs.
- [[Building an API - Models]] — `Course` and `Author` structs as API models.
- [[Defining Models for MongoDB]] — the `Netflix` struct with `bson` tags.
- [[Variables, Types and Constants]] — zero values, which apply field by field.
- [[fmt Package and Format Verbs]] — `%v`, `%+v` and `%#v`.
- [[Go Quick Reference]] — struct syntax at a glance.
