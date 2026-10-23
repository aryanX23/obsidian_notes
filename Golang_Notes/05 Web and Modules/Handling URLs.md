---
tags:
  - golang
  - web
section: "05 Web and Modules"
video: https://www.youtube.com/watch?v=cl7_ouTMFh0
video_title: "Handling URL in golang"
order: 26
---
# Handling URLs

> [!info] Source
> 🎬 [Handling URL in golang](https://www.youtube.com/watch?v=cl7_ouTMFh0) · Section **05 Web and Modules** of [[Golang Notes]]
> ⬅️ [[Handling Web Requests]] · ➡️ [[Creating a Test Web Server]]

## Overview

Every backend has to pull information out of URLs: which course was requested, which payment id came back from a gateway, which page number the client wants. Frameworks such as Gorilla Mux (used from [[Go Modules]] onwards) and Fiber hide this behind helpers, but the instructor insists on learning the raw tool first: the standard library's `net/url` package. The video goes in both directions, **parsing** a URL into its parts and **building** a URL from parts, and introduces `url.Values`, a type that reappears in [[Sending Form Data]].

## Anatomy of a URL

The video builds a fictitious URL that has every interesting piece: a scheme, a host, a port (as you would get on a cloud instance), a path and two query parameters.

```text
https://lco.dev:3000/learn?coursename=reactjs&paymentid=ghbj456ghb
└─┬─┘   └──┬──┘└─┬─┘└─┬──┘└───────────────────┬──────────────────┘
scheme    host  port path               query (RawQuery)
```

| Part | Example | Notes |
|---|---|---|
| Scheme | `https` | The protocol; `mongodb+srv` in [[Connecting to MongoDB]] is a scheme too |
| Host | `lco.dev:3000` | Domain **and** port together |
| Port | `3000` | Optional; obtained with the `Port()` method |
| Path | `/learn` | The route on the server; `/login`, `/api/movies`, ... |
| Query | `coursename=reactjs&paymentid=ghbj456ghb` | `key=value` pairs separated by `&`, after the `?` |
| Fragment | `#section` | Not in the example; browser-only, never sent to the server |

Query parameters are what JavaScript developers call *params*; the values are always strings on the wire.

## Parsing a URL

```go
result, err := url.Parse(myurl)
```

`url.Parse` returns a pointer to a `url.URL` struct and an error (for a malformed URL such as `url.Parse("://lco.dev")`, which fails with `missing protocol scheme`; the instructor's live typo, a missing colon after `https`, does *not* error: Go silently parses `https//lco.dev:3000/...` as a relative path with an empty `Scheme`). Once parsed, the parts are plain struct fields (see [[Structs]]):

```go
fmt.Println(result.Scheme)   // https
fmt.Println(result.Host)     // lco.dev:3000
fmt.Println(result.Path)     // /learn
fmt.Println(result.Port())   // 3000  <- a METHOD, not a field
fmt.Println(result.RawQuery) // coursename=reactjs&paymentid=ghbj456ghb
```

> [!warning] `Port()` has parentheses
> `Scheme`, `Host`, `Path` and `RawQuery` are fields; `Port()` and `Hostname()` are methods that split `Host` for you. The instructor calls this "very easy to miss", and what you get when you forget the parentheses is confusing the first time: `fmt.Println(result.Port)` compiles and prints a hex address such as `0x104a08c20` (the function value itself), while `go vet` warns `fmt.Println arg result.Port is a func value, not called`.

Useful members of `url.URL`:

| Member | Kind | Returns |
|---|---|---|
| `Scheme`, `Host`, `Path`, `RawQuery`, `Fragment` | fields | the raw parts |
| `User` | field | `*url.Userinfo` for `user:pass@host` URLs |
| `Hostname()` | method | `Host` without the port |
| `Port()` | method | the port without the colon, `""` if none |
| `Query()` | method | `RawQuery` decoded into `url.Values` |
| `String()` | method | the URL reassembled into text |

## Query parameters as `url.Values`

`RawQuery` is one long string. `result.Query()` turns it into something you can index:

```go
qparams := result.Query()
fmt.Printf("The type of query params are: %T\n", qparams) // url.Values
```

`url.Values` is declared as `map[string][]string`: each key maps to a **slice** of values, because a key may legally appear more than once (`?tag=go&tag=web`). Everything you know about [[Maps]] applies:

```go
fmt.Println(qparams["coursename"])    // [reactjs]      <- the whole slice
fmt.Println(qparams.Get("paymentid")) // ghbj456ghb     <- just the first value

for key, val := range qparams {
    fmt.Println("Param:", key, "=", val)
}
```

Indexing the map gives `[]string`, which prints inside square brackets; `Get` returns the first value as a plain string (or `""` if the key is absent). Ranging over the map visits keys in **no guaranteed order**, which the instructor points out when his output comes back shuffled (see [[Loops, Break, Continue and Goto]]).

Other helpers on `url.Values`: `Add(key, value)` appends, `Set(key, value)` replaces, `Del(key)`, `Has(key)`, and `Encode()` serialises the map back into `a=1&b=2` form with proper escaping.

## Building a URL from parts

The reverse direction is just as common: you have pieces in variables and need a full URL. You could concatenate strings, but `url.URL` does the escaping and the punctuation for you:

```go
partsOfUrl := &url.URL{
    Scheme:   "https",
    Host:     "lco.dev",
    Path:     "/tutcss",
    RawQuery: "user=hitesh",
}

anotherURL := partsOfUrl.String()
fmt.Println(anotherURL) // https://lco.dev/tutcss?user=hitesh
```

Three things the instructor stresses:

- The literal is written with `&url.URL{...}`, a **pointer** to the struct (see [[Pointers]]). The `String` method is defined on `*URL`, so working through a pointer is the natural form. (Go would take the address automatically for an addressable variable, but writing `&` makes the intent explicit.)
- The field names are fixed: `Scheme`, not `scheme` or `protocol`. Capitalised names are exported fields, as in every struct literal.
- `Scheme` has no `://`; `String()` adds the punctuation.

## Example: parse, then rebuild

```go
package main

import (
	"fmt"
	"net/url"
)

const myurl string = "https://lco.dev:3000/learn?coursename=reactjs&paymentid=ghbj456ghb"

func main() {
	fmt.Println("Welcome to handling URLs in Go")
	fmt.Println(myurl)

	// parsing: turn the string into a *url.URL with named parts
	result, err := url.Parse(myurl)
	if err != nil {
		panic(err)
	}

	fmt.Println("Scheme:   ", result.Scheme)
	fmt.Println("Host:     ", result.Host)
	fmt.Println("Path:     ", result.Path)
	fmt.Println("Port:     ", result.Port()) // a method, not a field
	fmt.Println("RawQuery: ", result.RawQuery)

	// query parameters come back as url.Values (map[string][]string)
	qparams := result.Query()
	fmt.Printf("The type of query params are: %T\n", qparams)

	fmt.Println(qparams["coursename"]) // the whole slice of values
	fmt.Println(qparams.Get("paymentid")) // just the first value

	for key, val := range qparams {
		fmt.Println("Param:", key, "=", val)
	}

	// building a URL from parts: note the pointer (&url.URL{...})
	partsOfUrl := &url.URL{
		Scheme:   "https",
		Host:     "lco.dev",
		Path:     "/tutcss",
		RawQuery: "user=hitesh",
	}

	anotherURL := partsOfUrl.String()
	fmt.Println(anotherURL)
}
```

**Output (the two `Param:` lines may swap places between runs):**
```text
Welcome to handling URLs in Go
https://lco.dev:3000/learn?coursename=reactjs&paymentid=ghbj456ghb
Scheme:    https
Host:      lco.dev:3000
Path:      /learn
Port:      3000
RawQuery:  coursename=reactjs&paymentid=ghbj456ghb
The type of query params are: url.Values
[reactjs]
ghbj456ghb
Param: coursename = [reactjs]
Param: paymentid = [ghbj456ghb]
https://lco.dev/tutcss?user=hitesh
```

The video ignores the error from `url.Parse` with `_` to save time; the version above handles it, which is what you should do in real code ([[Error Handling in Go]]).

## Extra example: safe query strings with `Encode` and `ParseQuery`

Hand-written query strings break as soon as a value contains a space, an `&` or a `=`. `url.Values.Encode` escapes everything correctly, and `url.ParseQuery` is the strict counterpart of `Query()` that reports errors:

```go
package main

import (
	"fmt"
	"net/url"
)

func main() {
	// Build query parameters safely: Encode() escapes spaces, &, = and so on
	params := url.Values{}
	params.Add("q", "go lang & web")
	params.Add("tag", "backend")
	params.Add("tag", "api") // same key twice is allowed

	u := &url.URL{Scheme: "https", Host: "example.com", Path: "/search"}
	u.RawQuery = params.Encode()
	fmt.Println(u.String())

	// Parse a raw query string back into url.Values
	values, err := url.ParseQuery("coursename=reactjs&paymentid=ghbj456ghb")
	if err != nil {
		panic(err)
	}
	fmt.Println(values.Get("coursename"), values.Has("paymentid"), values.Get("missing") == "")
}
```

**Output:**
```text
https://example.com/search?q=go+lang+%26+web&tag=backend&tag=api
reactjs true true
```

`Encode` sorts keys alphabetically, turns the space into `+` and the ampersand into `%26`. This is exactly the body format that `http.PostForm` sends in [[Sending Form Data]].

## Gotchas and Best Practices

> [!warning] `RawPath` is not the query string (a bug in the companion code)
> The instructor's repository builds the URL with `RawPath: "user=hitesh"`. `RawPath` is only an *encoding hint for the path* and is ignored unless it matches `Path`, so the query silently disappears:
> ```text
> RawPath : https://lco.dev/tutcss
> RawQuery: https://lco.dev/tutcss?user=hitesh
> ```
> The field for a query string is `RawQuery`, as used in the examples above.

> [!warning] `Host` includes the port
> `result.Host` is `lco.dev:3000`. Use `Hostname()` when you need only the domain, and `Port()` for the port.

> [!warning] `Query()` hides errors
> It "silently discards malformed value pairs". When the query string comes from an untrusted client and you need to know it was malformed, call `url.ParseQuery(result.RawQuery)` and check the error.

> [!tip] Prefer `Get` for single values, the map for repeated keys
> `qparams.Get("tag")` returns the first `tag`; `qparams["tag"]` returns all of them.

> [!tip] Inside an HTTP handler the parsed URL is already there
> `r.URL.Query().Get("page")` on an `*http.Request` uses exactly this machinery; the path variables (`/course/{id}`) are handled by the router in [[Get One Course Controller]].

## Modern Go Notes

Nothing in this video is deprecated. Two newer helpers are worth knowing:

- `url.JoinPath(base, elems...)` (Go 1.19+) joins path segments onto a base URL and cleans `./` and `../`, safer than string concatenation.
- `url.QueryEscape` and `url.PathEscape` escape a single value for use in a query or a path segment when you are not going through `url.Values`.

## Video Timeline

| Time | What happens |
|---|---|
| [00:13](https://youtu.be/cl7_ouTMFh0?t=13) | Why URL handling matters; `net` is a big module and `net/url` is one piece of it |
| [01:56](https://youtu.be/cl7_ouTMFh0?t=116) | Constructing the fictitious URL with scheme, host, port, path and query |
| [04:25](https://youtu.be/cl7_ouTMFh0?t=265) | `url.Parse(myurl)` and the result it returns |
| [05:56](https://youtu.be/cl7_ouTMFh0?t=356) | Extracting `Scheme`, `Host`, `Path`, `RawQuery`; `Port()` is a method |
| [07:46](https://youtu.be/cl7_ouTMFh0?t=466) | `result.Query()` gives `url.Values`, a key-value type |
| [09:15](https://youtu.be/cl7_ouTMFh0?t=555) | `qparams["coursename"]` prints `[reactjs]` |
| [09:49](https://youtu.be/cl7_ouTMFh0?t=589) | Ranging over the params; order is not guaranteed |
| [10:41](https://youtu.be/cl7_ouTMFh0?t=641) | Building a URL from parts with `&url.URL{...}`; it must be a reference |
| [12:37](https://youtu.be/cl7_ouTMFh0?t=757) | `.String()` assembles the final URL |

## Related Notes

- [[Handling Web Requests]] — the request that uses the URL you parse here.
- [[Sending Form Data]] — `url.Values` as the body of a `POST` with `http.PostForm`.
- [[Maps]] — `url.Values` is a map, with map iteration order rules.
- [[Slices]] — each value is a `[]string` because keys can repeat.
- [[Structs]] and [[Pointers]] — `url.URL` is a struct built through a pointer.
- [[Methods]] — `Port()`, `Query()` and `String()` are methods on `*url.URL`.
- [[Get One Course Controller]] — reading an `{id}` from the path with Gorilla Mux.
- [[Error Handling in Go]] — handle the error from `url.Parse` instead of discarding it.
- [[Go Quick Reference]] · [[Golang Notes]]
