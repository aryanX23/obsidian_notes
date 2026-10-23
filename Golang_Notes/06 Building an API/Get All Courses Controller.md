---
tags:
  - golang
  - api
section: "06 Building an API"
video: https://www.youtube.com/watch?v=V3S3hKrIdQo
video_title: "Sending a API json response for all courses in golang"
order: 35
---
# Get All Courses Controller

> [!info] Source
> 🎬 [Sending a API json response for all courses in golang](https://www.youtube.com/watch?v=V3S3hKrIdQo) · Section **06 Building an API** of [[Golang Notes]]
> ⬅️ [[Building an API - Models]] · ➡️ [[Get One Course Controller]]

## Overview
With the models from [[Building an API - Models]] in place, the next layer is the **controller**: the function that runs when a request hits a route. This video writes two of them. `serveHome` answers the root URL with a welcome message, and `getAllCourses` sends the entire fake database back as JSON. Neither function is connected to a URL yet; the routing happens in [[Routes and Testing the API]]. Separating "what to do" (controller) from "when to do it" (route) is the structure every Go web service follows, whatever router it uses.

## What a controller is
In [[Go Modules]] the `/` route was wired to a `serveHome` function with `r.HandleFunc("/", serveHome).Methods("GET")`. That function is a controller: when a `GET` request for `/` arrives, the router hands the request to it, and it decides what to send back. Some frameworks call them **handlers**; the instructor calls them controllers because they are "the brain of the operation". In bigger applications controllers live in their own files and folders, which is exactly what happens in [[MongoDB API Controllers]].

Every controller in Go has the same signature:

```go
func name(w http.ResponseWriter, r *http.Request)
```

| Parameter | Type | Role |
|---|---|---|
| `w` | `http.ResponseWriter` | The **writer**: everything you send back (status code, headers, body) goes through it |
| `r` | `*http.Request` | The **request**: everything the client sent (URL, method, headers, body, route parameters) |

> [!warning] Keep the order: writer first, request second
> The instructor calls this out as a common mistake. `func(r *http.Request, w http.ResponseWriter)` is a different type and will not satisfy `HandleFunc`.

`r` is a pointer because a request carries a lot of data and must not be copied (see [[Pointers]]); `w` is an interface, so it is passed as is. The name of the function is free; lowercase names are fine because these controllers are never exported from the package.

## `serveHome`
The root route should never answer with a blank page or an error, so the first controller writes a small HTML greeting:

```go
// serve home route
func serveHome(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("<h1>Welcome to API by LearnCodeOnline</h1>"))
}
```

`w.Write` takes a slice of bytes, the same way file writing did in [[Working with Files]], so the string is converted with `[]byte(...)`. Visiting `http://localhost:4000/` in a browser renders the heading.

**Output:**

```text
<h1>Welcome to API by LearnCodeOnline</h1>
```

Because no `Content-Type` header was set, Go sniffs the first bytes of the body and labels the response `text/html; charset=utf-8` on its own.

## `getAllCourses`
The second controller sends the whole `courses` slice to the client as JSON, in three steps:

```go
func getAllCourses(w http.ResponseWriter, r *http.Request) {
	fmt.Println("Get all courses")                      // 1. log for the developer
	w.Header().Set("Content-Type", "application/json")  // 2. describe the response
	json.NewEncoder(w).Encode(courses)                  // 3. stream the JSON
}
```

### 1. A log line
`fmt.Println("Get all courses")` prints in the **server's terminal**, not in the response. It is a cheap way to see which controller ran when several routes look alike, and it pays off during testing in [[Routes and Testing the API]].

### 2. Setting a header
`w.Header()` returns the response header map, and `Set` writes one key/value pair into it. `Content-Type: application/json` tells the client (and tools like Thunder Client) how to interpret the body. Headers can be set this way any number of times, but only **before** the body is written, because the first `Write` sends the headers down the wire.

> [!warning] The repository has a typo here
> The companion code sets `"applicatioan/json"`. Browsers and Thunder Client still display the body, so the typo goes unnoticed in the video, but a strict client would not treat the response as JSON. The notes use the correct `"application/json"`.

### 3. Encoding straight into the response
[[Creating JSON Data]] used `json.Marshal` to turn a value into a `[]byte`. Here the instructor uses the other API from the same package: `json.NewEncoder(w)` creates an **encoder bound to a writer**, and `.Encode(courses)` serialises the value and writes it directly to that writer. Since `http.ResponseWriter` is an `io.Writer`, the JSON goes straight to the client without an intermediate byte slice.

| | `json.Marshal(v)` | `json.NewEncoder(w).Encode(v)` |
|---|---|---|
| Returns | `[]byte, error` | `error` |
| Where the JSON goes | Into memory, you write it yourself | Straight into `w` |
| Trailing newline | No | Yes, `Encode` appends one |
| Typical use | Build a payload, log it, store it | HTTP responses, files, sockets |

Any value can be encoded. The instructor shows that a map like `map[string]string{"welcome": "hello world"}` would be sent just as happily; here it is the slice of courses.

### What the client receives
Before any data is seeded the response is surprising:

```text
null
```

`courses` is a **nil slice** (declared with `var courses []Course` and never appended to), and `encoding/json` encodes a nil slice as JSON `null`, not `[]`. After the seeding step from [[Routes and Testing the API]] adds two courses, the same route returns:

```json
[
  {
    "courseid": "2",
    "coursename": "ReactJS",
    "price": 299,
    "author": { "fullname": "Hitesh Choudhary", "website": "lco.dev" }
  },
  {
    "courseid": "4",
    "coursename": "MERN Stack",
    "price": 199,
    "author": { "fullname": "Hitesh Choudhary", "website": "go.dev" }
  }
]
```

(The server sends this on a single line; it is shown formatted here.) **Seeding** means filling a database with a few fake records so there is something to test against. It is a normal practice while developing, and it is done in `main` before the server starts listening.

## Example: a runnable version with just these two routes
Everything needed to try the two controllers, including the routing from [[Routes and Testing the API]] so the program actually runs:

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"

	"github.com/gorilla/mux"
)

