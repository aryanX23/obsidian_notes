---
tags:
  - golang
  - api
section: "06 Building an API"
video: https://www.youtube.com/watch?v=b1nJBaCzkHI
video_title: "Delete a course controller in golang"
order: 39
---
# Delete Course Controller

> [!info] Source
> 🎬 [Delete a course controller in golang](https://www.youtube.com/watch?v=b1nJBaCzkHI) · Section **06 Building an API** of [[Golang Notes]]
> ⬅️ [[Update Course Controller]] · ➡️ [[Routes and Testing the API]]

## Overview
Delete is the last of the four CRUD operations and the shortest video in the section, because every piece has already been built: read the id from the route as in [[Get One Course Controller]], find the course with an indexed loop, and remove it with the slice trick from [[Update Course Controller]]. The one new idea is a small quiz about loop control: after removing the element, should the loop `continue` or `break`? The video also hands out two exercises, a confirmation response and a `deleteAllCourses` controller.

## The plan
1. Grab the id from the request parameters.
2. Loop through `courses` with the index.
3. When the id matches, remove that index with `append(courses[:index], courses[index+1:]...)`.
4. Stop the loop.

## Example: the `deleteOneCourse` controller
```go
func deleteOneCourse(w http.ResponseWriter, r *http.Request) {
	fmt.Println("Delete one course")
	w.Header().Set("Content-Type", "application/json")

	params := mux.Vars(r)

	// loop, id, remove (index, index+1)
	for index, course := range courses {
		if course.CourseId == params["id"] {
			courses = append(courses[:index], courses[index+1:]...)
			// TODO: send a confirm or deny response
			break
		}
	}
}
```

- `params := mux.Vars(r)` reads the `{id}` route variable (declared in [[Routes and Testing the API]]).
- `for index, course := range courses` keeps the index because the removal needs it (see [[Loops, Break, Continue and Goto]]).
- The `append` line is the remove-by-index idiom from [[Slices]]: everything before the index, joined with everything after it.
- `break` leaves the loop as soon as the deletion is done.

> [!tip] Fix the copy-pasted log labels
> While copying the signature block, the instructor notices that the update controller still printed `"Create one course"`. The `fmt.Println` label is the only thing telling you which controller ran during testing, so make sure each one says the right name.

## `break` or `continue`?
The instructor's quiz: after removing the course, which keyword?

| Keyword | Effect | Right here? |
|---|---|---|
| `continue` | Skips the rest of **this iteration** and moves to the next element | No: the loop would keep walking a slice whose length just changed |
| `break` | Leaves the **whole loop** immediately | Yes: the job is done and there is nothing left to look for |

Because the element at `index` was removed, the element that used to be at `index+1` now sits at `index`, while the `range` loop moves on to `index+1`. Continuing would skip one element and, at the end, read past the shortened slice's last valid index. `break` avoids all of that. `return` would work too, and the stricter version below uses it so a response can be written and the function ended in one step.

## What the client sees
Once `DELETE /course/{id}` exists ([[Routes and Testing the API]]), deleting the seeded ReactJS course:

```bash
curl -i -X DELETE http://localhost:4000/course/2
```

```text
HTTP/1.1 200 OK
Content-Type: application/json
```

The body is empty, which is the `TODO` in the code: the client has no way to know whether anything was deleted and must call `GET /courses` to find out. Doing so confirms that only the MERN Stack course and the newly created course remain:

```json
[
  { "courseid": "4", "coursename": "MERN Stack", "price": 199, "author": { "fullname": "Hitesh Choudhary", "website": "go.dev" } },
  { "courseid": "3", "coursename": "NodeJS Crash Course", "price": 399, "author": { "fullname": "Brad T", "website": "brad.com" } }
]
```

Deleting an id that does not exist also returns an empty `200`.

## Exercises from the video
### 1. Send a confirmation
Craft a JSON response inside the `if`, before leaving the loop, saying that the id was removed. A complete version with a not-found branch:

```go
func deleteOneCourse(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	params := mux.Vars(r)

	for index, course := range courses {
		if course.CourseId == params["id"] {
			courses = append(courses[:index], courses[index+1:]...)
			json.NewEncoder(w).Encode("Course " + params["id"] + " was deleted")
			return
		}
	}

	w.WriteHeader(http.StatusNotFound)
	json.NewEncoder(w).Encode("No course found with id " + params["id"])
}
```

### 2. `deleteAllCourses`
The instructor suggests two approaches: loop and remove every element, or simply throw the slice away and recreate it. The second is one line:

```go
func deleteAllCourses(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	courses = []Course{} // drop the old backing array; an empty (not nil) slice encodes as []
	json.NewEncoder(w).Encode("All courses deleted")
}
```

Assigning `[]Course{}` rather than `nil` keeps `GET /courses` returning `[]` instead of `null` (see [[Get All Courses Controller]]). The MongoDB version of this exercise is `DeleteMany` in [[Delete One and Delete Many in MongoDB]].

## Gotchas and Best Practices
> [!warning] Never `continue` after mutating the slice you are ranging over
> Indices shift the moment an element is removed. Finish with `break` or `return`.

> [!tip] A delete should confirm or deny
> `200` with a message, or `204 No Content` with no body, for success; `404` when the id is unknown. Silence makes every client do a second request.

> [!note] Deleting by value in a loop is O(n)
> Fine for a slice of a few courses; a database index does this in constant time, as seen in [[Delete One and Delete Many in MongoDB]].

## Modern Go Notes
- Go 1.21's `slices` package offers `slices.IndexFunc` to find the position and `slices.Delete(courses, i, i+1)` to remove it, replacing the manual loop and the `append` idiom.
- With Go 1.22's `net/http` patterns the route is `"DELETE /course/{id}"` and the id is read with `r.PathValue("id")`.

## Video Timeline
| Time | What happens |
|---|---|
| [00:23](https://youtu.be/b1nJBaCzkHI?t=23) | Declaring `deleteOneCourse`, and the `deleteAllCourses` exercise |
| [01:12](https://youtu.be/b1nJBaCzkHI?t=72) | Fixing the copy-pasted log label in the update controller |
| [01:59](https://youtu.be/b1nJBaCzkHI?t=119) | Grabbing the params and writing the action plan |
| [02:35](https://youtu.be/b1nJBaCzkHI?t=155) | Loop with index, match the id, remove with the slice trick |
| [03:57](https://youtu.be/b1nJBaCzkHI?t=237) | `break` versus `continue`, and why `break` is the right choice |
| [04:17](https://youtu.be/b1nJBaCzkHI?t=257) | Exercise: send a JSON confirmation response |

## Related Notes
- [[Update Course Controller]] — the same removal, followed by a re-append.
- [[Get One Course Controller]] — the find-by-id loop and `mux.Vars`.
- [[Slices]] — the remove-by-index idiom.
- [[Loops, Break, Continue and Goto]] — `break` versus `continue`.
- [[Get All Courses Controller]] — why an empty slice should be `[]Course{}` rather than `nil`.
- [[Routes and Testing the API]] — where `DELETE /course/{id}` is declared and tested.
- [[Delete One and Delete Many in MongoDB]] — the database version of both exercises.
- [[Go Quick Reference]] — syntax cheat sheet.
