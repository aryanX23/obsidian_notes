---
tags:
  - golang
  - basics
section: "02 Basics"
video: https://www.youtube.com/watch?v=xsAz24z4Hdg
video_title: "Handling time in golang"
order: 9
---
# Handling Time

> [!info] Source
> 🎬 [Handling time in golang](https://www.youtube.com/watch?v=xsAz24z4Hdg) · Section **02 Basics** of [[Golang Notes]]
> ⬅️ [[Type Conversions]] · ➡️ [[Building for Windows, Linux and Mac]]

## Overview

Time is, in the instructor's words, "the most mismanaged thing" in programming, and Go's **`time`** package has one feature that trips up everyone: formatting is done by writing out a **reference date** rather than `YYYY-MM-DD` style codes. This video reads the `time` documentation, gets the current time with `time.Now()`, formats it, and then goes the other way by constructing a date with `time.Date`. The `time.Sleep` function mentioned here returns in [[Concurrency and Goroutines]], and `UnixNano` is used as a random seed in [[Switch Case]] and [[Math, Crypto and Random Numbers]].

## Project setup

```text
06mytime/
├── go.mod      ← go mod init mytime
└── main.go
```

(The folder is numbered 06 because the companion repository keeps `05mymaths` for the video on [[Math, Crypto and Random Numbers]].)

## The `time` package at a glance

Open https://pkg.go.dev/time. It is not a huge package, but it has its own vocabulary:

| Piece | What it is |
|---|---|
| `time.Time` | a point in time (date, clock, nanoseconds, location) |
| `time.Duration` | a length of time in nanoseconds; constants `time.Nanosecond` ... `time.Hour` |
| `time.Month`, `time.Weekday` | named integer types: `time.August`, `time.Monday` |
| `time.Location` | a time zone: `time.UTC`, `time.Local`, or one loaded by name |
| `time.Now()`, `time.Date(...)`, `time.Parse(...)` | the three ways to get a `Time` |
| `time.Sleep`, `time.After`, `time.Tick` | pausing and timers; used with goroutines later |

Methods on a `Time` value do the rest: `Format`, `Year`, `Month`, `Day`, `Weekday`, `Hour`, `Add`, `AddDate`, `Sub`, `Before`, `After`, `Unix`, `UnixNano`.

## Getting the current time

```go
presentTime := time.Now()
fmt.Println(presentTime)
```

**Output:**
```text
2026-10-23 15:06:19.910071 +0530 IST m=+0.000286834
```

Reading it left to right: the date, the clock time with microseconds, the zone offset and abbreviation (`+0530 IST`, Indian Standard Time on the instructor's machine), and `m=+0.000286834`, a **monotonic clock** reading Go attaches so that measuring elapsed time is immune to the wall clock being adjusted. It is correct and complete, but not what you would show a user.

## Formatting with the reference time

To change the layout you call `Format` and pass a **layout string**. This is the surprising part: instead of codes like `%Y`, you write the *specific reference date* laid out the way you want, and Go substitutes the real values in the same positions.

The reference moment is:

```text
Mon Jan 2 15:04:05 MST 2006
```

which is `01/02 03:04:05PM '06 -0700` when written numerically: month **1**, day **2**, hour **3** (15 in 24-hour form), minute **4**, second **5**, year **6** (2006), zone **7** (-0700). Memorise "1 2 3 4 5 6 7" and the layouts write themselves.

| Want | Write in the layout | Example result |
|---|---|---|
| Year, 4 digits | `2006` | `2021` |
| Year, 2 digits | `06` | `21` |
| Month, 2 digits | `01` | `08` |
| Month name | `January` / `Jan` | `August` / `Aug` |
| Day of month | `02` | `18` |
| Weekday name | `Monday` / `Mon` | `Wednesday` / `Wed` |
| Hour, 24-hour | `15` | `23` |
| Hour, 12-hour | `03` | `11` |
| Minute | `04` | `10` |
| Second | `05` | `28` |
| AM/PM | `PM` | `PM` |
| Zone abbreviation | `MST` | `IST` |
| Zone offset | `-0700` | `+0530` |
| Fractional seconds | `.000` | `.910` |

Any other characters (`-`, `/`, `:`, spaces, words that are not reference tokens) are copied through unchanged.

```go
fmt.Println(presentTime.Format("01-02-2006 15:04:05 Monday"))
```

**Output** (for the instructor's run on Wednesday 18 August 2021):
```text
08-18-2021 23:10:28 Wednesday
```

> [!warning] Always use the reference values, never today's values
> Writing `Monday` in the layout does not print "Monday"; it prints *whatever weekday the time actually is*. Writing `2021` instead of `2006` does not print the year either: the digits are still scanned as tokens (`2` and `02` are day tokens, `1` is a month token), so for 18 August 2021 it prints the garbage `18188`. Only text that contains no reference tokens, such as `Tuesday`, passes through literally. The instructor keeps this reference date on a sticky note on his desk; the docs for `time.Layout` and the executable example for `Time.Format` show every token.

![[09_time_format_layout_docs_0710.png]]
*Figure: the Time.Format example in the time package documentation, showing the reference layout (video at 07:10)*

### Ready-made layouts

The package defines constants for common formats, for example:

```go
t.Format(time.RFC3339)   // 2021-08-18T23:10:28+05:30
t.Format(time.Kitchen)   // 11:10PM
t.Format(time.DateOnly)  // 2021-08-18   (Go 1.20+)
t.Format(time.DateTime)  // 2021-08-18 23:10:28   (Go 1.20+)
```

## Creating a time from values: `time.Date`

The reverse direction, building a `Time` from numbers you choose:

```go
createdDate := time.Date(2020, time.August, 12, 23, 23, 0, 0, time.UTC)
fmt.Println(createdDate)
fmt.Println(createdDate.Format("01-02-2006 Monday"))
```

**Output:**
```text
2020-08-12 23:23:00 +0000 UTC
08-12-2020 Wednesday
```

The signature is `Date(year int, month Month, day, hour, min, sec, nsec int, loc *Location)`. Two things to notice:

- The **month is not an `int`**; it is `time.Month`, so you write `time.August` (typing `time.` in the editor lists all twelve). Everything else is a plain `int`.
- The last argument is the **location** (time zone). `time.UTC` is the safe default for stored timestamps; `time.Local` uses the machine's zone; `time.LoadLocation("Asia/Kolkata")` loads a named zone and returns a `(*Location, error)` pair.

Change the day to 10 and the formatted weekday becomes `Monday`; the weekday is derived, never typed.

## Example: the complete program

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println("Welcome to time study of golang")

	presentTime := time.Now()
	fmt.Println(presentTime)

	fmt.Println(presentTime.Format("01-02-2006 15:04:05 Monday"))

	createdDate := time.Date(2020, time.August, 12, 23, 23, 0, 0, time.UTC)
	fmt.Println(createdDate)
	fmt.Println(createdDate.Format("01-02-2006 Monday"))
}
```

**Output** (the first two lines depend on when you run it):
```text
Welcome to time study of golang
2026-10-23 15:06:19.910071 +0530 IST m=+0.000286834
10-23-2026 15:06:19 Friday
2020-08-12 23:23:00 +0000 UTC
08-12-2020 Wednesday
```

## Extra: parsing and arithmetic

Beyond the video, the operations you will reach for most often when writing APIs:

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t := time.Date(2021, time.August, 18, 23, 10, 28, 0, time.UTC)

	// pieces of a time
	fmt.Println(t.Year(), t.Month(), t.Day(), t.Weekday(), t.Hour())

	// arithmetic with Duration and calendar units
	fmt.Println(t.Add(48 * time.Hour).Format("02 Jan 2006 Monday"))
	fmt.Println(t.AddDate(0, 1, 0).Format("02 Jan 2006"))
	later := t.Add(90 * time.Minute)
	fmt.Println(later.Sub(t), later.After(t))

	// text to time: same reference-layout idea in reverse
	parsed, err := time.Parse("2006-01-02", "2024-12-25")
	fmt.Println(parsed, err)

	// Unix timestamps
	fmt.Println(t.Unix(), t.UnixNano())
}
```

**Output:**
```text
2021 August 18 Wednesday 23
20 Aug 2021 Friday
18 Sep 2021
1h30m0s true
2024-12-25 00:00:00 +0000 UTC <nil>
1629328228 1629328228000000000
```

`time.Parse` uses the comma ok pattern from [[User Input and Comma Ok Syntax]]: the layout tells it how to read the text, and `err` is set if the text does not match.

## Gotchas and Best Practices

> [!warning] The layout is a date, not a pattern
> `"01-02-2006"` is month-day-year (the American reference date). If you want day-month-year, write `"02-01-2006"`. Swapping them is the classic bug, because both compile.

> [!warning] Month is `time.Month`
> `time.Date(2020, 8, ...)` works only because `8` is an untyped constant; a variable holding an `int` must be converted: `time.Month(m)`. See [[Type Conversions]].

> [!tip] Store and compare in UTC, format in the user's zone
> Keep `time.UTC` for anything you persist (databases, JSON) and convert at the edge with `t.In(loc)`.

> [!tip] Use `Duration` constants for readable arithmetic
> `48 * time.Hour`, `500 * time.Millisecond`. A bare number is nanoseconds, which is almost never what you meant. This matters for `time.Sleep` in [[Concurrency and Goroutines]].

> [!note] Playground time is frozen
> On the online playground the clock does not behave like your machine's, one of the reasons the instructor avoids it (see [[Installation and Hello World]] and [[Switch Case]]).

## Modern Go Notes

- Go 1.20 added the layout constants `time.DateTime`, `time.DateOnly` and `time.TimeOnly` for the common `2006-01-02 15:04:05` family, so you do not have to spell those out.
- Everything else in this note is unchanged since the video; the `time` package API is very stable.

## Video Timeline

| Time | What happens |
|---|---|
| [00:28](https://youtu.be/xsAz24z4Hdg?t=28) | New folder `mytime`, `go mod init mytime` |
| [00:54](https://youtu.be/xsAz24z4Hdg?t=54) | Reading the `time` docs: functions, index, `After`, `Sleep`, `Tick` |
| [01:45](https://youtu.be/xsAz24z4Hdg?t=105) | Finding `Format` and its layout example in the docs |
| [03:02](https://youtu.be/xsAz24z4Hdg?t=182) | `time.Now()` and its raw output |
| [04:21](https://youtu.be/xsAz24z4Hdg?t=261) | `Format("01-02-2006")` and why those exact numbers |
| [05:39](https://youtu.be/xsAz24z4Hdg?t=339) | Adding `Monday` and `15:04:05` to the layout |
| [07:49](https://youtu.be/xsAz24z4Hdg?t=469) | `time.Date(...)` with `time.August` and `time.UTC` |
| [09:58](https://youtu.be/xsAz24z4Hdg?t=598) | Formatting the created date; weekday is derived |
| [11:15](https://youtu.be/xsAz24z4Hdg?t=675) | Tickers and `time.Sleep` to come later |

## Related Notes

- [[Building for Windows, Linux and Mac]] — next: the `mytime` program is cross-compiled there.
- [[Math, Crypto and Random Numbers]] and [[Switch Case]] — `time.Now().UnixNano()` as a random seed.
- [[Concurrency and Goroutines]] — `time.Sleep` and durations with goroutines.
- [[Type Conversions]] — converting ints to `time.Month`, strings to times with `Parse`.
- [[User Input and Comma Ok Syntax]] — the `(value, error)` return of `time.Parse` and `LoadLocation`.
- [[Creating JSON Data]] — timestamps are usually serialised in RFC 3339 form.
- [[fmt Package and Format Verbs]] — printing times with `%v`.
- [[Go Quick Reference]] — the layout tokens table in short form.
