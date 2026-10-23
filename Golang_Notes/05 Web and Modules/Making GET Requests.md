---
tags:
  - golang
  - web
section: "05 Web and Modules"
video: https://www.youtube.com/watch?v=V-sxFQ0fWlw
video_title: "How to make GET request in golang"
order: 28
---
# Making GET Requests

> [!info] Source
> 🎬 [How to make GET request in golang](https://www.youtube.com/watch?v=V-sxFQ0fWlw) · Section **05 Web and Modules** of [[Golang Notes]]
> ⬅️ [[Creating a Test Web Server]] · ➡️ [[Making POST Requests with JSON]]

## Overview

With the Express test server from [[Creating a Test Web Server]] running on `localhost:8000`, the series now sends data to a backend in the three ways every web developer meets: a **GET** request (this note), a **POST** with a JSON body ([[Making POST Requests with JSON]]) and a **POST** with URL-encoded form data ([[Sending Form Data]]). All three functions live in the same `main.go` (`21webreqverbs` in the companion repository) and are switched on from `main` one at a time.

The GET itself is the `http.Get` call from [[Handling Web Requests]], now wrapped in its own function and extended with two pieces of bookkeeping, the **status code** and the **content length**. The second half of the video introduces `strings.Builder`, an alternative to `string(bytes)` that you will see in a lot of Go code and should be able to read.

## Organising the code into functions

```go
func main() {
	fmt.Println("Welcome to web verb video - LCO")
	PerformGetRequest()
	// PerformPostJsonRequest()
	// PerformPostFormRequest()
}

func PerformGetRequest() {
	const myurl = "http://localhost:8000/get"
	// ...
}
```

- Each verb gets a function so it can be reused and tested on its own (see [[Functions]]).
- `PerformGetRequest` starts with a capital letter, which makes it **exported** (public outside the package). A lower-case `performGetRequest` would be private; the instructor says either is fine here.
- The URL is a constant inside the function. The instructor's exercise: make the function take the URL as a `string` parameter instead of hard-coding it.
- Typos in the URL are the number one reason "nothing works": `localhost:8000/get` must match the server's port and route exactly. A wrong port gives `connection refused`; a wrong path gives a `404` with a small HTML body (`Cannot GET /wrongpath`).

## The request and its bookkeeping

```go
response, err := http.Get(myurl)
if err != nil {
	panic(err)
}
defer response.Body.Close()

fmt.Println("Status code: ", response.StatusCode)
fmt.Println("Content length is: ", response.ContentLength)
```

- `StatusCode` is the integer (`200`) you branch on with [[If Else]]: proceed on `200`, handle `400`/`500` differently.
- `ContentLength` is the size of the body in bytes as announced by the server's `Content-Length` header. For `/get` it is `43`, the length of `{"message":"Hello from learnCodeonline.in"}`. It is `-1` when the server streams the body without announcing a size.
- `defer response.Body.Close()` comes right after the error check, as always.

## Two ways to turn the body into text

### Way 1: `string(bytes)`

```go
content, _ := io.ReadAll(response.Body)
fmt.Println(string(content))
```

Printing `content` without the conversion shows the raw bytes, the same gotcha as in [[Working with Files]]:

```text
[123 34 109 101 115 115 97 103 101 34 58 34 72 101 108 108 111 32 ...]
{"message":"Hello from learnCodeonline.in"}
```

(`123` is `{`, `34` is `"`, `109` is `m`.) Wrapping the slice in `string(...)` is the beginner-friendly way and there is nothing wrong with it.

### Way 2: `strings.Builder`

`string` is a type; `strings` is a package full of helpers (`Split`, `ToUpper`, `TrimSpace`, `Contains`, ...). One of its types is `Builder`, described in the docs as "used to efficiently build a string using Write methods" while minimising memory copying:

```go
var responseString strings.Builder
content, _ := io.ReadAll(response.Body)
byteCount, _ := responseString.Write(content)

fmt.Println("ByteCount is: ", byteCount)
fmt.Println(responseString.String())
```

- The zero value of `strings.Builder` is ready to use; no constructor needed.
- `Write(content)` appends the bytes and returns how many it wrote (`43`, matching `ContentLength`) plus an error that is always `nil` for a Builder.
- The text is not printed by `Write`; it is held inside the builder until you call `String()`.

Why bother? Because a builder can **accumulate** many pieces (several reads, a header line, a footer) without creating a new string on each `+`, and it exposes extra controls:

| Method | Purpose |
|---|---|
| `Write([]byte)`, `WriteString(string)`, `WriteRune(rune)`, `WriteByte(byte)` | append data in different forms |
| `String()` | the accumulated text |
| `Len()` | bytes accumulated so far |
| `Reset()` | empty the builder for reuse |
| `Grow(n)` | pre-allocate `n` bytes when you know the final size |

> [!tip] Which one to use?
> For a single response body, `string(content)` is shorter and perfectly idiomatic; the instructor goes back to it for the rest of the series. Reach for `strings.Builder` when you build a string in a loop or from many fragments. `Builder` also satisfies `io.Writer`, so you can `fmt.Fprintf(&sb, ...)` into it.

## Example: GET `/get` both ways

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"strings"
)

