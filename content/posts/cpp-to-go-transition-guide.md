---
title: "C++ to Go for SRE: The Mental Shifts That Actually Matter"
date: 2026-04-16
draft: false
tags: ["go", "cpp", "sre", "devops", "kubernetes", "career"]
summary: "A field guide for engineers switching their default language from C++ to Go — covering DSA, SRE dev work, the traps that waste your first month, and the five mindset shifts that make Go click."
---

If you're a C++ engineer moving toward SRE, DevOps, or platform work, Go is waiting for you. Kubernetes is Go. Prometheus is Go. Terraform is Go. Docker is Go. The tools you'll live inside for the next decade of your career are written in Go, and being fluent — not tourist-level, fluent — is the entry fee.

I started this transition this week. I'm still in hour-zero territory, but I've already hit a few walls that nobody on YouTube warned me about, and I've also seen enough engineers at my company (Deloitte USI) try this same switch and bounce off because they kept writing C++ in Go's clothing.

This is a field report, not a language tutorial. If you're making the same switch, here's what I wish someone had told me on Day 1.

---

## Part 1 — What C++ → Go feels like for DSA

If your C++ muscle memory comes from LeetCode, the first hour in Go will feel **worse**, not better. That's normal. You're trading away a rich, feature-dense toolkit for one that's deliberately austere. Here's what trips up most people.

### Trap 1: "Where is my `priority_queue`?"

C++ gives you `std::priority_queue<int>` in one line. Go's heap lives in `container/heap`, and it doesn't give you a usable heap — it gives you an **interface** you must implement yourself: `Len()`, `Less()`, `Swap()`, `Push()`, `Pop()`. For your first Dijkstra or Top-K problem, you'll stare at 30 lines of boilerplate and question your life choices.

**Fix:** Write the heap ONCE. Save it in a personal `dsa-snippets/` file. Copy-paste forever. Every experienced Go DSA solver has this snippet — nobody rewrites it each time.

### Trap 2: Maps have no order

`std::map<K,V>` in C++ is a red-black tree — ordered by key. Go's `map[K]V` is a hash table, **unordered by design**. Worse: Go deliberately **randomizes iteration order** each run so you don't accidentally rely on it.

This catches people on problems like "group anagrams and return in some order" — their tests pass locally, fail on submit because the random iteration gave a different grouping.

**Fix:** When order matters, collect keys, `sort.Strings(keys)`, then iterate. Don't fight the map.

### Trap 3: No `std::set`

No built-in set either. Idiom: `map[K]struct{}{}` where `struct{}` is a zero-byte placeholder. Clunky, but that's the cost of a small standard library.

```go
seen := map[int]struct{}{}
seen[42] = struct{}{}
if _, ok := seen[42]; ok { /* ... */ }
```

**Fix:** Learn this idiom in Week 1 and stop fighting it.

### Trap 4: Strings are byte sequences, not character sequences

In C++, `s[0]` gives you a `char`. In Go, `s[0]` gives you a **byte** — and if the string has any Unicode, that byte is half of a character. This will quietly break your "reverse a string" solution for non-ASCII input.

**Fix:** For character-wise work, convert: `runes := []rune(s)`. Now `runes[0]` is a full character.

### Trap 5: Slices look like arrays but aren't

A slice is a (pointer, length, capacity) triple pointing into an underlying array. `append` sometimes mutates in place, sometimes allocates a new array — you can't tell without thinking. This creates aliasing bugs:

```go
a := []int{1, 2, 3}
b := a           // b and a share backing array
b = append(b, 4) // may or may not affect a
```

**Fix:** When you pass slices around in recursion (backtracking, DFS), **copy** them or pre-allocate. When in doubt, use `make([]T, 0, cap)` up front.

### The DSA silver lining

Go's simpler, flatter syntax means your **solutions read faster to interviewers** than C++ solutions. No `std::` prefixes, no templates, no `auto& const`. A clean Go solution is often 30% fewer lines and 100% more readable than the C++ equivalent.

Once you're past Week 2, you'll be solving mediums faster in Go than you were in C++.

---

## Part 2 — What C++ → Go feels like for SRE/Dev work

DSA is the shallow end. Real Go work — services, tools, infrastructure code — has its own pain curve.

### Trap 1: Error handling is everywhere, and it's verbose

C++ throws exceptions. Go returns errors as values. Every function call that can fail becomes:

```go
result, err := doThing()
if err != nil {
    return fmt.Errorf("doThing failed: %w", err)
}
```

Your first Go service will feel 40% `if err != nil` by volume. You'll hate it.

Then, around Week 2, something clicks: you realize you can **see** every failure path. No hidden exceptions bubbling up from five layers deep. Errors become data, and data becomes visible.

**Fix:** Don't fight it. Embrace `errors.Is`, `errors.As`, and wrap errors with `%w` for context. It's verbose — it's also the reason Go services don't mysteriously explode in production.

### Trap 2: No classes. Really, none.

