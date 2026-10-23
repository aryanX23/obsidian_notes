---
tags:
  - golang
  - web
section: "05 Web and Modules"
video: https://www.youtube.com/watch?v=U_LjyX4iDbU
video_title: "How to send form data in golang"
order: 30
---
# Sending Form Data

> [!info] Source
> 🎬 [How to send form data in golang](https://www.youtube.com/watch?v=U_LjyX4iDbU) · Section **05 Web and Modules** of [[Golang Notes]]
> ⬅️ [[Making POST Requests with JSON]] · ➡️ [[Creating JSON Data]]

## Overview

JSON is what you will send most of the time, but some backends still expect the classic **HTML form** encoding, `application/x-www-form-urlencoded`, and the instructor mentions file uploads and similar flows as the typical reason to know it. Go makes this the shortest of the three requests: build the fields as a `url.Values` (the type from [[Handling URLs]]), hand it to `http.PostForm`, and the standard library encodes the body **and** sets the content type for you. The `/postform` route of the test server from [[Creating a Test Web Server]] echoes the fields back as JSON, so the result is easy to check.

## Building the form with `url.Values`

```go
data := url.Values{}
data.Add("firstname", "hitesh")
data.Add("lastname", "choudhary")
data.Add("email", "hitesh@go.dev")
```

- `url.Values{}` creates an **empty** map of type `map[string][]string` (see [[Maps]]). Its keys are the form field names and each value is a slice, because a field such as a multi-select can appear several times.
- `Add` appends a value; `Set` replaces all values for a key; `Get` reads the first one. In the video the values are hard-coded, but in a real program they come from variables, user input or a struct.
- Everything is a `string`. Numbers and booleans have to be formatted as text first (`strconv.Itoa`, see [[Type Conversions]]).

What goes on the wire is produced by `data.Encode()`:

```text
email=hitesh%40go.dev&firstname=hitesh&lastname=choudhary
```

Keys are sorted alphabetically, the `@` became `%40`, and a space would become `+`. This is exactly the query-string format from [[Handling URLs]], only placed in the request body instead of after a `?` in the URL.

## `http.PostForm`

```go
func PostForm(url string, data url.Values) (resp *Response, err error)
```

The documentation says it all: "PostForm issues a POST to the specified URL, with data's keys and values URL-encoded as the request body. The Content-Type header is set to application/x-www-form-urlencoded." Compared with `http.Post` from the previous note there is **no content type argument and no reader**: both are derived from `data`. The call is equivalent to

```go
http.Post(myurl, "application/x-www-form-urlencoded", strings.NewReader(data.Encode()))
```

which is handy to know when you need `http.NewRequest` for custom headers.

## Example: POST a form to `/postform`

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"net/url"
)

func main() {
	fmt.Println("Welcome to web verb video - LCO")
	PerformPostFormRequest()
}

