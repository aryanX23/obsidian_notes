---
tags:
  - golang
  - mongodb
section: "07 API with MongoDB"
video:
  - https://www.youtube.com/watch?v=Ycc9sXgDCNQ
  - https://www.youtube.com/watch?v=XdYpKfIgIQg
  - https://www.youtube.com/watch?v=Qh1H-6Ebz7E
video_title: "Get all movies from DB in golang / Mark movie as watched in golang / Delete 1 and all movie in golang"
order: 48
---
# MongoDB API Controllers

> [!info] Source
> 🎬 [Get all movies from DB in golang](https://www.youtube.com/watch?v=Ycc9sXgDCNQ) · 🎬 [Mark movie as watched in golang](https://www.youtube.com/watch?v=XdYpKfIgIQg) · 🎬 [Delete 1 and all movie in golang](https://www.youtube.com/watch?v=Qh1H-6Ebz7E) · Section **07 API with MongoDB** of [[Golang Notes]]
> ⬅️ [[Get All Documents from MongoDB]] · ➡️ [[MongoDB API Routes and Testing]]

## Overview

All the database helpers exist: insert, update, delete one, delete all, read all. None of them knows anything about HTTP. These three short videos add the second layer from the plan in [[Insert Data in MongoDB]]: the **controllers**, one per route, that read the request, call a helper and write a JSON response. They look exactly like the handlers of the slice-based API in [[Get All Courses Controller]] and friends, which is the point: the HTTP side of a Go API does not change when the storage changes. The only new ideas are response headers, and why these functions must be **exported**.

## Two layers, two naming rules

| Layer | Names | Visibility | Examples |
|---|---|---|---|
| Database helpers | lower-case | unexported: usable only inside `package controller` | `insertOneMovie`, `updateOneMovie`, `deleteOneMovie`, `deleteAllMovie`, `getAllMovies` |
| Controllers | Capitalised | exported: the `router` package imports them | `GetMyAllMovies`, `CreateMovie`, `MarkAsWatched`, `DeleteAMovie`, `DeleteAllMovies` |

The helpers were never meant to leave the package, so they start with a lower-case letter. The controllers will be referenced from `router/router.go` in [[MongoDB API Routes and Testing]], so they must start with a capital letter (see [[Variables, Types and Constants]] for the exported-names rule and [[Go Modules]] for cross-package imports). Go is case-sensitive, so `getAllMovies` and `GetAllMovies` could coexist, but the instructor finds that confusing and names the controller `GetMyAllMovies` instead, admitting it is not a great naming convention.

## The handler signature

Every controller has the shape the `net/http` package demands:

```go
func CreateMovie(w http.ResponseWriter, r *http.Request)
```

- `w`, the **writer**, is where headers and the body go.
- `r`, the **request**, is a pointer because it carries everything about the incoming call (body, URL, headers) and you never want a copy of that.

The same signature was explained in [[Routes and Testing the API]]; with gorilla/mux the URL variables are read through `mux.Vars(r)`, as in [[Get One Course Controller]].

## Response headers

`w.Header()` returns an `http.Header`, a map from header name to values, with helper methods:

| Method | Effect |
|---|---|
| `Set(key, value)` | replaces any existing value; the one you will use almost always |
| `Add(key, value)` | appends another value for the same key |
| `Get(key)` | first value for the key |
| `Del(key)` | removes the key |
| `Clone()` | copies the whole header map |

The video sets two headers on every controller:

```go
w.Header().Set("Content-Type", "application/x-www-form-urlencode")
w.Header().Set("Allow-Control-Allow-Methods", "POST")
```

Both lines are **wrong as written**, and the mistakes are instructive:

1. **Content type.** `application/x-www-form-urlencoded` (with a trailing *d*, which the video also drops) describes HTML form bodies such as the ones sent in [[Sending Form Data]]. Every controller here writes JSON, so the honest header is `application/json`. Thunder Client and Postman render the body either way, which is why the mistake never shows up during testing; a browser or a strict client would not parse the response as JSON.
2. **CORS header.** The real header is `Access-Control-Allow-Methods`. It belongs to **CORS** (Cross-Origin Resource Sharing), the browser rule that a page served from one origin may only call an API on another origin if the API says so. Browsers ignore unknown header names, so the misspelt one does nothing; Thunder Client ignores CORS headers entirely. The instructor also notes that gorilla/mux can set such headers for all routes at once, which is what middleware is for.

The corrected pair, used in the example at the end of this note:

```go
w.Header().Set("Content-Type", "application/json")
w.Header().Set("Access-Control-Allow-Methods", "POST")
```

> [!warning] Copy-paste the headers carefully
> Each controller advertises the verb it expects: `GET` for reading, `POST` for creating, `PUT` for marking as watched, `DELETE` for the two deletes. The instructor copies the header block from controller to controller and warns that leaving `POST` in a delete handler is exactly the kind of "tricky error" that copy-paste produces.

## The five controllers

### `GetMyAllMovies` (video 48)

```go
func GetMyAllMovies(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/x-www-form-urlencode")
	allMovies := getAllMovies()
	json.NewEncoder(w).Encode(allMovies)
}
```

Call the helper from [[Get All Documents from MongoDB]], encode the `[]primitive.M` straight into the writer. `json.NewEncoder(w).Encode(v)` is the streaming form of `json.Marshal` from [[Creating JSON Data]]: it writes the JSON directly to the response. If you wanted to strip the ids or the `watched` flag before answering, this is the place to loop over `allMovies` first.

### `CreateMovie` (video 49)

```go
func CreateMovie(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/x-www-form-urlencode")
	w.Header().Set("Allow-Control-Allow-Methods", "POST")

	var movie model.Netflix
	_ = json.NewDecoder(r.Body).Decode(&movie)
	insertOneMovie(movie)
	json.NewEncoder(w).Encode(movie)
}
```

The plan is three lines long: declare a `model.Netflix` (see [[Defining Models for MongoDB]]), decode the request body into it with `json.NewDecoder(r.Body).Decode(&movie)` (the pointer lets the decoder fill your variable, as in [[Consuming JSON Data]]), hand it to `insertOneMovie` from [[Insert Data in MongoDB]]. The response simply echoes the decoded struct. That has a visible consequence in the next video: the client sent no `_id`, the struct's `ID` is still the zero value, and the response shows it as 24 zeros:

```json
{"_id":"000000000000000000000000","movie":"Deadpool"}
```

`watched` is missing from that output because `false` is the zero value and the field has `omitempty`. The generated id exists only on the server console, printed by the helper (`Inserted 1 movie in db with id: ObjectID("...")`). The example below fixes both.

### `MarkAsWatched` (video 49)

```go
func MarkAsWatched(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/x-www-form-urlencode")
	w.Header().Set("Allow-Control-Allow-Methods", "PUT")

	params := mux.Vars(r)
	updateOneMovie(params["id"])
	json.NewEncoder(w).Encode(params["id"])
}
```

The id comes from the URL (`/api/movie/{id}` once the routes exist), so `mux.Vars(r)` returns a `map[string]string` and `params["id"]` is the string the helper from [[Update a Record in MongoDB]] expects. The response is just the id, encoded as a JSON string (`"652f1a2b3c4d5e6f7a8b9c0d"`); the instructor admits this is "not very thought out" but it proves the round trip works.

### `DeleteAMovie` and `DeleteAllMovies` (video 50)

```go
func DeleteAMovie(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/x-www-form-urlencode")
	w.Header().Set("Allow-Control-Allow-Methods", "DELETE")

	params := mux.Vars(r)
	deleteOneMovie(params["id"])
	json.NewEncoder(w).Encode(params["id"])
}

func DeleteAllMovies(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/x-www-form-urlencode")
	w.Header().Set("Allow-Control-Allow-Methods", "DELETE")

	count := deleteAllMovie()
	json.NewEncoder(w).Encode(count)
}
```

`deleteOneMovie` from [[Delete One and Delete Many in MongoDB]] only prints its count, so the controller has nothing better to return than the id it was given. `deleteAllMovie` was written to **return** the `int64` count, so `DeleteAllMovies` can answer with a number (`3` in the test run of the next video). The instructor suggests pausing and writing these two yourself before watching: they are the same pattern a third time.

## Example: the same controllers with proper headers and error handling

What changes when the helpers return errors instead of calling `log.Fatal`, and the controllers map those errors to HTTP status codes. Only the two most interesting controllers are shown; the delete ones follow `MarkAsWatched`.

```go
package controller

import (
	"context"
	"encoding/json"
	"net/http"

	"github.com/gorilla/mux"
	"github.com/hiteshchoudhary/mongoapi/model"
	"go.mongodb.org/mongo-driver/bson/primitive"
)

// helpers rewritten to report instead of exit (full bodies in the Delete and Update notes)
func insertMovie(ctx context.Context, movie model.Netflix) (primitive.ObjectID, error) { /* ... */ return primitive.NewObjectID(), nil }
func markWatched(ctx context.Context, hexID string) (matched int64, err error)        { /* ... */ return 1, nil }

func writeJSON(w http.ResponseWriter, status int, v interface{}) {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(status)
	json.NewEncoder(w).Encode(v)
}

func CreateMovie(w http.ResponseWriter, r *http.Request) {
	var movie model.Netflix
	if err := json.NewDecoder(r.Body).Decode(&movie); err != nil {
		http.Error(w, "body must be JSON with a movie field", http.StatusBadRequest)
		return
	}
	if movie.Movie == "" {
		http.Error(w, "movie name is required", http.StatusBadRequest)
		return
	}
	id, err := insertMovie(r.Context(), movie)
	if err != nil {
		http.Error(w, "could not save the movie", http.StatusInternalServerError)
		return
	}
	movie.ID = id // the response now carries the real id, not 24 zeros
	writeJSON(w, http.StatusCreated, movie)
}

func MarkAsWatched(w http.ResponseWriter, r *http.Request) {
	id := mux.Vars(r)["id"]
	matched, err := markWatched(r.Context(), id)
	switch {
	case err != nil:
		http.Error(w, err.Error(), http.StatusBadRequest) // invalid id (a driver failure would deserve a 500 instead)
	case matched == 0:
		http.Error(w, "no movie with id "+id, http.StatusNotFound)
	default:
		writeJSON(w, http.StatusOK, map[string]interface{}{"id": id, "watched": true})
	}
}
```

Responses from this version, with the status codes a client can act on:

```text
POST /api/movie   {"movie":"Deadpool"}     -> 201 {"_id":"652f1a2b3c4d5e6f7a8b9c0d","movie":"Deadpool"}
POST /api/movie   {}                       -> 400 movie name is required
PUT  /api/movie/652f1a2b3c4d5e6f7a8b9c0d  -> 200 {"id":"652f1a2b3c4d5e6f7a8b9c0d","watched":true}
PUT  /api/movie/not-an-id                 -> 400 invalid movie id "not-an-id": the provided hex string is not a valid ObjectID
```

`http.Error` writes a plain-text body with the given status; `r.Context()` passes the request's context to the driver so a cancelled request also cancels the database call (see [[Connecting to MongoDB]]).

## Gotchas and Best Practices

> [!warning] `_ = json.NewDecoder(r.Body).Decode(&movie)` hides bad input
> An empty or malformed body leaves `movie` as the zero struct, and `insertOneMovie` happily stores a movie with an empty name. Check the error and the required fields before touching the database (see [[Error Handling in Go]]).

> [!warning] The response echoes what you decoded, not what you stored
> `CreateMovie` returns the struct before the database assigned an id, hence the zeros. Capture `InsertedID` from the helper and put it into `movie.ID` before encoding.

> [!tip] Write the status code before the body
> `w.WriteHeader(http.StatusCreated)` must come before the first `Write`; after the body starts, the status is fixed at `200`. The `writeJSON` helper above keeps that order in one place.

> [!note] Headers are case-insensitive, values are not
> `Set` canonicalises the key (`content-type` becomes `Content-Type`), but it cannot fix a typo in the name or the value. Spell `application/json` and `Access-Control-Allow-Methods` exactly.

> [!tip] One controller, one helper
> Every controller does three things in order: read input, call one helper, write output. If a controller needs two helpers, write a new helper that does both; it keeps the HTTP layer trivial to test.

## Modern Go Notes

- Since Go 1.22 the standard `http.NewServeMux` matches methods and path variables itself (`"PUT /api/movie/{id}"`, then `r.PathValue("id")`), so these controllers work without gorilla/mux; the next note shows the router both ways.
- `http.Error`, `w.WriteHeader` and `r.Context()` have been in the standard library since Go 1.0 and 1.7; nothing here needs a third-party package.
- Driver v2 renames `primitive.ObjectID` to `bson.ObjectID` (verify in the v2 documentation); the controller code itself is unaffected.

## Video Timeline

**Video 48, Get all movies from DB**

| Time | What happens |
|---|---|
| [00:31](https://youtu.be/Ycc9sXgDCNQ?t=31) | Helpers stay lower-case; controllers must be exported for the router package |
| [01:48](https://youtu.be/Ycc9sXgDCNQ?t=108) | Naming the controller `GetMyAllMovies` to avoid confusion with `getAllMovies` |
| [02:13](https://youtu.be/Ycc9sXgDCNQ?t=133) | The `(w http.ResponseWriter, r *http.Request)` signature |
| [02:53](https://youtu.be/Ycc9sXgDCNQ?t=173) | `w.Header().Set("Content-Type", ...)` |
| [03:39](https://youtu.be/Ycc9sXgDCNQ?t=219) | Calling `getAllMovies()` and getting a `[]primitive.M` |
| [04:25](https://youtu.be/Ycc9sXgDCNQ?t=265) | `json.NewEncoder(w).Encode(allMovies)` |

**Video 49, Mark movie as watched**

| Time | What happens |
|---|---|
| [00:13](https://youtu.be/XdYpKfIgIQg?t=13) | Plan for `CreateMovie`: decode JSON into `model.Netflix`, call the helper |
| [01:11](https://youtu.be/XdYpKfIgIQg?t=71) | `Header()` methods: `Set` versus `Add`, `Clone`, `Del` |
| [02:00](https://youtu.be/XdYpKfIgIQg?t=120) | The allow-methods header and what it controls |
| [03:03](https://youtu.be/XdYpKfIgIQg?t=183) | `var movie model.Netflix` and `json.NewDecoder(r.Body).Decode(&movie)` |
| [03:55](https://youtu.be/XdYpKfIgIQg?t=235) | `insertOneMovie(movie)` and echoing the movie back |
| [04:48](https://youtu.be/XdYpKfIgIQg?t=288) | `MarkAsWatched`: `mux.Vars(r)` and `updateOneMovie(params["id"])` |
| [06:29](https://youtu.be/XdYpKfIgIQg?t=389) | Encoding the id as the response |

**Video 50, Delete 1 and all movie**

| Time | What happens |
|---|---|
| [00:35](https://youtu.be/Qh1H-6Ebz7E?t=35) | `DeleteAMovie`, capital D so it is exported |
| [01:05](https://youtu.be/Qh1H-6Ebz7E?t=65) | The copy-paste trap: the header should say `DELETE` (and `PUT` for the update) |
| [01:39](https://youtu.be/Qh1H-6Ebz7E?t=99) | `mux.Vars(r)["id"]` into `deleteOneMovie` |
| [02:21](https://youtu.be/Qh1H-6Ebz7E?t=141) | Encoding the id back because the helper only prints its count |
| [02:42](https://youtu.be/Qh1H-6Ebz7E?t=162) | `DeleteAllMovies` returns the count from `deleteAllMovie` |
| [03:51](https://youtu.be/Qh1H-6Ebz7E?t=231) | No guarantee these run yet: testing comes after the router |

## Related Notes

- [[Insert Data in MongoDB]] — the helper/controller strategy and `insertOneMovie`
- [[Update a Record in MongoDB]], [[Delete One and Delete Many in MongoDB]], [[Get All Documents from MongoDB]] — the helpers each controller calls
- [[MongoDB API Routes and Testing]] — the routes that give these controllers URLs, and the test run
- [[Get All Courses Controller]] and [[Create Course Controller]] — the same handlers over an in-memory slice
- [[Consuming JSON Data]] and [[Creating JSON Data]] — `Decode` into a struct, `Encode` out of one
- [[Defining Models for MongoDB]] — `model.Netflix` and why `watched:false` vanishes from responses
- [[Sending Form Data]] — the content type the video's header actually describes
- [[Error Handling in Go]] — turning helper errors into status codes
- [[Go Quick Reference]] — the controller skeleton in one block
