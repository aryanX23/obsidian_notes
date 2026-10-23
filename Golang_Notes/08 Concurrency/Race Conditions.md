---
tags:
  - golang
  - concurrency
section: "08 Concurrency"
video: https://www.youtube.com/watch?v=4TCNUa-5wmI
video_title: "Race Condition in golang"
order: 55
---
# Race Conditions

> [!info] Source
> 🎬 [Race Condition in golang](https://www.youtube.com/watch?v=4TCNUa-5wmI) · Section **08 Concurrency** of [[Golang Notes]]
> ⬅️ [[Mutex]] · ➡️ [[Channels and Deadlock]]

## Overview

A **race condition** (more precisely, a *data race*) is what [[Mutex]] warned about: several goroutines touching the same memory at the same time, with at least one of them writing. The program may still print the right answer, which is why the instructor insists this is "the easiest topic conceptually" and yet the one that cost him the most time on a real project: the symptom was a mysterious `exit status 66`. This video builds a tiny program where three goroutines append to one slice, shows that the output *looks* fine, then switches on Go's built-in **race detector** (`go run -race`) to prove the bug exists, and fixes it with the mutex and the RWMutex from the previous video. Along the way it introduces anonymous, immediately-invoked goroutines and the pointer style for WaitGroups and mutexes.

## The Setup: Three Anonymous Goroutines

Instead of named functions the video uses **immediately invoked anonymous functions** (he borrows the JavaScript nickname "IIFE", but any language with anonymous functions can do it). The shape is: a function literal with no name, followed immediately by `( ... )` to call it, with `go` in front to run it as a goroutine:

```go
go func(wg *sync.WaitGroup) {
	fmt.Println("One R")
	score = append(score, 1)
	wg.Done()
}(wg) // called right away, with wg as the argument
```

The `score` slice starts as `[]int{0}` and each of the three goroutines appends its own number. Because the goroutines must be waited for, a [[WaitGroups|WaitGroup]] is created **as a pointer** this time (`wg := &sync.WaitGroup{}`), which is the style you will use in real code, and passed into each function as `wg *sync.WaitGroup` (see [[Pointers]]). The counter is set once with `wg.Add(3)` before the goroutines start; the instructor notes that one `Add(1)` per goroutine is equally valid.

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	fmt.Println("Race condition - LearnCodeonline.in")

	wg := &sync.WaitGroup{}
	var score = []int{0}

	wg.Add(3)
	go func(wg *sync.WaitGroup) {
		fmt.Println("One R")
		score = append(score, 1)
		wg.Done()
	}(wg)
	go func(wg *sync.WaitGroup) {
		fmt.Println("Two R")
		score = append(score, 2)
		wg.Done()
	}(wg)
	go func(wg *sync.WaitGroup) {
		fmt.Println("Three R")
		score = append(score, 3)
		wg.Done()
	}(wg)

	wg.Wait()
	fmt.Println(score)
}
```

**Output:** (two consecutive runs)

```text
Race condition - LearnCodeonline.in
Three R
One R
Two R
[0 3 1 2]
```

```text
Race condition - LearnCodeonline.in
Three R
Two R
One R
[0 3 2 1]
```

Two observations. The goroutines finish in a **different order each run**; there is no guarantee which "independent thread" (the instructor's deliberately loose word for goroutine) reports back first. And every run ends with all four numbers in the slice, so "everything is working fine". The next section shows that it is not.

## Proving It: `go run -race`

Go ships a race detector. Add the `-race` flag to `run`, `build` or `test` and the compiled program watches every memory access from every goroutine and reports conflicting ones:

```bash
go run -race .
```

**Output:** (trimmed; paths shortened)

```text
Race condition - LearnCodeonline.in
Two R
Three R
One R
==================
WARNING: DATA RACE
Read at 0x00c0001ce000 by goroutine 9:
  main.main.func3()
      main.go:27 +0x7c

Previous write at 0x00c0001ce000 by goroutine 8:
  main.main.func2()
      main.go:22 +0xe0

Goroutine 9 (running) created at:
  main.main()
      main.go:25 +0x3e0

Goroutine 8 (finished) created at:
  main.main()
      main.go:20 +0x2ec
