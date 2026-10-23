---
tags:
  - golang
  - mongodb
section: "07 API with MongoDB"
video: https://www.youtube.com/watch?v=GszGvj6eBZY
video_title: "Making a connection to database in golang"
order: 43
---
# Connecting to MongoDB

> [!info] Source
> 🎬 [Making a connection to database in golang](https://www.youtube.com/watch?v=GszGvj6eBZY) · Section **07 API with MongoDB** of [[Golang Notes]]
> ⬅️ [[Defining Models for MongoDB]] · ➡️ [[Insert Data in MongoDB]]

## Overview

With the model in place, the next job is to open a connection to the Atlas cluster created in [[MongoDB Setup]] and keep a handle to the collection that every database helper can reuse. The pattern is the same for any database driver (MySQL, Postgres, MongoDB): build a set of **client options**, **connect** with them, check the error, then grab a reference to the database and collection you care about. Along the way the video introduces two Go features that have not appeared in the series before: the special `init()` function and the `context` package.

## Where the code lives

For a bigger application the instructor would keep the connection in one file, the database helpers in another and the HTTP controllers in a third. Since the project is small, all three go into a single `controller/controller.go` (package `controller`). Because they are in the same package they can share unexported variables freely, and because the folder is its own package nothing collides with `main.go`. Splitting the file later is a good exercise: Go only cares that the files sit in the same folder.

## The constants

```go
package controller

const connectionString = "mongodb+srv://learncodeonline:<password>@cluster0.humov.mongodb.net/myFirstDatabase?retryWrites=true&w=majority"
const dbName = "netflix"
const colName = "watchlist"
```

- `connectionString` is the string Atlas gave us. For a local server it would be `mongodb://localhost:27017`. **Replace `<password>`** with the database user's real password; pasting the string blindly is the most common first error.
- `dbName` and `colName` name the database and the collection inside it. Neither needs to exist beforehand: MongoDB creates both lazily on the first insert.

> [!warning] Secrets do not belong in code
> The instructor says it himself: both the password and the whole connection string normally come from **environment variables**, and he only hard-codes them "taking the liberty of a tutorial". The repository on GitHub still contains that string. See *Modern Go Notes* below for the environment-variable version.

## The shared collection handle

```go
// MOST IMPORTANT
var collection *mongo.Collection
```

This package-level variable is a [[Pointers|pointer]] to a `mongo.Collection`. Every helper in the next notes (`insertOneMovie`, `updateOneMovie`, ...) uses it, so instead of digging through client → database → collection on every call we resolve the reference once and let it "float around" the package. Typing `*mongo.Collection` (capital C) makes the editor add the `go.mongodb.org/mongo-driver/mongo` import automatically.

## `init()`: code that runs before `main`

The connection is established inside a function named `init`:

- Go calls `init()` **automatically**, exactly once, when the package is initialised, which happens before `main()` runs and after the package-level variables have been set up.
- You cannot call it yourself, and a package may contain several `init` functions (they run in the order they appear).
- It is the natural place for one-time setup such as opening a database connection, which is why the instructor uses it here instead of a normal function (see [[Functions]] for ordinary functions).

> [!note] `init` only runs for packages that are imported
> At the end of this video `main.go` does not import `controller` yet, so running the program would not even attempt the connection. Once [[MongoDB API Routes and Testing]] wires `main → router → controller`, the `init` output appears **before** `main`'s own first print.

## Four steps to a connection

```go
func init() {
	// 1. client options: which server, with which settings
	clientOption := options.Client().ApplyURI(connectionString)

	// 2. actually connect
	client, err := mongo.Connect(context.TODO(), clientOption)

	// 3. proceed with caution
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("MongoDB connection success")

	// 4. resolve the collection reference once
	collection = client.Database(dbName).Collection(colName)
	fmt.Println("Collection instance is ready")
}
```

1. `options.Client()` returns a `*options.ClientOptions`; `ApplyURI` parses the connection string into it (hosts, credentials, TLS, `retryWrites`...). Options can be chained, and later setters override earlier ones.
2. `mongo.Connect(ctx, opts...)` creates a `*mongo.Client` and initialises it. Everything before it was just configuration; `Connect` starts the connection pool in the background (see the Gotcha below: it does not prove the cluster is reachable).
3. Like the `try/catch` of other languages, the call returns an error you must look at. `log.Fatal(err)` prints the error with a timestamp and exits the program; `panic(err)` would also work but `log.Fatal` is the cleaner choice for "the app cannot start" (see [[Error Handling in Go]]).
4. `client.Database(dbName)` returns a `*mongo.Database`, and `.Collection(colName)` a `*mongo.Collection`. Storing it in the package variable is what the comment "MOST IMPORTANT" is about.

```mermaid
sequenceDiagram
    participant P as package controller (init)
    participant O as options.ClientOptions
    participant C as mongo.Client
    participant A as MongoDB Atlas
    P->>O: Client().ApplyURI(connectionString)
    P->>C: mongo.Connect(context.TODO(), opts)
    C-->>A: open connection pool (in the background)
    C-->>P: client, err
    P->>C: Database("netflix").Collection("watchlist")
    C-->>P: *mongo.Collection stored in `collection`
```

## What is a `context`?

`mongo.Connect`, and every other driver call, takes a `context.Context` as its first argument. The instructor opens the package documentation on go.dev to explain it: *"Package context defines the Context type, which carries deadlines, cancellation signals, and other request-scoped values across API boundaries and between processes."*

Whenever your program talks to something outside itself (a database on another machine, a web service) the call might hang. A context lets the caller say *how long* the operation may take and *cancel* it when it is no longer needed. Two ready-made empty contexts exist:

| Function | Documentation says | When the series uses it |
|---|---|---|
| `context.Background()` | never cancelled, no values, no deadline; the top-level context used by `main`, initialisation and incoming requests | every database operation (`InsertOne`, `UpdateOne`, ...) |
| `context.TODO()` | also empty; use it when it is unclear which context to use, or the function has not been extended to accept one yet | the `mongo.Connect` call |

> [!tip] `TODO` is not a to-do list
> The name means "a context is required here, figure out the right one later". Both `Background` and `TODO` behave identically at runtime; the difference is documentation of intent. The instructor reports that `Background` works for `Connect` as well.

![[43_connecting_mongodb_context_docs_0612.png]]
*Figure: the `context` package overview on go.dev that the video walks through (video at 06:12)*

## Full listing (so far)

```go
package controller

import (
	"context"
	"fmt"
	"log"

	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

const connectionString = "mongodb+srv://learncodeonline:<password>@cluster0.humov.mongodb.net/myFirstDatabase?retryWrites=true&w=majority"
const dbName = "netflix"
const colName = "watchlist"

// MOST IMPORTANT
var collection *mongo.Collection

// connect with MongoDB
func init() {
	clientOption := options.Client().ApplyURI(connectionString)

	client, err := mongo.Connect(context.TODO(), clientOption)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("MongoDB connection success")

	collection = client.Database(dbName).Collection(colName)
	fmt.Println("Collection instance is ready")
}
```

**Output** (once the package is imported by `main`, see [[MongoDB API Routes and Testing]]):

```text
MongoDB connection success
Collection instance is ready
MongoDB API
Server is getting started...
```

## Example: a standalone connection check

The video does not test the connection yet. This small program does, reading the URI from an environment variable and **pinging** the cluster so a wrong password or a blocked IP shows up immediately:

```go
package main

import (
	"context"
	"fmt"
	"log"
	"os"
	"time"

	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
	"go.mongodb.org/mongo-driver/mongo/readpref"
)

func main() {
	uri := os.Getenv("MONGODB_URI")
	if uri == "" {
		log.Fatal("set MONGODB_URI, e.g. export MONGODB_URI='mongodb+srv://user:pass@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority'")
	}

	// Give the whole handshake at most 10 seconds.
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()

	client, err := mongo.Connect(ctx, options.Client().ApplyURI(uri))
	if err != nil {
		log.Fatal(err)
	}
	defer client.Disconnect(context.Background())

	if err := client.Ping(ctx, readpref.Primary()); err != nil {
		log.Fatal("cannot reach MongoDB: ", err)
	}
	fmt.Println("MongoDB connection success")

	collection := client.Database("netflix").Collection("watchlist")
	fmt.Println("Collection instance is ready:", collection.Name())
}
```

**Output:**

```text
MongoDB connection success
Collection instance is ready: watchlist
```

With a wrong password the `Ping` fails with an authentication error; with an IP that is not whitelisted it fails with a server selection timeout. Those are the two setup mistakes from [[MongoDB Setup]].

## Gotchas and Best Practices

> [!warning] `mongo.Connect` succeeding does not mean the cluster is reachable
> `Connect` validates the options and starts the connection pool in the background; a bad password or a firewall only surfaces on the first real operation. Use `client.Ping` right after connecting if you want to fail fast, as in the example above.

> [!warning] `log.Fatal` skips deferred calls
> `log.Fatal` is `Print` followed by `os.Exit(1)`, so any `defer` (see [[Defer]]) never runs. That is acceptable during start-up, but inside request handlers prefer returning an error.

> [!tip] One client for the whole application
> A `*mongo.Client` is safe for concurrent use and manages a pool of connections. Create it once (here in `init`) and share it, instead of connecting inside every handler.

> [!note] Database and collection are created lazily
> Nothing is sent to Atlas when you call `Database()` / `Collection()`; they are cheap handles. The database `netflix` and collection `watchlist` appear in Atlas after the first insert in [[Insert Data in MongoDB]].

## Modern Go Notes

- **Environment variables** instead of constants: `uri := os.Getenv("MONGODB_URI")` and `export MONGODB_URI=...` before running, or load a `.env` file with a small helper library such as `github.com/joho/godotenv`.
- Give the connection a **timeout** with `context.WithTimeout` and `defer cancel()`, and release resources with `defer client.Disconnect(ctx)` when the program ends.
- In driver **v2** the `Connect` signature no longer takes a context (`mongo.Connect(opts ...*options.ClientOptions)`); check the v2 documentation before migrating. The series stays on v1 where `mongo.Connect(ctx, opts)` is correct.

## Video Timeline

| Time | What happens |
|---|---|
| [00:22](https://youtu.be/GszGvj6eBZY?t=22) | Why connection, helpers and controllers share one file here |
| [01:28](https://youtu.be/GszGvj6eBZY?t=88) | `connectionString`, `dbName`, `colName` and replacing `<password>` |
| [03:05](https://youtu.be/GszGvj6eBZY?t=185) | The shared `*mongo.Collection` variable |
| [03:51](https://youtu.be/GszGvj6eBZY?t=231) | `init()` and the client options with `ApplyURI` |
| [05:50](https://youtu.be/GszGvj6eBZY?t=350) | Reading the `context` docs: `Background` vs `TODO` |
| [08:16](https://youtu.be/GszGvj6eBZY?t=496) | Error handling with `log.Fatal` and resolving the collection |

## Related Notes

- [[MongoDB Setup]] — where the connection string comes from
- [[Defining Models for MongoDB]] — the struct the collection stores
- [[Insert Data in MongoDB]] — first use of the `collection` handle
- [[Functions]] — ordinary functions, for contrast with `init`
- [[Pointers]] — why the collection handle is a pointer
- [[Error Handling in Go]] — `log.Fatal`, `panic` and the `if err != nil` idiom
- [[Defer]] — why `log.Fatal` and `defer` do not mix
- [[Concurrency and Goroutines]] — the driver runs its connection pool concurrently
- [[Go Quick Reference]] — the connection recipe in four lines
