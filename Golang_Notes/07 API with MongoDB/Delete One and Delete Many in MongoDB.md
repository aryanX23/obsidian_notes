---
tags:
  - golang
  - mongodb
section: "07 API with MongoDB"
video: https://www.youtube.com/watch?v=qeZueI6d1Wg
video_title: "Delete one and delete many in mongodb in golang"
order: 46
---
# Delete One and Delete Many in MongoDB

> [!info] Source
> 🎬 [Delete one and delete many in mongodb in golang](https://www.youtube.com/watch?v=qeZueI6d1Wg) · Section **07 API with MongoDB** of [[Golang Notes]]
> ⬅️ [[Update a Record in MongoDB]] · ➡️ [[Get All Documents from MongoDB]]

## Overview

Deleting is the simplest of the four database operations, because it needs only one input: a **filter**. There is no update document, no `$set`, nothing to decode. MongoDB offers two flavours and the Go driver mirrors them exactly: `DeleteOne` removes at most one document that matches the filter, `DeleteMany` removes every document that matches. The instructor's rule of thumb covers both: *provide a filter that matches exactly one value to delete one record; provide an empty filter to select everything and delete it all.* This video writes the two helpers `deleteOneMovie` and `deleteAllMovie`, and it is deliberately short because the shape is the same "repetitive syntax" you already met in [[Update a Record in MongoDB]].

## Deletion is a filter, nothing else

| Operation | What you pass | What comes back |
|---|---|---|
| `InsertOne` ([[Insert Data in MongoDB]]) | the document | `*mongo.InsertOneResult` with `InsertedID` |
| `UpdateOne` ([[Update a Record in MongoDB]]) | filter **and** update document | `*mongo.UpdateResult` with `MatchedCount` / `ModifiedCount` |
| `DeleteOne` | filter | `*mongo.DeleteResult` with `DeletedCount` (0 or 1) |
| `DeleteMany` | filter | `*mongo.DeleteResult` with `DeletedCount` |

Both delete methods have the same signature, straight from the driver documentation:

```go
func (coll *Collection) DeleteOne(ctx context.Context, filter interface{}, opts ...*options.DeleteOptions) (*DeleteResult, error)
func (coll *Collection) DeleteMany(ctx context.Context, filter interface{}, opts ...*options.DeleteOptions) (*DeleteResult, error)
```

The `opts` parameter is **variadic** (see [[Functions]]), so you can leave it out entirely, pass `nil`, or pass real options such as a collation. The series never needs options, which is why the two helpers look slightly different: one omits the argument, the other passes `nil`, and both are fine.

## `deleteOneMovie`: delete a document by id

The id arrives as a **string** (it will come from the URL in [[MongoDB API Controllers]]), so the first line is the same conversion as in the update helper, then the filter, then the call:

```go
id, _ := primitive.ObjectIDFromHex(movieId)
filter := bson.M{"_id": id}
deleteCount, err := collection.DeleteOne(context.Background(), filter)
```

- `primitive.ObjectIDFromHex` turns the 24-character hex string into the `primitive.ObjectID` stored in `_id` (see [[Defining Models for MongoDB]]). It also returns an error for a malformed string; the video throws that error away with `_`, and the gotchas below show why that hurts.
- `bson.M{"_id": id}` is the whole query: "the document whose `_id` equals this value". `bson.M` is an unordered map and `bson.D` an ordered slice of pairs; for a one-key filter the choice does not matter, and most code you will read uses `bson.M` (see [[Maps]]).
- `context.Background()` is the context every driver call requires (see [[Connecting to MongoDB]]).

### What `DeleteOne` returns

The instructor names the result `deleteCount`, but the first return value is a `*mongo.DeleteResult`, a struct with a single field:

```go
type DeleteResult struct {
	DeletedCount int64 `bson:"n"` // The number of documents deleted.
}
```

Printing the pointer directly therefore shows the struct, not a number:

```go
fmt.Println("Movie got delete with delete count: ", deleteCount)
```

**Output:**

```text
Movie got delete with delete count:  &{1}
```

`&{1}` is how `fmt` prints a pointer to a struct holding the value `1` (see [[fmt Package and Format Verbs]]). To print the number itself use `deleteCount.DeletedCount`. Nothing else comes back: a delete does not return the document or its id, so if a client needs to know *which* id was removed you have to echo back the id you were given.

## `deleteAllMovie`: the empty filter and `DeleteMany`

```go
deleteResult, err := collection.DeleteMany(context.Background(), bson.D{{}}, nil)
```

Three things are new here compared with the single delete:

1. **The empty filter.** `bson.D{{}}` carries no key/value pair, so it selects every document. The instructor first writes it into a `filter` variable, then recalls a lesson from a senior colleague: when the filter is fixed and will never change, write it **inline** instead of inventing a variable for it. Most code on GitHub does the same.
2. **`nil` options.** Passing `nil` for the options is allowed and does nothing; leaving the argument out is equivalent and cleaner.
3. **Returning the count.** Instead of only printing, the helper returns `deleteResult.DeletedCount`, an `int64`, so the controller can send the number back to the client. That is why the function has a return type while `deleteOneMovie` has none.

### Three spellings of "no filter"

All three select everything, but they do not encode the same way. The encodings below were produced with `bson.Marshal` on driver v1.17:

| Spelling | Encodes as | Why it matches every document |
|---|---|---|
| `bson.D{}` | `{}` | an empty query document is MongoDB's "match all"; the idiomatic form |
| `bson.M{}` | `{}` | same, unordered map form |
| `bson.D{{}}` | `{"": null}` | one element with an empty key and a `nil` value; `null` also matches documents where the field is *missing*, and no document has a field named `""`, so everything matches |

The video (and the MongoDB blog it follows) uses `bson.D{{}}`. It works, but `bson.D{}` says what you mean and does not rely on the "missing field equals null" rule.

## The helpers as written in the video

```go
// delete 1 record
func deleteOneMovie(movieId string) {
	id, _ := primitive.ObjectIDFromHex(movieId)
	filter := bson.M{"_id": id}
	deleteCount, err := collection.DeleteOne(context.Background(), filter)

	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("Movie got delete with delete count: ", deleteCount)
}

// delete all records from mongodb
func deleteAllMovie() int64 {
	deleteResult, err := collection.DeleteMany(context.Background(), bson.D{{}}, nil)

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("Number of movies deleted: ", deleteResult.DeletedCount)
	return deleteResult.DeletedCount
}
```

**Output** on the server console when the controllers in [[MongoDB API Controllers]] call them (one existing id, then a collection holding three movies):

```text
Movie got delete with delete count:  &{1}
Number of movies deleted:  3
```

Deleting the same id a second time prints `&{0}`: the filter matched nothing, and that is not an error.

## Example: insert three movies, delete one, delete the rest

A standalone program with the error handling the video skips. It needs a reachable cluster in `MONGODB_URI`, so the output shown is what the video's cluster produces rather than a run from these notes.

```go
package main

import (
	"context"
	"fmt"
	"log"
	"os"
	"time"

	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/bson/primitive"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

type Netflix struct {
	ID      primitive.ObjectID `json:"_id,omitempty" bson:"_id,omitempty"`
	Movie   string             `json:"movie,omitempty" bson:"movie"`
	Watched bool               `json:"watched,omitempty" bson:"watched"`
}

// deleteMovie reports how many documents were removed, or an error the caller can map to an HTTP status.
func deleteMovie(ctx context.Context, col *mongo.Collection, hexID string) (int64, error) {
	id, err := primitive.ObjectIDFromHex(hexID)
	if err != nil {
		return 0, fmt.Errorf("invalid movie id %q: %w", hexID, err)
	}
	res, err := col.DeleteOne(ctx, bson.M{"_id": id})
	if err != nil {
		return 0, err
	}
	return res.DeletedCount, nil
}

func deleteEverything(ctx context.Context, col *mongo.Collection) (int64, error) {
	res, err := col.DeleteMany(ctx, bson.D{})
	if err != nil {
		return 0, err
	}
	return res.DeletedCount, nil
}

func main() {
	uri := os.Getenv("MONGODB_URI")
	if uri == "" {
		log.Fatal("set MONGODB_URI first")
	}
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()

	client, err := mongo.Connect(ctx, options.Client().ApplyURI(uri))
	if err != nil {
		log.Fatal(err)
	}
	defer client.Disconnect(context.Background())
	col := client.Database("netflix").Collection("watchlist")

	var firstID string
	for _, name := range []string{"Deadpool", "KGF", "Iron Man"} {
		inserted, err := col.InsertOne(ctx, Netflix{Movie: name})
		if err != nil {
			log.Fatal(err)
		}
		if firstID == "" {
			firstID = inserted.InsertedID.(primitive.ObjectID).Hex()
		}
	}

	n, err := deleteMovie(ctx, col, firstID)
	fmt.Println("deleted by id:", n, "error:", err)

	n, err = deleteMovie(ctx, col, firstID) // same id again: nothing left to match
	fmt.Println("deleted by id:", n, "error:", err)

	_, err = deleteMovie(ctx, col, "not-a-valid-id")
	fmt.Println("error:", err)

	n, err = deleteEverything(ctx, col)
	fmt.Println("deleted everything:", n, "error:", err)
}
```

**Output (with a reachable cluster):**

```text
deleted by id: 1 error: <nil>
deleted by id: 0 error: <nil>
error: invalid movie id "not-a-valid-id": the provided hex string is not a valid ObjectID
deleted everything: 2 error: <nil>
```

The error text on the third line is the driver's own message, reproduced locally; the counts depend on what the collection held before the run.

## Gotchas and Best Practices

> [!warning] An ignored `ObjectIDFromHex` error deletes nothing, silently
> With `id, _ :=`, a bad id becomes the zero value `ObjectID("000000000000000000000000")`. The filter matches no document, `DeletedCount` is `0`, and the client never learns that the id was malformed. Check the error and answer `400 Bad Request` (see [[Error Handling in Go]]).

> [!warning] An empty filter on `DeleteOne` is a trap
> `DeleteOne` with `bson.D{}` removes *some* document, whichever the server finds first. `DeleteMany` with an empty filter wipes the collection. Never let user input decide whether the filter is empty, and never expose a "delete all" route without a guard in production.

> [!tip] `DeletedCount == 0` means "not found"
> Like `UpdateOne`, a delete that matches nothing succeeds. Controllers should turn `0` into a `404 Not Found` instead of reporting success.

> [!note] `log.Fatal` inside a helper stops the whole API
> A single failed delete would exit the server process. The standalone example returns `(int64, error)` instead, which is the shape to prefer once the helpers are called from HTTP handlers.

> [!tip] Prefer `bson.D{}` or `bson.M{}` for "everything"
> `bson.D{{}}` works because of how `null` matches missing fields; the empty literal is clearer and is what the driver documentation describes as the match-all filter.

## Modern Go Notes

- Driver v2 (`go.mongodb.org/mongo-driver/v2`) keeps `DeleteOne`, `DeleteMany` and `DeleteResult`; the ObjectID helpers move to the `bson` package (`bson.ObjectIDFromHex`), so check the v2 documentation before upgrading.
- Returning `(int64, error)` or `(*mongo.DeleteResult, error)` from helpers, as in the example, lets controllers choose between `200`, `400` and `404` instead of killing the process with `log.Fatal`.

## Video Timeline

| Time | What happens |
|---|---|
| [00:16](https://youtu.be/qeZueI6d1Wg?t=16) | Two ways to delete, one record or all of them, and both are driven by a filter |
| [01:29](https://youtu.be/qeZueI6d1Wg?t=89) | `primitive.ObjectIDFromHex(movieId)` turns the string id into an ObjectID |
| [01:53](https://youtu.be/qeZueI6d1Wg?t=113) | The `bson.M{"_id": id}` filter; no update document this time |
| [02:21](https://youtu.be/qeZueI6d1Wg?t=141) | `collection.DeleteOne(context.Background(), filter)` and the delete count it returns |
| [04:48](https://youtu.be/qeZueI6d1Wg?t=288) | `deleteAllMovie` with `collection.DeleteMany` and the empty filter |
| [05:44](https://youtu.be/qeZueI6d1Wg?t=344) | Why seniors write a fixed filter inline; optional options or `nil` |
| [07:57](https://youtu.be/qeZueI6d1Wg?t=477) | `DeletedCount` is an `int64`; returning it from the helper |

## Related Notes

- [[Update a Record in MongoDB]] — the same id conversion and filter, plus an update document
- [[Get All Documents from MongoDB]] — the next helper, where the empty filter returns a cursor instead of a count
- [[MongoDB API Controllers]] — `DeleteAMovie` and `DeleteAllMovies` call these helpers
- [[Delete Course Controller]] — deleting from the in-memory slice in the previous section, for comparison
- [[Defining Models for MongoDB]] — `primitive.ObjectID` and the `_id` field
- [[Insert Data in MongoDB]] — where the documents being deleted were created
- [[Functions]] — variadic parameters explain the optional `opts`
- [[fmt Package and Format Verbs]] — why a pointer to a struct prints as `&{1}`
- [[Error Handling in Go]] — handling the conversion and driver errors instead of discarding them
- [[Go Quick Reference]] — the delete recipe in one line