Go has `struct`s and methods. It has **no classes, no inheritance, no constructors**. If you've spent years in C++ building class hierarchies, you'll keep reaching for `extends` and it won't be there.

```go
type Server struct {
    addr string
    log  *slog.Logger
}

func (s *Server) Start() error { /* ... */ }
```

That's "a class" in Go. That's all there is.

**Fix:** Embrace composition over inheritance. If you need polymorphism, define an interface and let any struct satisfy it **implicitly**. You don't declare "implements" — Go figures it out.

```go
type Startable interface { Start() error }
// Server implements Startable automatically because it has a Start() method.
```

This feels weird for two days. Then it feels liberating.

### Trap 3: `context.Context` is in every function signature

In C++, cancellation and deadlines are whatever you build. In Go, there's **one canonical way**: `context.Context`, threaded through every function call.

```go
func FetchUser(ctx context.Context, id string) (*User, error)
```

As an SRE, you'll see `ctx` everywhere. Request timeouts, graceful shutdowns, request-scoped logging — all carried by context. Most real bugs in Go services are "I forgot to pass ctx" or "I ignored ctx.Done()."

**Fix:** From Day 1, accept `ctx context.Context` as the first parameter of any function that does I/O. Check `ctx.Err()` inside loops. It becomes muscle memory.

### Trap 4: Concurrency is cheap, so you will misuse it

Goroutines are almost free. `go doThing()` spawns a new "thread" that costs ~2KB of stack. So you'll spawn 10,000 of them and discover you can't coordinate them.

C++ threads were expensive; you thought carefully before spawning. Go goroutines are cheap; you stop thinking.

**Fix:** Learn three patterns and use them religiously:
- **Worker pool** with buffered channels for bounded parallelism
- **`errgroup.Group`** (from `golang.org/x/sync/errgroup`) for "run N tasks, cancel on first error"
- **`sync.WaitGroup`** for "wait until all N are done"

If you reach for raw goroutines in production code without one of these, you're writing a race condition generator.

### Trap 5: The standard library is bigger than you expect

In C++, Boost exists because stdlib was small. In Go, stdlib **is** the ecosystem for 80% of SRE work:
- `net/http` — production-grade HTTP client + server
- `encoding/json` — JSON marshal/unmarshal
- `log/slog` — structured logging
- `context` — cancellation/deadlines
- `sync` — locks, waitgroups, atomics
- `time` — everything deadline-related
- `os/exec` — shell out to subprocesses
- `flag` — CLI argument parsing

Before you pull in a library, search stdlib. Nine times out of ten, it's already there.

---

## Part 3 — Five mindset shifts that make Go click

These aren't syntax. They're orientation changes. Get them right and Go stops feeling like "C++ with features removed" and starts feeling like a different, smaller, sharper tool.

**1. Stop optimizing before measuring.** Go's simplicity is the feature. If you're writing custom allocators or reaching for `unsafe`, you're probably wrong. Write the obvious code, profile it with `pprof`, optimize only the hot path.

**2. Flat code beats layered code.** In C++ you build elaborate type hierarchies. In Go, you write a function. Three files of 200 lines each beats thirty files of 20 lines each for most services.

**3. Interfaces are small and defined at the point of use.** Don't declare interfaces up front like in Java. Define them where they're consumed. The canonical example — `io.Reader` has one method: `Read([]byte) (int, error)`. That's the entire interface. Steal that taste.

**4. `gofmt` and `go vet` are non-negotiable.** There is no style debate in Go because the tools win every argument. Run `gofmt -w .` on save. Add `golangci-lint` to your editor. Stop thinking about formatting.

**5. Read the stdlib.** Unlike most languages, Go's stdlib source is small, idiomatic, and readable. When you wonder "how do you do X in Go?" — search for X in the stdlib and read how the Go team did it. That's your style guide.

---

## What I'm actually doing this month

I'm 21 days into my SRE-focused Go plan:

- **Week 1:** A Tour of Go (go.dev/tour) — syntax fluency
- **Week 2:** Go by Example — idiomatic patterns, with a focus on concurrency, HTTP, and CLI tooling
- **Week 3:** Building `hc` — a concurrent URL health-check CLI that reads URLs from a file, probes them in a worker pool, and emits JSON for scripting

In parallel, I'm re-solving my core LeetCode patterns (two pointers, sliding window, hashmap, heap, BFS/DFS) in Go. The goal isn't to re-grind easy problems — it's to build Go reflexes on patterns I already understand, so that in an interview, Go is faster than C++.

If you're doing the same transition, the thing I'd tell my Day-1 self: **don't try to speed-run it**. Go is a smaller language than C++, but "small" means every feature has a right way to use it, and the wrong way costs you. Spend the first two weeks reading idiomatic code before writing much of your own.

I'll write up the `hc` tool once it's shipped. Follow along at [umeshdas.dev](/) if you're on the same path.

---

*If this helped, send it to one person making the same switch. That's the whole reason I'm publishing.*
