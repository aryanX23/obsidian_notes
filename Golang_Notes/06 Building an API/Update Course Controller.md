---
tags:
  - golang
  - api
section: "06 Building an API"
video: https://www.youtube.com/watch?v=TQpB_QSgmXU
video_title: "Update a course controller in golang"
order: 38
---
# Update Course Controller

> [!info] Source
> 🎬 [Update a course controller in golang](https://www.youtube.com/watch?v=TQpB_QSgmXU) · Section **06 Building an API** of [[Golang Notes]]
> ⬅️ [[Create Course Controller]] · ➡️ [[Delete Course Controller]]

## Overview
Updating is the most involved CRUD operation in this section because it needs **two inputs at once**: the id of the course to change (from the URL, as in [[Get One Course Controller]]) and the new values (from the JSON body, as in [[Create Course Controller]]). A database would offer a "find by id and update" call; a slice does not, so the instructor composes the operation out of pieces already learned: find the index, **remove** the old record with the slice trick from [[Slices]], decode the replacement, force its id to the one from the URL, and append it back.

## Two inputs, one request
| Input | Where it comes from | How it is read |
|---|---|---|
| Which course | URL path, `PUT /course/39` | `mux.Vars(r)["id"]` |
| New values | JSON request body | `json.NewDecoder(r.Body).Decode(&course)` |

An update normally **resends every field**, whether it changed or not. That matches HTTP `PUT` semantics (replace the resource); partial updates are what `PATCH` is for, and the series does not implement them.

## The action plan
The instructor writes the strategy as comments before coding, and the comments survive in the repository:

```go
// first - grab id from req
// loop, id, remove, add with my ID
```

Spelled out:

1. Grab the id from the request parameters.
2. Loop through `courses` and stop at the course with that id.
3. Remove that course from the slice.
4. Decode the new values from the body into a fresh `Course`.
5. Overwrite its `CourseId` with the id from the URL.
6. Append it to `courses` and send it back.

## Step by step
### Finding the course, with its index
Unlike the read controller, the loop keeps the **index** this time, because the removal needs it:

```go
params := mux.Vars(r)

for index, course := range courses {
	if course.CourseId == params["id"] {
		// steps 3 to 6 go here
	}
}
```

### Removing by index
This is the exact operation from the second half of [[Slices]] (removing a value based on its index):

```go
courses = append(courses[:index], courses[index+1:]...)
```

`courses[:index]` keeps everything before the match, `courses[index+1:]` everything after it, and `append` joins the two. The three dots are required because `append` is **variadic**: it takes individual elements, so a slice has to be spread with `...` (see [[Functions]]).

### Decoding the replacement and keeping the id
```go
var course Course
_ = json.NewDecoder(r.Body).Decode(&course)
course.CourseId = params["id"]
```

The client may send any `courseid` it likes, or none; the controller **overwrites** it with the id from the URL so the record keeps its identity. Everything else comes from the body.

### Appending and responding
```go
courses = append(courses, course)
json.NewEncoder(w).Encode(course)
return
```

The updated course is appended at the **end** of the slice, so its position changes; order does not matter for this API. The `return` is what prevents the loop from continuing over a slice that was just modified.

## Example: the complete `updateOneCourse` controller
```go
func updateOneCourse(w http.ResponseWriter, r *http.Request) {
	fmt.Println("Update one course")
	w.Header().Set("Content-Type", "application/json")

	// first - grab id from req
	params := mux.Vars(r)

	// loop, id, remove, add with my ID
	for index, course := range courses {
		if course.CourseId == params["id"] {
			courses = append(courses[:index], courses[index+1:]...)
			var course Course
			_ = json.NewDecoder(r.Body).Decode(&course)
			course.CourseId = params["id"]
			courses = append(courses, course)
			json.NewEncoder(w).Encode(course)
			return
		}
	}
	// TODO: send a response when id is not found
}
```

With the course created in [[Create Course Controller]] (id `3` in this run), once the route `PUT /course/{id}` exists ([[Routes and Testing the API]]):

```bash
curl -X PUT http://localhost:4000/course/3 -H 'Content-Type: application/json' \
  -d '{"coursename":"NodeJS Crash Course","price":399,"author":{"fullname":"Brad T","website":"brad.com"}}'
```

```json
{
  "courseid": "3",
  "coursename": "NodeJS Crash Course",
  "price": 399,
  "author": { "fullname": "Brad T", "website": "brad.com" }
}
```

A following `GET /courses` shows the renamed course in place of the old one. For an id that does not exist, the loop finishes without a match and the function ends silently:

```bash
curl -i -X PUT http://localhost:4000/course/999 -H 'Content-Type: application/json' -d '{"coursename":"Ghost"}'
```

```text
HTTP/1.1 200 OK
Content-Type: application/json
```

An empty `200` for a failed update is misleading, which is why the instructor leaves a `TODO` there.

## Finishing the TODO
The not-found case is the exercise. One line of status and one encoded message after the loop complete the controller, and the same pattern fits [[Delete Course Controller]]:

```go
	w.WriteHeader(http.StatusNotFound)
	json.NewEncoder(w).Encode("No course found with id " + params["id"])
}
```

## Gotchas and Best Practices
> [!warning] Two variables called `course`
> Inside the `if`, `var course Course` declares a **new** variable that shadows the loop's `course`. It compiles, and it works because the loop variable is not used afterwards, but it reads badly. Naming the decoded value `updated` makes the intent obvious.

> [!warning] Fields you do not send become zero values
> A body without `price` produces `"price": 0`, and a body without `author` produces `"author": null`. That is replace semantics, not merge semantics. Clients must send the full record, or the API needs a `PATCH` route.

> [!note] Modifying a slice inside its own `range` loop
> `range courses` evaluates the slice once at the start, so changing `courses` in the body does not crash. It is still only safe here because the function returns immediately; continuing the loop over stale indices would skip or repeat elements.

> [!tip] Same checks as create
> Empty body, `{}`, duplicate names: all the validation from [[Create Course Controller]] applies to updates too. The instructor skips it to avoid repetition, not because it is unnecessary.

## Modern Go Notes
- Go 1.21 added the `slices` package: `courses = slices.Delete(courses, index, index+1)` does the removal without the `append` idiom.
- With Go 1.22's `net/http` patterns the route becomes `"PUT /course/{id}"` and the id is read with `r.PathValue("id")`; see [[Routes and Testing the API]].

## Video Timeline
| Time | What happens |
|---|---|
| [00:47](https://youtu.be/TQpB_QSgmXU?t=47) | What an update needs: the id from the URL and the new values from the body |
| [01:49](https://youtu.be/TQpB_QSgmXU?t=109) | Reusing the remove-by-index trick from the slices video |
| [02:50](https://youtu.be/TQpB_QSgmXU?t=170) | Grabbing the id with `mux.Vars` |
| [04:25](https://youtu.be/TQpB_QSgmXU?t=265) | Looping with the index and matching the id |
| [05:33](https://youtu.be/TQpB_QSgmXU?t=333) | Removing the old course with `append(courses[:index], courses[index+1:]...)` |
| [06:44](https://youtu.be/TQpB_QSgmXU?t=404) | Decoding the new values, forcing the id from params, appending and responding |
| [08:53](https://youtu.be/TQpB_QSgmXU?t=533) | TODO: respond when the id is not found |

## Related Notes
- [[Slices]] — removing an element by index, the heart of this controller.
- [[Get One Course Controller]] — the find-by-id loop and `mux.Vars`.
- [[Create Course Controller]] — decoding the request body and the validation that should be repeated here.
- [[Delete Course Controller]] — the same removal without the re-append.
- [[Functions]] — variadic parameters and the `...` spread.
- [[Pointers]] — `Decode(&course)` fills the struct through its address.
- [[Routes and Testing the API]] — where `PUT /course/{id}` is declared and tested.
- [[Update a Record in MongoDB]] — the same operation once a database does the heavy lifting.
- [[Go Quick Reference]] — syntax cheat sheet.