type Course struct {
	CourseId    string  `json:"courseid"`
	CourseName  string  `json:"coursename"`
	CoursePrice int     `json:"price"`
	Author      *Author `json:"author"`
}

type Author struct {
	Fullname string `json:"fullname"`
	Website  string `json:"website"`
}

var courses []Course

func main() {
	fmt.Println("API - LearnCodeOnline.in")
	r := mux.NewRouter()

	// seeding
	courses = append(courses, Course{CourseId: "2", CourseName: "ReactJS", CoursePrice: 299,
		Author: &Author{Fullname: "Hitesh Choudhary", Website: "lco.dev"}})

	r.HandleFunc("/", serveHome).Methods("GET")
	r.HandleFunc("/courses", getAllCourses).Methods("GET")

	log.Fatal(http.ListenAndServe(":4000", r))
}

func serveHome(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("<h1>Welcome to API by LearnCodeOnline</h1>"))
}

func getAllCourses(w http.ResponseWriter, r *http.Request) {
	fmt.Println("Get all courses")
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(courses)
}
```

Run it with `go run main.go` and call it from a second terminal:

```bash
curl http://localhost:4000/courses
```

**Output:**

```text
[{"courseid":"2","coursename":"ReactJS","price":299,"author":{"fullname":"Hitesh Choudhary","website":"lco.dev"}}]
```

## Gotchas and Best Practices
> [!tip] Return `[]`, not `null`, for an empty list
> Front-end code usually expects an array. Initialise the fake database as `courses = []Course{}` (or `make([]Course, 0)`) if you want an empty response to be `[]`.

> [!warning] Headers after the body are lost
> Calling `w.Header().Set` after `w.Write` or `Encode` has no effect, and the status code is also frozen at that moment. Set headers first, write last.

> [!note] `Encode` returns an error you may want to check
> The instructor ignores it. A failed encode (for example a value containing a channel or a function) writes nothing at all, so the client gets an empty `200` and no hint why. In production, check it or log it; see [[Error Handling in Go]].

> [!tip] Snippets save typing
> The instructor copies the signature line between controllers. A VS Code snippet for `func name(w http.ResponseWriter, r *http.Request) {}` is the usual time saver.

## Video Timeline
| Time | What happens |
|---|---|
| [00:03](https://youtu.be/V3S3hKrIdQo?t=3) | What a controller is, recapped from the `serveHome` handler in the modules video |
| [01:32](https://youtu.be/V3S3hKrIdQo?t=92) | Writing `serveHome`: `http.ResponseWriter`, `*http.Request`, and their order |
| [04:10](https://youtu.be/V3S3hKrIdQo?t=250) | `getAllCourses` and setting the `Content-Type` header |
| [05:53](https://youtu.be/V3S3hKrIdQo?t=353) | `json.NewEncoder(w).Encode(courses)` streams JSON straight into the response |
| [06:56](https://youtu.be/V3S3hKrIdQo?t=416) | Why the response is empty for now, and what seeding means |

## Related Notes
- [[Building an API - Models]] — the `Course` struct and the `courses` slice this controller sends.
- [[Get One Course Controller]] — the next controller, which reads a route parameter.
- [[Routes and Testing the API]] — where `getAllCourses` is attached to `GET /courses` and the data is seeded.
- [[Creating JSON Data]] — `json.Marshal` versus the encoder used here.
- [[Go Modules]] — the first `serveHome` and the router setup.
- [[Working with Files]] — `Write` with a byte slice, the same pattern as writing to a file.
- [[Error Handling in Go]] — what to do with the error `Encode` returns.
- [[MongoDB API Controllers]] — the same controllers moved into their own package.
- [[Go Quick Reference]] — syntax cheat sheet.
