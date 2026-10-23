---
tags:
  - golang
  - hub
section: "Hub"
---
# Go Quick Reference

> [!info] Hub note
> A one-page cheat sheet of everything the series teaches, with a link to the detailed note for each topic. Every full program here compiles with Go 1.27; the shorter fragments are syntax-checked against the same toolchain. Part of [[Golang Notes]].

## Program structure

```go
package main            // every executable lives in package main

import (
	"fmt"               // standard library packages, grouped and sorted
	"strings"
)

const Pi = 3.14159      // package-level constant

func main() {           // the entry point, no arguments, no return value
	fmt.Println(strings.ToUpper("hello"), Pi)
}
```

- One **module** per project: `go mod init github.com/you/project` creates `go.mod` (see [[Go Modules]]).
- Run with `go run .`, compile with `go build`, run static checks with `go vet`, format with `go fmt` (see [[Installation and Hello World]]).
- Identifiers that start with a **capital letter are exported** (public); lowercase ones are private to the package (see [[Variables, Types and Constants]]).

## Variables and constants

```go
var a int = 10          // explicit type
var b = 20              // type inferred: int
c := 1.5                // short declaration, only inside functions
var d, e = "x", true    // several at once
var zero int            // declared, not assigned: zero value 0
const Token = "abc"     // constants cannot use :=
```

| Family | Types | Zero value | Note |
|---|---|---|---|
| Booleans | `bool` | `false` | |
| Integers | `int` `int8/16/32/64` `uint` `uint8/16/32/64` `uintptr` | `0` | `byte` = `uint8`, `rune` = `int32` |
| Floats | `float32` `float64` | `0` | untyped float literals default to `float64` |
| Complex | `complex64` `complex128` | `0` | rarely used |
| Strings | `string` | `""` | immutable, UTF-8 bytes |
| Composite | array, slice, map, struct, pointer, func, chan, interface | `nil` for slice/map/pointer/func/chan/interface | |

Details: [[Lexer and Types]], [[Variables, Types and Constants]].

## Printing

`fmt.Println(a, b)` · `fmt.Printf("%v %+v %#v %T\n", x, x, x, x)` · `s := fmt.Sprintf("%d items", n)` — see [[fmt Package and Format Verbs]].

## Reading input and converting

```go
reader := bufio.NewReader(os.Stdin)
input, _ := reader.ReadString('\n')                      // includes the newline
f, err := strconv.ParseFloat(strings.TrimSpace(input), 64) // "4.5\n" -> 4.5
n := strconv.Itoa(42)        // int -> "42"
x := 3.99
i := int(x)                  // 3, truncates; conversions are explicit T(v)
half := float64(7) / 2       // 3.5, convert before dividing
```

Details: [[User Input and Comma Ok Syntax]], [[Type Conversions]].

## Control flow

```go
if n := len(sl); n > 2 {    // optional init statement, scoped to the if
	fmt.Println("big")
} else if n == 2 {
	fmt.Println("two")
} else {
	fmt.Println("small")
}

switch day := time.Now().Weekday(); day {
case time.Saturday, time.Sunday:    // several values per case
	fmt.Println("weekend")
default:
	fmt.Println("weekday")          // no break needed; use fallthrough to continue
}

for i := 0; i < 3; i++ { }     // classic
for i := range 3 { fmt.Print(i) }          // 0,1,2 (Go 1.22+, needs `go 1.22` or later in go.mod)
for i, v := range sl { fmt.Println(i, v) } // index, value over slice/array/string
for k, v := range m { fmt.Println(k, v) }  // key, value over map
for n < 3 { n++ }              // while-style
for { break }                  // infinite, exit with break
```

Details: [[If Else]], [[Switch Case]], [[Loops, Break, Continue and Goto]].

## Functions, methods, interfaces

```go
func divide(a, b float64) (float64, error) {   // multiple return values
	if b == 0 {
		return 0, errors.New("division by zero")
	}
	return a / b, nil
}

func sum(nums ...int) int {                    // variadic: nums is []int
	total := 0
	for _, n := range nums {
		total += n
	}
	return total
}

type Rect struct{ W, H float64 }

func (r Rect) Area() float64 { return r.W * r.H }  // value receiver: works on a copy
func (r *Rect) Scale(f float64) {                  // pointer receiver: mutates the caller's value
	r.W *= f
	r.H *= f
}

type Shape interface{ Area() float64 }            // satisfied implicitly by Rect
```

Details: [[Functions]], [[Methods]], [[Structs]].

## Pointers

```go
p := &a        // address of a, type *int
*p = 99        // write through the pointer; a is now 99
var np *int    // nil pointer; dereferencing it panics
```

Details: [[Pointers]], [[Memory Management]].

## Collections

```go
arr := [3]int{1, 2, 3}                 // array: fixed length, value type
sl := []int{5, 2, 9}                   // slice: dynamic view over an array
sl = append(sl, 7)                     // grow (may reallocate)
sort.Ints(sl)                          // [2 5 7 9]
sl = append(sl[:1], sl[2:]...)         // remove index 1 -> [2 7 9]
hs := make([]int, 4)                   // len 4, cap 4, zero-filled
fmt.Println(len(sl), cap(sl))

m := map[string]int{"x": 1}            // or make(map[string]int)
m["y"] = 2
v, ok := m["z"]                        // ok == false when missing
delete(m, "x")

type User struct {
	Name  string
	Email string
}
u := User{Name: "Hitesh", Email: "hitesh@go.dev"}
fmt.Printf("%+v\n", u)
```

Details: [[Arrays]], [[Slices]], [[Maps]], [[Structs]].

## Errors, defer, panic