func PerformPostFormRequest() {
	const myurl = "http://localhost:8000/postform"

	// form data: url.Values is a map[string][]string
	data := url.Values{}
	data.Add("firstname", "hitesh")
	data.Add("lastname", "choudhary")
	data.Add("email", "hitesh@go.dev")

	fmt.Println("Encoded body:", data.Encode())

	response, err := http.PostForm(myurl, data)
	if err != nil {
		panic(err)
	}
	defer response.Body.Close()

	content, _ := io.ReadAll(response.Body)
	fmt.Println(string(content))
}
```

**Output (with the test server running on port 8000):**
```text
Welcome to web verb video - LCO
Encoded body: email=hitesh%40go.dev&firstname=hitesh&lastname=choudhary
{"email":"hitesh@go.dev","firstname":"hitesh","lastname":"choudhary"}
```

The server decoded the URL-encoded body into an object and echoed it as a JSON string, so every field round-tripped intact. The `Encoded body` line is an addition to the video's code to make the wire format visible.

## The three ways to send data, side by side

| | GET ([[Making GET Requests]]) | POST JSON ([[Making POST Requests with JSON]]) | POST form (this note) |
|---|---|---|---|
| Data travels in | the URL (query string) | the body | the body |
| Content type | none | `application/json` | `application/x-www-form-urlencoded` |
| Go call | `http.Get(url)` | `http.Post(url, "application/json", reader)` | `http.PostForm(url, values)` |
| Payload built from | `url.Values` → `Encode()` into the URL | raw string or `json.Marshal` | `url.Values` |
| Typical use | fetch, search, filters | APIs, mobile and web apps | legacy forms, some OAuth and payment endpoints |
| Not suitable for | large or secret data | file uploads | file uploads (needs `multipart/form-data`) |

> [!note] File uploads are a fourth format
> Both JSON and URL-encoded forms are text. Binary uploads use `multipart/form-data`, which the standard library supports through the `mime/multipart` package. That is why the instructor mentions "image upload" when introducing form data: forms are the family those uploads belong to, even though this video only covers the URL-encoded member.

## What the server sees

The Express server reads the fields from `req.body` thanks to `express.urlencoded(...)`. A Go server does the equivalent with the request's form helpers:

```go
func handler(w http.ResponseWriter, r *http.Request) {
	if err := r.ParseForm(); err != nil {
		http.Error(w, "bad form", http.StatusBadRequest)
		return
	}
	first := r.FormValue("firstname") // "" if the field is missing
	fmt.Fprintln(w, "Hello", first)
}
```

`r.ParseForm` fills `r.PostForm` (body fields) and `r.Form` (body plus query string), both of type `url.Values` again. You will write handlers like this from [[Go Modules]] onwards.

## Gotchas and Best Practices

> [!warning] Form-encoded, not multipart
> In Thunder Client this is the "Form-encode" tab, not the plain "Form" tab; the test server ignores multipart bodies and echoes `{}`. `http.PostForm` always sends the URL-encoded form, so Go cannot make this mistake.

> [!warning] `Add` versus `Set`
> Calling `Add("email", ...)` twice sends the field twice. Use `Set` when a field must appear exactly once.

> [!warning] Values are strings only
> `url.Values` cannot hold an `int`; convert first, and expect the server to receive text.

> [!tip] `Encode` is a free debugging tool
> Print `data.Encode()` while developing to see precisely what the body will contain.

> [!tip] Same bookkeeping as always
> Check `err`, `defer response.Body.Close()`, read with `io.ReadAll`, then look at `StatusCode` before trusting the body.

## Modern Go Notes

- `ioutil.ReadAll` in the video is deprecated in favour of `io.ReadAll`.
- Nothing about `url.Values` or `http.PostForm` has changed. For custom headers, build the request yourself with `http.NewRequest(http.MethodPost, url, strings.NewReader(data.Encode()))` and set `Content-Type` to `application/x-www-form-urlencoded`.

## Video Timeline

| Time | What happens |
|---|---|
| [00:18](https://youtu.be/U_LjyX4iDbU?t=18) | When form data is needed: uploads and other special cases |
| [00:41](https://youtu.be/U_LjyX4iDbU?t=41) | `PerformPostFormRequest` with the `/postform` URL |
| [01:32](https://youtu.be/U_LjyX4iDbU?t=92) | `data := url.Values{}`: an empty key-value collection |
| [01:53](https://youtu.be/U_LjyX4iDbU?t=113) | `data.Add("firstname", "hitesh")` and the other fields |
| [02:39](https://youtu.be/U_LjyX4iDbU?t=159) | `http.PostForm(myurl, data)` |
| [03:05](https://youtu.be/U_LjyX4iDbU?t=185) | No content type to pass: `PostForm` sets it |
| [03:29](https://youtu.be/U_LjyX4iDbU?t=209) | Error check, `ReadAll`, and `defer response.Body.Close()` |
| [04:54](https://youtu.be/U_LjyX4iDbU?t=294) | The server echoes email, first name and last name |

## Related Notes

- [[Handling URLs]] — where `url.Values`, `Add`, `Get` and `Encode` were introduced.
- [[Making POST Requests with JSON]] — the same POST with a JSON body and explicit content type.
- [[Making GET Requests]] — the first of the three verb functions.
- [[Creating a Test Web Server]] — the `/postform` route and the form-data trap.
- [[Creating JSON Data]] — next: producing JSON from Go values instead of typing it.
- [[Maps]] — `url.Values` is a map of string slices.
- [[Type Conversions]] — turning numbers into the strings a form requires.
- [[Go Modules]] — the first Go server that would receive such a form.
- [[Defer]] and [[Error Handling in Go]] — closing the body and handling `err`.
- [[Go Quick Reference]] · [[Golang Notes]]