==================
==================
WARNING: DATA RACE
Read at 0x00c00019e068 by goroutine 7:
  runtime.growslice()
      /usr/local/go/src/runtime/slice.go:178 +0x0
  main.main.func1()
      main.go:17 +0xac
...
[0 2 3 1]
Found 2 data race(s)
exit status 66
```

Read the report the way the instructor does: `WARNING: DATA RACE`, then *which goroutine read or wrote which address at which line*, then where each goroutine was created. Here goroutine 9 (the third function literal, `func3`, line 27) read the `score` slice header that goroutine 8 (`func2`, line 22) had just written. The second report even shows the race happening inside `runtime.growslice`, the function behind `append` (see [[Slices]]). The program still finished and printed a plausible slice, but it **exits with status 66**, the race detector's signature, which is the number that puzzled the instructor for days on his project. The detector reports any two unsynchronised accesses to the same memory that both executed during the run (it reasons about happens-before relationships, not about whether the accesses overlapped in time), so only code paths that were never exercised stay hidden: a clean run does not prove the absence of races, but a report always means a real bug.

![[55_race_detector_output_1005.png]]
*Figure: the race detector's WARNING: DATA RACE report and exit status 66 in the instructor's terminal (video at 10:05)*

## The Fix: a Mutex Passed by Pointer

The solution is the [[Mutex]] from the previous video, this time created as a pointer and passed into every goroutine alongside the WaitGroup. Every **write** to `score` is wrapped in `Lock` / `Unlock`:

```go
mut := &sync.Mutex{}

go func(wg *sync.WaitGroup, m *sync.Mutex) {
	fmt.Println("One R")
	m.Lock()
	score = append(score, 1)
	m.Unlock()
	wg.Done()
}(wg, mut)
```

The bank-account picture from the video: one goroutine deposits a thousand, another withdraws a thousand, and without a lock on the account both can read the same old balance and one update is lost. With the lock, the second one waits.

Run again with `-race` and the detector prints nothing extra, which "is always a good sign".

## RWMutex and Where `RLock` Belongs

The last part of the video adds a fourth goroutine that only **reads** the slice, and switches the type to `sync.RWMutex` so readers do not have to queue behind each other. The signatures change to `m *sync.RWMutex`, the writers keep `Lock`/`Unlock`, and the reader uses `RLock`/`RUnlock`:

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	fmt.Println("Race condition - LearnCodeonline.in")

	wg := &sync.WaitGroup{}
	mut := &sync.RWMutex{}
	var score = []int{0}

	wg.Add(4)
	go func(wg *sync.WaitGroup, m *sync.RWMutex) {
		fmt.Println("One R")
		m.Lock()
		score = append(score, 1)
		m.Unlock()
		wg.Done()
	}(wg, mut)
	go func(wg *sync.WaitGroup, m *sync.RWMutex) {
		fmt.Println("Two R")
		m.Lock()
		score = append(score, 2)
		m.Unlock()
		wg.Done()
	}(wg, mut)
	go func(wg *sync.WaitGroup, m *sync.RWMutex) {
		fmt.Println("Three R")
		m.Lock()
		score = append(score, 3)
		m.Unlock()
		wg.Done()
	}(wg, mut)
	go func(wg *sync.WaitGroup, m *sync.RWMutex) {
		fmt.Println("Reader R")
		m.RLock()
		fmt.Println(score)
		m.RUnlock()
		wg.Done()
	}(wg, mut)

	wg.Wait()
	fmt.Println(score)
}
```

**Output of `go run -race .`:** (no race report; order varies)

```text
Race condition - LearnCodeonline.in
One R
Three R
Two R
Reader R
[0 1 3 2]
[0 1 3 2]
```

This is the companion program (folder `27mutexAndAwaitGroups`) with the writers using the function parameter `m` rather than the outer `mut` and the fourth goroutine counted in `Add(4)`; the repository's `Add(3)` with four `Done()` calls is one of the small inconsistencies worth fixing when you type it yourself (a fourth `Done` on a counter of three can panic with `sync: negative WaitGroup counter`, see [[WaitGroups]]).

The instructor spends a minute on *where* the read lock goes, and the answer is: **around the read**, inside the goroutine that reads. Putting `RLock` next to the variable declaration is a mistake he has seen in real code; the lock protects an *operation* on the memory, not the place where the memory was declared.