```go
res, err := divide(10, 0)
if err != nil {
	fmt.Println("error:", err)   // handle it here, once
	return
}
defer file.Close()               // runs when the surrounding function returns, LIFO order
panic("unrecoverable")           // crash with a stack trace; recover() in a deferred func catches it
```

Details: [[Error Handling in Go]], [[Defer]].

## Files

```go
err := os.WriteFile("notes.txt", []byte("hello"), 0644)
data, err := os.ReadFile("notes.txt")        // []byte
f, err := os.Create("out.txt"); defer f.Close()
n, err := io.WriteString(f, "content")
```

Details: [[Working with Files]].

## HTTP client

```go
res, err := http.Get("https://go.dev")
defer res.Body.Close()                       // always
body, err := io.ReadAll(res.Body)            // []byte -> string(body)
endpoint := "http://localhost:8000/post"
res, err = http.Post(endpoint, "application/json", strings.NewReader(`{"a":1}`))
res, err = http.PostForm(endpoint, url.Values{"name": {"hitesh"}})
u, err := url.Parse("https://lco.dev:3000/learn?course=go")   // u.Host, u.Port(), u.Query()["course"]
```

Details: [[Handling Web Requests]], [[Handling URLs]], [[Making GET Requests]], [[Making POST Requests with JSON]], [[Sending Form Data]].

## JSON

```go
type Course struct {
	Name     string   `json:"coursename"`
	Price    int      `json:"price"`
	Password string   `json:"-"`              // never encoded
	Tags     []string `json:"tags,omitempty"`  // dropped when empty
}
out, err := json.MarshalIndent(courses, "", "\t")     // Go -> JSON
var c Course
err = json.Unmarshal(data, &c)                        // JSON -> Go (pointer!)
var generic map[string]any
err = json.Unmarshal(data, &generic)                  // unknown shape
json.NewEncoder(w).Encode(courses)                    // straight into an http.ResponseWriter
json.NewDecoder(r.Body).Decode(&c)                    // straight from a request body
```

Details: [[Creating JSON Data]], [[Consuming JSON Data]].

## HTTP server and REST API (gorilla/mux)

```go
r := mux.NewRouter()
r.HandleFunc("/courses", getAllCourses).Methods("GET")
r.HandleFunc("/course/{id}", getOneCourse).Methods("GET")
log.Fatal(http.ListenAndServe(":4000", r))

func getOneCourse(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	id := mux.Vars(r)["id"]
	// ...
}
```

Since Go 1.22 the standard `http.NewServeMux` accepts `"GET /course/{id}"` patterns and `r.PathValue("id")`, so mux is optional for new projects. Details: [[Building an API - Models]] through [[Routes and Testing the API]].

## MongoDB (mongo-driver v1)

```go
client, err := mongo.Connect(context.TODO(), options.Client().ApplyURI(uri))
coll := client.Database("netflix").Collection("watchlist")
res, err := coll.InsertOne(ctx, movie)                                   // res.InsertedID
id, _ := primitive.ObjectIDFromHex(hex)
_, err = coll.UpdateOne(ctx, bson.M{"_id": id}, bson.M{"$set": bson.M{"watched": true}})
_, err = coll.DeleteOne(ctx, bson.M{"_id": id})
cur, err := coll.Find(ctx, bson.D{{}}); defer cur.Close(ctx)
for cur.Next(ctx) { var m bson.M; cur.Decode(&m) }
```

Details: [[Connecting to MongoDB]], [[Insert Data in MongoDB]], [[Update a Record in MongoDB]], [[Delete One and Delete Many in MongoDB]], [[Get All Documents from MongoDB]].

## Concurrency

```go
var wg sync.WaitGroup
var mu sync.Mutex
counter := 0
ch := make(chan int, 3)          // buffered; make(chan int) is unbuffered

for i := 1; i <= 3; i++ {
	wg.Add(1)                    // before the goroutine starts
	go func(id int) {
		defer wg.Done()
		mu.Lock()
		counter++                // protected shared state
		mu.Unlock()
		ch <- id                 // send
	}(i)
}
wg.Wait()
close(ch)                        // senders close; receivers range until drained
for id := range ch {
	fmt.Println(id)
}
v, open := <-ch                  // comma ok: open == false once closed and empty
```

Run with `go run -race .` to detect data races. Details: [[Concurrency and Goroutines]], [[WaitGroups]], [[Mutex]], [[Race Conditions]], [[Channels and Deadlock]].

## Toolchain commands

| Command | Purpose | Note |
|---|---|---|
| `go mod init <module>` | start a module | [[Go Modules]] |
| `go run .` / `go build` | run / compile the current package | [[Installation and Hello World]] |
| `go vet ./...` | static checks (bad Printf verbs, unreachable code …) | [[fmt Package and Format Verbs]] |
| `go fmt ./...` | format code | |
| `go doc fmt.Println` | read documentation offline | [[GOPATH and Go Docs]] |
| `go get github.com/gorilla/mux@v1.8.1` | add a dependency to go.mod | [[Go Modules]] |
| `go install pkg@latest` | install a tool binary into `$GOPATH/bin` | [[Go Modules]] |
| `go mod tidy` / `go mod verify` / `go mod vendor` | clean up / check / vendor dependencies | [[Go Modules]] |
| `go env GOPATH GOROOT GOOS GOARCH` | inspect the environment | [[GOPATH and Go Docs]] |
| `GOOS=windows GOARCH=amd64 go build` | cross-compile | [[Building for Windows, Linux and Mac]] |
| `go run -race .` | race detector | [[Race Conditions]] |

## Related Notes

- [[Golang Notes]] — the full learning path.
- [[Error Handling in Go]] — the idiom explained.
- [[fmt Package and Format Verbs]] — every verb.
