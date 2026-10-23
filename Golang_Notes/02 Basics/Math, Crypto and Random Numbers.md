---
tags:
  - golang
  - basics
section: "02 Basics"
video: https://www.youtube.com/watch?v=yPeYZ0QuspU
video_title: "Math, crypto and random number in golang"
order: 57
---
# Math, Crypto and Random Numbers

> [!info] Source
> 🎬 [Math, crypto and random number in golang](https://www.youtube.com/watch?v=yPeYZ0QuspU) · Section **02 Basics** of [[Golang Notes]]
> ⬅️ [[Building for Windows, Linux and Mac]] · ➡️ [[Memory Management]]

## Overview

The instructor frames this video with a tiny product idea: an API that returns a number between 1 and 6 so anyone building a dice game can call it (the API part comes much later, in [[Building an API - Models]]). Getting that number turns out to teach three things: why Go refuses to add an `int` to a `float64` (a reminder of [[Type Conversions]]), how **pseudo-random** numbers from `math/rand` are driven by a **seed**, and how **cryptographically secure** numbers from `crypto/rand` differ. Random numbers are, in his words, "the most confusing topic for beginners", mostly because of the seed. Note that this video sits last in the playlist but its companion folder is `05mymaths`; it was recorded alongside the basics, which is why it lives in this section.

## Types Still Matter: `int` + `float64`

```go
var myNumberOne int = 2
var myNumberTwo float64 = 4.5
fmt.Println("The sum is:", myNumberOne+myNumberTwo)
```

This does not compile:

```text
invalid operation: myNumberOne + myNumberTwo (mismatched types int and float64)
```

Go never converts numeric types silently (see [[Lexer and Types]] and [[Variables, Types and Constants]]). You choose the conversion, and the choice has consequences:

```go
package main

import "fmt"

func main() {
	var myNumberOne int = 2
	var myNumberTwo float64 = 4.5

	fmt.Println("int(float) loses the fraction:", myNumberOne+int(myNumberTwo))
	fmt.Println("float64(int) keeps it:        ", float64(myNumberOne)+myNumberTwo)
}
```

**Output:**

```text
int(float) loses the fraction: 6
float64(int) keeps it:         6.5
```

`int(4.5)` truncates to `4`. The instructor's warning: imagine the `.5` is half a crore of currency. Convert the *integer* to a float, not the float to an integer, unless you really mean to drop the fraction.

## Two Kinds of Random

Go keeps two separate packages on purpose:

| | `math/rand` | `crypto/rand` |
|---|---|---|
| Kind | **Pseudo-random**: an algorithm driven by a seed; the same seed produces the same sequence | **Cryptographically secure**: reads from the operating system's entropy source |
| Speed | Very fast | Slower |
| Use for | Games, simulations, shuffling, tests, load balancing | Passwords, tokens, session ids, keys, anything an attacker must not guess |
| Main function in the video | `rand.Intn(n)` returns an `int` in `[0, n)` | `rand.Int(rand.Reader, max)` returns a `*big.Int` in `[0, max)` and an `error` |

The documentation line the instructor highlights for `Intn`: it returns "a non-negative pseudo-random number in the half-open interval [0,n)" and *panics if n <= 0* (`go doc math/rand.Intn`). The upper bound is **exclusive**, as in Python's `range`, so a dice needs `Intn(6) + 1`, never `Intn(6)`.

## `math/rand`: the Seed Story

```go
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	fmt.Println("Dice:", rand.Intn(6)+1) // Intn(6) gives 0..5, so add 1
}
```

**Output:** (three runs on Go 1.27)

```text
Dice: 6
Dice: 6
Dice: 1
```

When the video was recorded this program printed the **same number on every run**, and that is the moment that "gets a lot of people crazy". Nothing in a computer is random; `math/rand` runs an algorithm that starts from a **seed**, and older Go versions always started from seed `1`, so every run replayed the same sequence. The instructor's walk-through:

1. `rand.Seed(30)` changes the sequence, but it is still the *same* sequence on every run: he gets `3` every time. (With the deterministic behaviour restored, see Modern Go Notes, `Seed(30)` followed by `Intn(5)` still prints `3` on Go 1.27: same algorithm, same answer.)
2. Changing the seed to `302` or `7` gives a different but equally fixed sequence.
3. The fix is to seed with something that is different on every run, and the thing that always changes is the clock: `rand.Seed(time.Now().UnixNano())`. The `time` package gets its own video, [[Handling Time]].

Then `Intn(6)` finally varies between runs, and `+ 1` turns `0..5` into the dice range `1..6`. Because `Intn` returns an `int`, no conversion is needed for the `+ 1`, which is exactly the kind of detail reading the documentation saves you from guessing.

## `crypto/rand`: the Secure Version

The crypto package has fewer functions ("prime numbers are there, it is cryptography, it has to"), and its `Int` is more demanding: it wants a *source of randomness* (`rand.Reader`, an `io.Reader` backed by the OS) and an upper bound as a **`*big.Int`**, and it returns a `*big.Int` **and an error**, so the comma-ok pattern from [[User Input and Comma Ok Syntax]] applies.

```go
package main

import (
	"crypto/rand"
	"fmt"
	"math/big"
)

func main() {
	myRandomNum, err := rand.Int(rand.Reader, big.NewInt(6)) // 0 <= n < 6
	if err != nil {
		panic(err)
	}
	fmt.Println("Dice (crypto):", myRandomNum.Int64()+1)
	fmt.Printf("Type: %T\n", myRandomNum)
}
```

**Output:** (varies)

```text
Dice (crypto): 1
Type: *big.Int
```

Three new things appear here. The import is `crypto/rand`, not `math/rand`; both packages are called `rand`, so a file can only import one of them without an alias, which is why the instructor comments the `math/rand` code out before switching. **`math/big`** provides integers of arbitrary size (`big.NewInt(6)` builds one from an `int64`), which cryptography needs because keys are far larger than 64 bits. And the result is a pointer to a big integer, so `.Int64()` converts it back to a normal number when the value is known to be small.

## The `math` Package Itself

The video only touches `math` through the type error, but the package is where the usual functions and constants live. A few you will reach for, with the output of one program:

```go
package main

import (
	"fmt"
	"math"
	"math/rand/v2"
)

func main() {
	fmt.Println(math.Sqrt(16), math.Pow(2, 10), math.Floor(4.7), math.Ceil(4.2), math.Round(4.5))
	fmt.Println(math.Pi, math.MaxInt64, math.Abs(-3.5))
	fmt.Println("Dice (v2):", rand.IntN(6)+1)
	fmt.Println("Float:", rand.Float64())
	fmt.Println("N:", rand.N(6)+1)
}
```

**Output:** (the last three lines vary)

```text
4 1024 4 5 5
3.141592653589793 9223372036854775807 3.5
Dice (v2): 2
Float: 0.5478064125591865
N: 2
```

All `math` functions work on `float64`, so integers must be converted first: `math.Sqrt(float64(n))`.

## Example: The Companion Program

The file in folder `05mymaths` keeps the `math/rand` experiments commented out and ends with the crypto version:

```go
package main

import (
	"fmt"
	"math/big"

	//"math/rand"
	"crypto/rand"
)

func main() {
	fmt.Println("Welcome to maths in golang")

	//var mynumberOne int = 2
	//var mynumberTwo float64 = 4.5

	// fmt.Println("The sum is: ", mynumberOne+int(mynumberTwo))

	//random number
	// rand.Seed(time.Now().UnixNano())
	// fmt.Println(rand.Intn(5) + 1)

	//random from crypto

	myRandomNum, _ := rand.Int(rand.Reader, big.NewInt(5))
	fmt.Println(myRandomNum)

}
```

**Output:** (a number from 0 to 4; varies)

```text
Welcome to maths in golang
4
```

The `_` throws the error away, which the instructor admits is not what you should do "in the ideal case"; the version earlier in this note handles it.

## Gotchas and Best Practices

> [!warning] The upper bound is exclusive
> `rand.Intn(6)` never returns `6`. For a range `[low, high]` use `rand.Intn(high-low+1) + low`.

> [!warning] `Intn` panics on `n <= 0`
> `rand.Intn(0)` crashes with `panic: invalid argument to Intn`. Validate user-supplied bounds first.

> [!warning] Never use `math/rand` for secrets
> Its sequence is reproducible by design. Tokens, passwords, OTPs and keys come from `crypto/rand`, and its `error` must be checked, because a failing entropy source is exactly the situation you must not ignore (see [[Error Handling in Go]]).

> [!tip] Convert the int, not the float
> `float64(i) + f` keeps precision; `i + int(f)` silently truncates.

> [!note] Same name, different packages
> `math/rand` and `crypto/rand` are both imported as `rand`. To use both in one file, alias one: `crand "crypto/rand"`.

## Modern Go Notes

- **Automatic seeding.** Since Go 1.20 the global `math/rand` generator is seeded randomly at start-up, so `rand.Intn(6)` already varies between runs and the `rand.Seed(time.Now().UnixNano())` line is unnecessary. `go doc math/rand.Seed` marks `Seed` as *deprecated* and says programs that want a reproducible sequence should create their own generator with `rand.New(rand.NewSource(seed))`.
- **Top-level `Seed` no longer does anything.** On the Go 1.27 toolchain used for these notes, `rand.Seed(30)` followed by `rand.Intn(5)` printed different values on consecutive runs; the old deterministic behaviour only came back with the environment setting `GODEBUG=randseednop=0`, which then printed `3` every time, the same value the instructor sees. This no-op behaviour arrived in Go 1.24, whose release notes state that calls to the deprecated top-level `Seed` no longer have any effect and that `GODEBUG=randseednop=0` restores the old behaviour.
- **`math/rand/v2`** (Go 1.22+) is the package to use in new code: `rand.IntN(6)`, the generic `rand.N(6)`, `rand.Float64()`, no `Seed` at all, and better algorithms. The companion program's `math/rand` still works and is not going away.
- `crypto/rand` is unchanged; `rand.Int(rand.Reader, max)` is still the call for a bounded secure integer, and `rand.Read` fills a byte slice when you need raw token bytes.

## Video Timeline

| Time | What happens |
|---|---|
| [01:52](https://youtu.be/yPeYZ0QuspU?t=112) | `int` + `float64` is an invalid operation; conversions lose precision |
| [03:50](https://youtu.be/yPeYZ0QuspU?t=230) | Two categories: pseudo-random vs cryptographic |
| [04:49](https://youtu.be/yPeYZ0QuspU?t=289) | Reading `rand.Intn` in the documentation |
| [06:46](https://youtu.be/yPeYZ0QuspU?t=406) | The same "random" number on every run |
| [07:29](https://youtu.be/yPeYZ0QuspU?t=449) | `rand.Seed(30)` always gives `3` |
| [08:20](https://youtu.be/yPeYZ0QuspU?t=500) | Seeding with `time.Now().UnixNano()` |
| [09:12](https://youtu.be/yPeYZ0QuspU?t=552) | `+ 1` for the dice range, no conversion needed |
| [10:00](https://youtu.be/yPeYZ0QuspU?t=600) | Switching to `crypto/rand` |
| [11:35](https://youtu.be/yPeYZ0QuspU?t=695) | `rand.Int(rand.Reader, big.NewInt(5))` and the `big` package |

## Related Notes

- [[Type Conversions]] — explicit numeric conversions and what they lose.
- [[Variables, Types and Constants]] — `int` vs `float64` and typed literals.
- [[Handling Time]] — `time.Now()` and `UnixNano()`, used as a seed here.
- [[User Input and Comma Ok Syntax]] — the `value, err` return of `rand.Int`.
- [[Switch Case]] — the dice game that uses `rand.Intn(6) + 1`.
- [[Error Handling in Go]] — why the crypto error must not be discarded.
- [[Memory Management]] — next note; `big.NewInt` returns a pointer.
- [[Go Quick Reference]] — cheat sheet.