func main() {
	fmt.Println("Welcome to web verb video - LCO")
	PerformGetRequest()
}

func PerformGetRequest() {
	const myurl = "http://localhost:8000/get"

	response, err := http.Get(myurl)
	if err != nil {
		panic(err)
	}
	defer response.Body.Close()

	fmt.Println("Status code: ", response.StatusCode)
	fmt.Println("Content length is: ", response.ContentLength)

	// Way 1: read the bytes and convert them with string()
	// content, _ := io.ReadAll(response.Body)
	// fmt.Println(string(content))

	// Way 2: collect the bytes in a strings.Builder
	var responseString strings.Builder
	content, _ := io.ReadAll(response.Body)
	byteCount, _ := responseString.Write(content)

	fmt.Println("ByteCount is: ", byteCount)
	fmt.Println(responseString.String())
}
```

**Output (with the test server running on port 8000):**
```text
Welcome to web verb video - LCO
Status code:  200
Content length is:  43
ByteCount is:  43
{"message":"Hello from learnCodeonline.in"}
```

The body is JSON, but at this point it is handled as plain text. Turning it into a Go value (`map[string]interface{}` or a struct) is the subject of [[Consuming JSON Data]].

### Exercise: take the URL as a parameter

```go
func performGetRequest(url string) (string, error) {
	response, err := http.Get(url)
	if err != nil {
		return "", err
	}
	defer response.Body.Close()

	content, err := io.ReadAll(response.Body)
	if err != nil {
		return "", err
	}
	return string(content), nil
}
```

Returning `(string, error)` instead of panicking makes the function usable from any caller, the pattern recommended in [[Error Handling in Go]].

## Gotchas and Best Practices

> [!warning] The video discards the error from `ReadAll`
> `content, _ := io.ReadAll(...)` ignores a possible read failure (connection dropped mid-body). The instructor admits it and asks you to handle it; the exercise above does.

> [!warning] `ContentLength` can be `-1`
> Do not use it to size buffers blindly; `io.ReadAll` already grows as needed.

> [!warning] `connection refused` means the server is not running
> Start the Express server first, and check the port. A `404` means the server is up but the path is wrong.

> [!tip] Check `StatusCode` before reading the body
> An error page from the server is still a body; reading it as if it were the expected JSON leads to confusing parse errors later.

> [!tip] One `http.Client` for the whole program
> `http.Get` uses `http.DefaultClient`, which has no timeout. See the improved client in [[Handling Web Requests]].

## Modern Go Notes

- `ioutil.ReadAll` (used in the video) is deprecated; the examples use `io.ReadAll`.
- `strings.Builder` has been in the standard library since Go 1.10 and is unchanged; its docs warn "do not copy a non-zero Builder", so pass it by pointer if a helper needs to write into it.

## Video Timeline

| Time | What happens |
|---|---|
| [00:36](https://youtu.be/V-sxFQ0fWlw?t=36) | The plan: three videos for GET, POST with JSON and POST with form data |
| [01:52](https://youtu.be/V-sxFQ0fWlw?t=112) | `PerformGetRequest` as its own function; capital `P` exports it |
| [02:04](https://youtu.be/V-sxFQ0fWlw?t=124) | `const myurl = "http://localhost:8000/get"`; beware typos |
| [02:55](https://youtu.be/V-sxFQ0fWlw?t=175) | `http.Get`, the error check and `defer response.Body.Close()` |
| [04:03](https://youtu.be/V-sxFQ0fWlw?t=243) | `response.StatusCode` |
| [04:57](https://youtu.be/V-sxFQ0fWlw?t=297) | `response.ContentLength` |
| [06:14](https://youtu.be/V-sxFQ0fWlw?t=374) | First run: status `200`, length `43`, the message as a string |
| [06:54](https://youtu.be/V-sxFQ0fWlw?t=414) | The `strings` package and `strings.Builder` |
| [08:15](https://youtu.be/V-sxFQ0fWlw?t=495) | `responseString.Write(content)` returns a byte count of 43 |
| [10:02](https://youtu.be/V-sxFQ0fWlw?t=602) | `responseString.String()` returns the text |
| [11:03](https://youtu.be/V-sxFQ0fWlw?t=663) | Other Builder methods: `WriteString`, `WriteRune`, `Reset`, `Len`, `Grow` |

## Related Notes

- [[Creating a Test Web Server]] — the `/get` route this code talks to.
- [[Handling Web Requests]] — the first `http.Get`, the `Response` struct and why `Body` must be closed.
- [[Making POST Requests with JSON]] and [[Sending Form Data]] — the other two functions in the same file.
- [[Consuming JSON Data]] — decoding the `{"message": ...}` body instead of printing it.
- [[Working with Files]] — the same bytes-to-string conversion for file contents.
- [[Functions]] — exported versus unexported names, parameters and multiple return values.
- [[Slices]] — `[]byte` and why `Write` reports a byte count.
- [[Error Handling in Go]] — returning errors instead of panicking.
- [[fmt Package and Format Verbs]] — `Println` with several arguments.
- [[Go Quick Reference]] · [[Golang Notes]]
