---
tags:
  - golang
  - api
section: "06 Building an API"
video: https://www.youtube.com/watch?v=vMpBUkIMLzY
video_title: "Add a course controller in golang"
order: 37
---
# Create Course Controller

> [!info] Source
> 🎬 [Add a course controller in golang](https://www.youtube.com/watch?v=vMpBUkIMLzY) · Section **06 Building an API** of [[Golang Notes]]
> ⬅️ [[Get One Course Controller]] · ➡️ [[Update Course Controller]]

## Overview
So far JSON has only flowed **out** of the API. Creating a course reverses the direction: the client sends a JSON body with a `POST` request, the controller **decodes** it into a `Course`, validates it, gives it an id and appends it to the fake database. This is the first controller that has to defend itself against bad input, so the video spends time in "what-if mode": what if the body is empty, what if it is `{}`, what if the client sends an id of its own. It also hits a type error that leads to the `strconv` package, because the model's `CourseId` is a string while random numbers are integers.

## Decoding the request body
[[Consuming JSON Data]] used `json.Unmarshal` on a byte slice. The request body is a stream (`r.Body` is an `io.ReadCloser`), so the matching tool is a **decoder**:

```go
var course Course
_ = json.NewDecoder(r.Body).Decode(&course)
```

- `json.NewDecoder(r.Body)` wraps the body stream, the mirror image of `json.NewEncoder(w)` from [[Get All Courses Controller]].
- `Decode(&course)` reads one JSON value and fills the struct through the pointer (see [[Pointers]]); JSON keys are matched to fields through the struct tags from [[Building an API - Models]].
- `Decode` returns an error. The instructor discards it with `_` and relies on the `IsEmpty` check instead. In real code you would inspect it, because it also catches malformed JSON; see [[Error Handling in Go]].

## What-if mode: defending the controller
### What if the body is empty?
The instructor's first guard checks whether there is a body at all:

```go
// what if: body is empty
if r.Body == nil {
	json.NewEncoder(w).Encode("Please send some data")
	return // added: the companion code falls through here
}
```

> [!warning] This check never fires on a server
> The `net/http` documentation says: *"For server requests, the Request Body is always non-nil but will return EOF immediately when no body is present."* A `POST` without a body therefore skips this `if`; `Decode` returns `io.EOF`, leaves `course` zero-valued, and the **next** check catches it. The guard is harmless, but the real protection is `IsEmpty`. Note also that the companion code has no `return` inside this `if`, so even if it did fire, execution would continue.

### What if the body is `{}`?
An empty object is valid JSON, decodes without error, and produces a `Course` whose fields are all zero values. That is what the helper from [[Building an API - Models]] is for:

```go
// what about - {}
if course.IsEmpty() {
	json.NewEncoder(w).Encode("No data inside JSON")
	return
}
```

![[40_thunder_post_empty_json_1350.png]]
*Figure: Thunder Client sending `{}` to POST /course and receiving "No data inside JSON" (video at 13:50)*

### Changing `IsEmpty`
The original helper rejected a course only when **both** the id and the name were missing. The instructor now rewrites it to look at the name alone:

```go
func (c *Course) IsEmpty() bool {
	// return c.CourseId == "" && c.CourseName == ""
	return c.CourseName == ""
}
```

The reason is a design decision: the client should **not** supply ids. Clients cannot be trusted to produce unique values, so the server generates the id itself, and a missing id is no longer an error.

## Generating the id
Two problems remain, and the instructor lists them as comments:

1. Generate a unique id.
2. Convert it to a string, because `CourseId` is a string.

```go
rand.Seed(time.Now().UnixNano())
course.CourseId = rand.Intn(100)
```

The second line does not compile. `rand.Intn` (see [[Math, Crypto and Random Numbers]]) returns an `int`:

```text
cannot use rand.Intn(100) (value of type int) as string value in assignment
```

The fix is the `strconv` package, met in [[Type Conversions]] through `ParseFloat`. Its most-used function goes the other way, integer to ASCII:

```go
course.CourseId = strconv.Itoa(rand.Intn(100))
```

> [!warning] `string(42)` is not `"42"`
> It is tempting to write `string(rand.Intn(100))`. That conversion interprets the number as a Unicode code point and yields a one-character string (`string(65)` is `"A"`), and `go vet` warns about it. Integer-to-digits conversion is always `strconv.Itoa` or `fmt.Sprint`.

Had `CourseId` been an `int`, the random number could have been assigned directly. The instructor keeps the string on purpose so the series meets `strconv`.

## Appending and responding
With a validated course and a fresh id, the last steps are the `append` from [[Slices]] and an encoded reply:

```go
courses = append(courses, course)
json.NewEncoder(w).Encode(course)
return
```

The client gets the stored record back, including the id it did not know yet. The trailing `return` is redundant again; the instructor leaves it for clarity.

## Example: the complete `createOneCourse` controller
```go
func createOneCourse(w http.ResponseWriter, r *http.Request) {
	fmt.Println("Create one course")
	w.Header().Set("Content-Type", "application/json")

	// what if: body is empty
	if r.Body == nil {
		json.NewEncoder(w).Encode("Please send some data")
		return
	}

	// what about - {}
	var course Course
	_ = json.NewDecoder(r.Body).Decode(&course)
	if course.IsEmpty() {
		json.NewEncoder(w).Encode("No data inside JSON")
		return
	}

	// TODO: check only if title is duplicate
	// loop, title matches with course.coursename, JSON

	// generate unique id, string
	// append course into courses
	rand.Seed(time.Now().UnixNano()) // not needed since Go 1.20, see Modern Go Notes
	course.CourseId = strconv.Itoa(rand.Intn(100))
	courses = append(courses, course)
	json.NewEncoder(w).Encode(course)
}
```

Requests and responses, once the route `POST /course` exists ([[Routes and Testing the API]]):

```bash
curl -X POST http://localhost:4000/course
```

```text
"No data inside JSON"
```

```bash
curl -X POST http://localhost:4000/course -H 'Content-Type: application/json' -d '{}'
```

```text
"No data inside JSON"
```

```bash
curl -X POST http://localhost:4000/course -H 'Content-Type: application/json' \
  -d '{"coursename":"Angular JS","price":199,"author":{"fullname":"Telusko","website":"telusko.com"}}'
```

```json
{
  "courseid": "3",
  "coursename": "Angular JS",
  "price": 199,
  "author": { "fullname": "Telusko", "website": "telusko.com" }
}
```

The `courseid` is random, so it differs on every run (the instructor gets `7`). The JSON keys must match the struct tags exactly (`coursename`, `price`, `author.fullname`), which trips the instructor up during testing.

## Exercise: reject duplicate titles
The `TODO` in the code is an assignment: before appending, loop through `courses` and refuse the request if a course with the same name already exists. One possible solution, using the same loop shape as [[Get One Course Controller]]:

```go
for _, existing := range courses {
	if existing.CourseName == course.CourseName {
		json.NewEncoder(w).Encode("A course with this name already exists")
		return
	}
}
```

Place it after the `IsEmpty` check and before the id is generated.

## Gotchas and Best Practices
> [!warning] `rand.Intn(100)` is not unique
> Only 100 possible values, and the seeded courses already use `"2"` and `"4"`. A collision silently creates two courses with the same id, and `getOneCourse` would always return the first. Real services use a database-generated id, a counter, or a UUID.

> [!tip] Validate, then mutate
> Every early `return` happens before `append`. Keep that order so a rejected request never touches the data.

> [!tip] Reply with `201 Created`
> `w.WriteHeader(http.StatusCreated)` before `Encode` tells the client a new resource exists. The series always answers `200`.

> [!note] Keep the error messages in one place
> Production APIs standardise their messages (and usually return a JSON object such as `{"error": "..."}`) so the front end can rely on them. The free-form strings here are tutorial shortcuts.

## Modern Go Notes
- `rand.Seed` is **deprecated since Go 1.20**: the generator is seeded randomly at start-up, so the `rand.Seed(time.Now().UnixNano())` line (and the `time` import it needs) can be deleted. Since Go 1.24 the call is a no-op.
- Go 1.22 added `math/rand/v2`, where the function is `rand.IntN(100)` (capital `N`).
- The empty-body guard can be replaced by checking the decoder's error: `if err := json.NewDecoder(r.Body).Decode(&course); err != nil { http.Error(w, "invalid JSON", http.StatusBadRequest); return }`.

## Video Timeline
| Time | What happens |
|---|---|
| [01:15](https://youtu.be/vMpBUkIMLzY?t=75) | Why a string `CourseId` makes life harder, and the string conversion that follows |
| [02:26](https://youtu.be/vMpBUkIMLzY?t=146) | What-if mode: handling an empty request body |
| [04:01](https://youtu.be/vMpBUkIMLzY?t=241) | What about an empty JSON object `{}`? |
| [05:05](https://youtu.be/vMpBUkIMLzY?t=305) | Decoding the body with `json.NewDecoder(r.Body).Decode(&course)` |
| [07:30](https://youtu.be/vMpBUkIMLzY?t=450) | Changing `IsEmpty` to check only the course name |
| [08:31](https://youtu.be/vMpBUkIMLzY?t=511) | Generating an id with `rand.Seed` and `rand.Intn`, and the type error |
| [09:25](https://youtu.be/vMpBUkIMLzY?t=565) | `strconv.Itoa` converts the int id to a string; append and respond |

## Related Notes
- [[Consuming JSON Data]] — `Unmarshal` versus the decoder used here.
- [[Building an API - Models]] — the `IsEmpty` helper and the `Course` struct being filled.
- [[Type Conversions]] — `strconv.ParseFloat` earlier, `strconv.Itoa` now.
- [[Math, Crypto and Random Numbers]] — `rand.Intn` and why `rand.Seed` is obsolete.
- [[Slices]] — `append` onto the fake database.
- [[Pointers]] — why `Decode` needs `&course`.
- [[Error Handling in Go]] — the decoder error that should not be ignored.
- [[Update Course Controller]] — reuses the decoder, this time combined with a route parameter.
- [[Routes and Testing the API]] — where `POST /course` is declared and tested with real bodies.
- [[Go Quick Reference]] — syntax cheat sheet.