How the RWMutex behaves, in his summary: any number of goroutines may hold the read lock at once; when a goroutine asks for the write lock, readers are kept out until the write is finished, and during the write nobody reads. For the three-writer program it changes nothing measurable, but in read-heavy code it is what keeps readers from serialising needlessly.

## Gotchas and Best Practices

> [!tip] Run your tests with `-race`
> `go test -race ./...` costs roughly 2 to 20 times the execution time and 5 to 10 times the memory (the figures given in the Go race detector documentation), which is why it is not the default, but it is the single most effective way to catch concurrency bugs before production. Treat any report as a bug, even if the output looked right.

> [!warning] A clean run is not a proof
> The detector can only judge code that actually ran: a racy path your test never exercised stays hidden. Design with locks or channels instead of relying on the detector alone.

> [!warning] Exit status 66
> A program built with `-race` exits with code 66 when it detected a race, even if it otherwise ran to completion. If a deployment or CI job dies with 66, look for `WARNING: DATA RACE` in the log.

> [!note] Lock the operation, not the declaration
> `RLock` goes around each read, `Lock` around each write, inside whichever goroutine performs it.

> [!tip] Same lock for readers and writers
> Readers using `RLock` are only protected from writers that use `Lock` on the *same* RWMutex. Two different mutexes guard nothing.

> [!note] Match `Add` and `Done`
> `wg.Add(3)` up front or `wg.Add(1)` per goroutine are both fine; what matters is that the total equals the number of `Done` calls.

## Modern Go Notes

- The race detector is unchanged and works on all major platforms. On macOS (including Apple Silicon) `go run -race .` needs no extra setup; on Linux and Windows it requires cgo, so a C compiler must be installed and `CGO_ENABLED=1` (otherwise the build stops with `go: -race requires cgo`).
- Go 1.22's per-iteration loop variables removed the most common *accidental* race, closures capturing a shared loop variable (see [[Loops, Break, Continue and Goto]]). Passing values as arguments, as the video does, was always safe.
- `go vet` cannot find data races (it only finds copied locks); the `-race` flag is the tool.

## Video Timeline

| Time | What happens |
|---|---|
| [00:35](https://youtu.be/4TCNUa-5wmI?t=35) | What a race condition is: two goroutines, one memory location |
| [02:20](https://youtu.be/4TCNUa-5wmI?t=140) | Immediately invoked anonymous functions as goroutines |
| [04:34](https://youtu.be/4TCNUa-5wmI?t=274) | `wg := &sync.WaitGroup{}` and passing `*sync.WaitGroup` |
| [06:13](https://youtu.be/4TCNUa-5wmI?t=373) | `wg.Add(3)` once vs `Add(1)` per goroutine |
| [08:17](https://youtu.be/4TCNUa-5wmI?t=497) | Running it: the order changes, the output looks fine |
| [09:38](https://youtu.be/4TCNUa-5wmI?t=578) | `go run -race`: WARNING: DATA RACE and exit status 66 |
| [10:54](https://youtu.be/4TCNUa-5wmI?t=654) | `mut := &sync.Mutex{}`, `Lock`/`Unlock` around every write |
| [14:05](https://youtu.be/4TCNUa-5wmI?t=845) | Switching to `RWMutex`, `RLock`/`RUnlock` for the reader |
| [15:39](https://youtu.be/4TCNUa-5wmI?t=939) | Where the read lock belongs: around the read, not the declaration |

## Related Notes

- [[Mutex]] — the lock itself, explained in the previous video.
- [[WaitGroups]] — `Add`, `Done`, `Wait` and why `wg` is a pointer here.
- [[Concurrency and Goroutines]] — goroutines are scheduled in no guaranteed order.
- [[Channels and Deadlock]] — the next video: communicating instead of sharing.
- [[Functions]] — anonymous functions and function literals.
- [[Pointers]] — `&sync.WaitGroup{}` and `*sync.RWMutex` parameters.
- [[Slices]] — why `append` from several goroutines corrupts the header.
- [[Go Quick Reference]] — `go run -race .` in the toolchain table.
- [[Golang Notes]] — map of content.
