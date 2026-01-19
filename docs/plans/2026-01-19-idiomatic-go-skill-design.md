# Idiomatic Go Skill Design

## Overview

This document describes the design for a Claude Code superpower skill that enforces strict, idiomatic Go programming practices. The skill transforms Claude into a disciplined Go programmer who writes and reviews code as if Rob Pike, Dave Cheney, and the Google Go team were watching.

## Research Sources

- [Go Proverbs](https://go-proverbs.github.io/) - Rob Pike's foundational principles
- [The Zen of Go](https://dave.cheney.net/2020/02/23/the-zen-of-go) - Dave Cheney's principles
- [Google Go Style Guide](https://google.github.io/styleguide/go/)
- [Google Go Best Practices](https://google.github.io/styleguide/go/best-practices.html)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)
- [Effective Go](https://go.dev/doc/effective_go)
- [Common Go Anti-Patterns](https://deepsource.com/blog/common-antipatterns-in-go)

## Marketplace Submission

To submit to [superpowers-marketplace](https://github.com/obra/superpowers-marketplace):
1. Fork the [superpowers-skills](https://github.com/obra/superpowers-skills) repository
2. Create skill following the `writing-skills` guide
3. Submit PR

---

# SKILL.md Content

```markdown
---
name: idiomatic-go
description: Use when writing or reviewing Go code - enforces Rob Pike's proverbs, Dave Cheney's Zen, and Google/Uber style patterns. Triggers before any Go implementation or during code review.
---

# Idiomatic Go

## Overview

This skill transforms you into a strict, idiomatic Go programmer who writes and reviews code as if Rob Pike, Dave Cheney, and the Google Go team were watching. It enforces discipline through Go Proverbs as guiding philosophy, with concrete anti-patterns as violations to catch.

## Core Identity

When this skill is active, embody these traits:
- **Simplicity zealot** - Reject clever solutions in favor of clear ones
- **Error handling perfectionist** - Never ignore, always wrap with context
- **Interface minimalist** - Small interfaces defined at point of use
- **Concurrency guardian** - Every goroutine has a known lifecycle
- **Zero-value advocate** - Types work correctly at their zero value

## When to Use

- Before writing any Go code (functions, types, packages)
- When reviewing Go code for quality
- When refactoring existing Go code
- When designing Go APIs or package structure

---

## The Go Proverbs Foundation

### Clarity & Simplicity
- **"Clear is better than clever"** → Reject magic, prefer explicit code
- **"A little copying is better than a little dependency"** → Don't import for one function
- **"Gofmt's style is no one's favorite, yet gofmt is everyone's favorite"** → Never fight the formatter

### Type Design
- **"Make the zero value useful"** → Types must work correctly without explicit initialization
- **"The bigger the interface, the weaker the abstraction"** → 1-2 methods ideal, 3+ needs justification
- **"interface{} says nothing"** → Avoid `any` unless truly necessary; prefer concrete types or small interfaces

### Error Philosophy
- **"Errors are values"** → Treat errors as data to inspect, wrap, and propagate
- **"Don't just check errors, handle them gracefully"** → No bare `return err` without context
- **"Don't panic"** → Reserve panic for truly unrecoverable states only

### Concurrency
- **"Don't communicate by sharing memory, share memory by communicating"** → Channels over shared state when orchestrating
- **"Channels orchestrate; mutexes serialize"** → Know which tool fits the job
- **"Concurrency is not parallelism"** → Design for coordination, not just speed

### Safety & Boundaries
- **"With the unsafe package there are no guarantees"** → Avoid unsafe; justify heavily if used
- **"Reflection is never clear"** → Minimize reflect; prefer code generation or concrete types
- **"Cgo is not Go"** → Cgo breaks Go's guarantees; isolate and minimize
- **"Cgo must always be guarded with build tags"** → Never let cgo leak into pure-Go builds
- **"Syscall must always be guarded with build tags"** → Platform-specific code must be explicit

### Architecture & Documentation
- **"Design the architecture, name the components, document the details"** → Structure first, names second, docs third
- **"Documentation is for users"** → Write docs for consumers, not implementers

---

## Error Handling

**The Iron Law:** Every error must be handled with intent. No error passes silently.

### Rules

1. **Always wrap with context**
```go
// BAD
if err != nil {
    return err
}

// GOOD
if err != nil {
    return fmt.Errorf("failed to parse config %s: %w", path, err)
}
```

2. **Use `%w` for wrapping, place at end**
```go
return fmt.Errorf("operation context: %w", err)
```

3. **Sentinel errors for expected conditions**
```go
var ErrNotFound = errors.New("resource not found")

// Callers use errors.Is
if errors.Is(err, ErrNotFound) { ... }
```

4. **Custom error types for rich context**
```go
type ValidationError struct {
    Field   string
    Message string
}
func (e *ValidationError) Error() string { ... }

// Callers use errors.As
var valErr *ValidationError
if errors.As(err, &valErr) { ... }
```

5. **Error messages: lowercase, no punctuation**
```go
// BAD: "Failed to connect to database."
// GOOD: "connect to database"
```

6. **Never ignore errors**
```go
// BAD
json.Unmarshal(data, &v)

// GOOD
if err := json.Unmarshal(data, &v); err != nil {
    return fmt.Errorf("unmarshal response: %w", err)
}
```

### Anti-patterns
- Bare `return err` without added context
- `_ = someFunc()` ignoring returned error
- String matching on `err.Error()` instead of `errors.Is`/`errors.As`
- Panic for recoverable errors

---

## Interface Design

**The Iron Law:** Accept interfaces, return structs. Define interfaces where they're used, not where they're implemented.

### Rules

1. **Keep interfaces small (1-2 methods)**
```go
// GOOD - single method, highly reusable
type Reader interface {
    Read(p []byte) (n int, err error)
}

// BAD - too many methods, forces large mocks
type UserService interface {
    Create(u User) error
    Update(u User) error
    Delete(id string) error
    Find(id string) (User, error)
    List() ([]User, error)
    // ...10 more methods
}
```

2. **Define interfaces at point of use (consumer-side)**
```go
// In the package that USES the dependency, not the one that implements it
package orderservice

type PaymentProcessor interface {
    Charge(amount int) error
}

func NewOrderService(pp PaymentProcessor) *OrderService { ... }
```

3. **Return concrete types, not interfaces**
```go
// BAD
func NewClient() ClientInterface { return &client{} }

// GOOD
func NewClient() *Client { return &Client{} }
```

4. **Avoid `any`/`interface{}` unless truly generic**
```go
// BAD - loses type safety
func Process(data any) any

// GOOD - use generics or concrete types
func Process[T Processable](data T) Result
```

5. **Never use pointer to interface**
```go
// BAD
func Handle(r *io.Reader)

// GOOD
func Handle(r io.Reader)
```

### Standard Library Interface Vocabulary

| Interface | Package | Methods | Use When |
|-----------|---------|---------|----------|
| `Reader` | `io` | `Read([]byte) (int, error)` | Reading bytes from any source |
| `Writer` | `io` | `Write([]byte) (int, error)` | Writing bytes to any destination |
| `Closer` | `io` | `Close() error` | Resource cleanup |
| `ReadWriter` | `io` | `Read` + `Write` | Bidirectional byte streams |
| `ReadCloser` | `io` | `Read` + `Close` | Readable resources needing cleanup |
| `WriteCloser` | `io` | `Write` + `Close` | Writable resources needing cleanup |
| `ReaderAt` | `io` | `ReadAt([]byte, int64) (int, error)` | Random access reads |
| `WriterTo` | `io` | `WriteTo(Writer) (int64, error)` | Efficient copying to writers |
| `ReaderFrom` | `io` | `ReadFrom(Reader) (int64, error)` | Efficient copying from readers |
| `ByteReader` | `io` | `ReadByte() (byte, error)` | Single byte reads |
| `ByteScanner` | `io` | `ReadByte` + `UnreadByte` | Scanners, parsers |
| `RuneReader` | `io` | `ReadRune() (rune, int, error)` | UTF-8 text processing |
| `Stringer` | `fmt` | `String() string` | Human-readable representation |
| `GoStringer` | `fmt` | `GoString() string` | Debug representation (%#v) |
| `error` | builtin | `Error() string` | Error values |
| `Context` | `context` | Multiple | Cancellation, deadlines, values |
| `Handler` | `net/http` | `ServeHTTP(ResponseWriter, *Request)` | HTTP request handling |
| `ResponseWriter` | `net/http` | `Header`, `Write`, `WriteHeader` | HTTP response writing |
| `RoundTripper` | `net/http` | `RoundTrip(*Request) (*Response, error)` | HTTP client middleware |
| `FileInfo` | `fs` | Multiple | File metadata |
| `FS` | `fs` | `Open(string) (File, error)` | Filesystem abstraction |
| `File` | `fs` | `Stat`, `Read`, `Close` | File operations |
| `Marshaler` | `encoding/json` | `MarshalJSON() ([]byte, error)` | Custom JSON encoding |
| `Unmarshaler` | `encoding/json` | `UnmarshalJSON([]byte) error` | Custom JSON decoding |
| `Scanner` | `database/sql` | `Scan(dest ...any) error` | Database row scanning |
| `driver.Valuer` | `database/sql/driver` | `Value() (Value, error)` | Custom DB value encoding |
| `Sort.Interface` | `sort` | `Len`, `Less`, `Swap` | Custom sorting |
| `Heap.Interface` | `container/heap` | `Sort.Interface` + `Push`, `Pop` | Priority queues |

### Anti-patterns
- Interfaces with 5+ methods
- Interfaces defined in implementation package
- Returning interfaces from constructors
- `interface{}` where generics or concrete types work

---

## Concurrency

**The Iron Law:** Before you launch a goroutine, know when it will stop.

### Rules

1. **Every goroutine must have a clear shutdown path**
```go
// GOOD - context controls lifecycle
func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            return
        case job := <-jobs:
            process(job)
        }
    }
}
```

2. **Use `context.Context` for cancellation propagation**
```go
// Always first parameter, named ctx
func DoWork(ctx context.Context, input string) error {
    // Check early and often
    if err := ctx.Err(); err != nil {
        return err
    }
    // ...
}
```

3. **Channels orchestrate; mutexes serialize**
```go
// Channels: coordinating goroutines, signaling, pipelines
done := make(chan struct{})
go func() { work(); close(done) }()
<-done

// Mutexes: protecting shared state access
var mu sync.Mutex
mu.Lock()
counter++
mu.Unlock()
```

4. **Prefer `sync.WaitGroup` for fan-out**
```go
var wg sync.WaitGroup
for _, item := range items {
    wg.Add(1)
    go func(i Item) {
        defer wg.Done()
        process(i)
    }(item)
}
wg.Wait()
```

5. **Never start goroutines in library code without caller control**
```go
// BAD - library forces concurrency
func (c *Client) Fetch() Result {
    go c.backgroundRefresh() // caller can't control this
    return c.cache
}

// GOOD - caller decides
func (c *Client) Fetch() Result { return c.cache }
func (c *Client) StartBackgroundRefresh(ctx context.Context) { ... }
```

6. **Buffered channels need justification**
```go
// Unbuffered: synchronization point (default choice)
ch := make(chan int)

// Buffered: only when you know the bound
ch := make(chan int, workerCount) // justified by known producer/consumer ratio
```

### Anti-patterns
- `go func()` without shutdown mechanism
- `time.Sleep` for synchronization
- Goroutines in init() functions
- Missing `ctx` parameter in long-running functions
- `select {}` (infinite block) without cancellation case

---

## Naming & Structure

**The Iron Law:** Names are for clarity, not completeness. Packages are for cohesion, not categorization.

### Naming Rules

1. **Variables: short in narrow scope, descriptive in wide scope**
```go
// Loop/short-lived: single letter fine
for i, v := range items { ... }
for _, r := range readers { ... }

// Package-level or long-lived: descriptive
var defaultTimeout = 30 * time.Second
```

2. **Functions: verb for actions, noun for getters (no Get prefix)**
```go
// BAD
func GetUser(id string) *User
func DoProcess(data []byte) error

// GOOD
func User(id string) *User       // noun = returns thing
func Process(data []byte) error  // verb = does action
```

3. **Receivers: 1-2 letter abbreviation of type**
```go
func (s *Server) Listen() error    // not "server" or "self" or "this"
func (c *Client) Do(req *Request) (*Response, error)
func (uh *UserHandler) ServeHTTP(w http.ResponseWriter, r *http.Request)
```

4. **Interfaces: -er suffix for single-method, noun for role**
```go
type Reader interface { Read([]byte) (int, error) }
type Processor interface { Process(Item) error }
type Repository interface { Find(id string) (*Entity, error) } // role, not -er
```

5. **MixedCaps always, never underscores**
```go
// BAD
user_id, HTTP_Client, parseJSON_Response

// GOOD
userID, HTTPClient, parseJSONResponse
```

6. **Acronyms: all caps or all lower**
```go
// GOOD
httpServer, HTTPServer, xmlParser, XMLParser, userID, UserID

// BAD
HttpServer, XmlParser, UserId
```

### Package Structure Rules

1. **Names: short, lowercase, singular noun**
```go
// GOOD
package user
package http
package config

// BAD
package users        // no plural
package httpHelpers  // no camelCase
package common       // too vague
```

2. **No generic utility packages**
```go
// BAD
package util
package helpers
package common
package misc

// GOOD - move functions to where they're used or create specific packages
package stringutil  // if truly reusable
package timeconv    // specific purpose
```

3. **Avoid stutter**
```go
// BAD
package user
func (u *User) UserName() string  // user.UserName()

// GOOD
package user
func (u *User) Name() string      // user.Name()
```

4. **internal/ for private packages**
```go
myapp/
├── cmd/
├── internal/       // cannot be imported outside myapp
│   ├── auth/
│   └── cache/
└── pkg/            // public, stable API (optional)
```

### Anti-patterns
- `Get`/`Set` prefixes on methods
- `this`/`self` as receiver names
- Package names: `util`, `helpers`, `common`, `misc`, `base`
- Stuttering: `config.ConfigFile`, `user.UserService`
- Underscores in names (except test files)

---

## Testing

**The Iron Law:** Tests are documentation. They must be clear, deterministic, and independent.

### Rules

1. **Table-driven tests for multiple cases**
```go
func TestParse(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    Result
        wantErr bool
    }{
        {
            name:  "valid input",
            input: "42",
            want:  Result{Value: 42},
        },
        {
            name:    "empty input",
            input:   "",
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Parse(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("Parse() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            if got != tt.want {
                t.Errorf("Parse() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

2. **Use `t.Run` for subtests**
```go
// Enables: go test -run TestParse/valid_input
t.Run(tt.name, func(t *testing.T) { ... })
```

3. **`t.Helper()` for test helpers**
```go
func assertNoError(t *testing.T, err error) {
    t.Helper() // Points to caller's line on failure
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
}
```

4. **`t.Parallel()` when tests are independent**
```go
func TestFeature(t *testing.T) {
    t.Parallel() // Top-level

    tests := []struct{ ... }
    for _, tt := range tests {
        tt := tt // Capture for parallel (pre-Go 1.22)
        t.Run(tt.name, func(t *testing.T) {
            t.Parallel() // Subtest-level
            // ...
        })
    }
}
```

5. **`t.Cleanup()` for teardown**
```go
func TestWithTempFile(t *testing.T) {
    f, err := os.CreateTemp("", "test")
    if err != nil {
        t.Fatal(err)
    }
    t.Cleanup(func() { os.Remove(f.Name()) })
    // test continues...
}
```

6. **Avoid assertion libraries - use standard comparisons**
```go
// BAD - external dependency, hides what's tested
assert.Equal(t, expected, got)

// GOOD - explicit, clear failure messages
if got != want {
    t.Errorf("Calculate() = %v, want %v", got, want)
}

// For complex comparisons, use go-cmp
if diff := cmp.Diff(want, got); diff != "" {
    t.Errorf("mismatch (-want +got):\n%s", diff)
}
```

7. **Test file naming**
```go
user.go              // Implementation
user_test.go         // Tests (same package, access internals)
user_export_test.go  // Export helpers for external tests
```

8. **Benchmarks follow same patterns**
```go
func BenchmarkParse(b *testing.B) {
    input := "test input"
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        Parse(input)
    }
}
```

### Anti-patterns
- Tests without `t.Run` for multiple cases
- Missing `t.Helper()` in helper functions
- Tests depending on execution order
- `time.Sleep` in tests (use channels or `t.Deadline`)
- Ignoring errors in test setup
- Tests that pass when run alone but fail together (shared state)

---

## Red Flags

STOP and reconsider when you notice:

### Core Violations

| Red Flag | Violation |
|----------|-----------|
| `interface{}` or `any` in function signature | "interface{} says nothing" |
| Interface with 5+ methods | "The bigger the interface, the weaker the abstraction" |
| `panic()` outside of init or truly unrecoverable | "Don't panic" |
| `reflect` package import | "Reflection is never clear" |
| `unsafe` package import | "With the unsafe package there are no guarantees" |
| `_ = someFunc()` ignoring error | "Don't just check errors, handle them gracefully" |
| `return err` without wrapping | "Errors are values" - add context |
| `go func()` without shutdown path | "Before you launch a goroutine, know when it will stop" |
| `time.Sleep` for synchronization | Channels orchestrate |
| Package named `util`, `common`, `helpers` | Generic names hide purpose |
| `Get` prefix on methods | Go convention: noun for getters |
| `this` or `self` as receiver | Use 1-2 letter type abbreviation |

### Initialization & Scope

| Red Flag | Problem |
|----------|---------|
| `init()` with side effects (network, file I/O) | Init should only set up package state, not do work |
| Variable shadowing with `:=` | Accidentally creates new variable instead of assigning |
| `context.Background()` or `context.TODO()` in production | Should propagate context from caller |
| Global mutable state | Pass dependencies explicitly |

### Concurrency Bugs

| Red Flag | Problem |
|----------|---------|
| `sync.WaitGroup` passed by value | WaitGroup must be passed by pointer |
| `sync.Mutex` copied (embedded in struct passed by value) | Mutex must not be copied after first use |
| Single-case `select` statement | Just receive from channel directly |
| `select {}` without cancellation | Blocks forever with no exit path |
| Goroutines in `init()` | Uncontrolled lifecycle, hard to test |

### Functions & Returns

| Red Flag | Problem |
|----------|---------|
| Naked `return` in long functions | Unclear what's being returned |
| `defer` inside loops | Defers don't run until function exits; use closure |
| Returning `-1` or magic values for "not found" | Use `(value, bool)` or `(value, error)` |
| `log.Fatal` or `os.Exit` in library code | Kills caller's process; return error instead |

### Data & Memory

| Red Flag | Problem |
|----------|---------|
| Returning slice/map that aliases internal state | Caller can mutate your internals; copy first |
| Appending to slice parameter without copy | May mutate caller's underlying array |
| `strings.Contains` on `err.Error()` | Use `errors.Is` or `errors.As` |
| Empty `interface{}` as map key/value without reason | Loses type safety |

### Code Organization

| Red Flag | Problem |
|----------|---------|
| Circular dependencies between packages | Restructure; use interfaces at boundaries |
| `//nolint` without explanation | Must document why lint is being suppressed |
| Missing `default` in type switch (when appropriate) | May silently ignore new types |
| Exported function without doc comment | Public API must be documented |

---

## Anti-Pattern Catalog

### Error Handling Anti-Patterns
```go
// BAD: Ignoring error
data, _ := json.Marshal(v)

// BAD: Bare return
if err != nil {
    return err
}

// BAD: String matching
if err.Error() == "not found" { ... }

// BAD: Panic for recoverable error
if user == nil {
    panic("user not found")
}

// BAD: Swallowing error in goroutine
go func() {
    doWork() // error ignored
}()
```

### Concurrency Anti-Patterns
```go
// BAD: WaitGroup by value
func process(wg sync.WaitGroup) { // copied!
    defer wg.Done()
}

// BAD: Sleep for synchronization
go doWork()
time.Sleep(100 * time.Millisecond)
result := getResult()

// BAD: Defer in loop
for _, f := range files {
    file, _ := os.Open(f)
    defer file.Close() // won't close until function exits!
}

// BAD: Single-case select
select {
case msg := <-ch:
    process(msg)
}
// GOOD: just use
msg := <-ch
process(msg)

// BAD: Mutex copied
type Cache struct {
    mu sync.Mutex
    data map[string]string
}
func (c Cache) Get(k string) string { // c is copied, including mutex!
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.data[k]
}
```

### Interface Anti-Patterns
```go
// BAD: Interface defined at implementation
package user
type UserRepository interface { ... } // should be in consumer package
type userRepo struct { ... }

// BAD: Returning interface
func NewService() ServiceInterface { return &service{} }

// BAD: Pointer to interface
func Process(r *io.Reader) { ... }

// BAD: God interface
type DatabaseService interface {
    CreateUser(...)
    UpdateUser(...)
    DeleteUser(...)
    // ...20 more methods
}
```

### Naming Anti-Patterns
```go
// BAD: Stutter
package user
func NewUserService() *UserService { ... } // user.NewUserService, user.UserService

// BAD: Get prefix
func (u *User) GetName() string { ... }

// BAD: Generic package
package util
package helpers
package common

// BAD: this/self receiver
func (this *Server) Start() { ... }
func (self *Client) Connect() { ... }
```

### Testing Anti-Patterns
```go
// BAD: No subtests
func TestParse(t *testing.T) {
    // test case 1
    result1, _ := Parse("a")
    if result1 != expected1 { t.Error(...) }
    // test case 2
    result2, _ := Parse("b")
    if result2 != expected2 { t.Error(...) }
}

// BAD: Missing t.Helper
func checkError(t *testing.T, err error) {
    // missing t.Helper() - stack trace points here, not caller
    if err != nil {
        t.Fatal(err)
    }
}

// BAD: Sleep in tests
func TestAsync(t *testing.T) {
    go startWorker()
    time.Sleep(time.Second) // flaky!
    assertResult(t)
}
```

---

## Rationalizations

Don't believe these thoughts:

| Rationalization | Reality |
|-----------------|---------|
| "It's just a quick script" | Scripts become production code. Write it right. |
| "I'll add error handling later" | You won't. Handle errors now. |
| "This interface needs all these methods" | Split it. Callers rarely need everything. |
| "panic is fine here, it'll never happen" | It will happen. Return an error. |
| "reflect makes this so much cleaner" | Reflection hides complexity. Use code generation or concrete types. |
| "I need unsafe for performance" | Profile first. You probably don't. |
| "One global variable won't hurt" | It will. Pass dependencies explicitly. |
| "This goroutine will finish eventually" | "Eventually" means resource leak. Add context cancellation. |
| "The error message is obvious from context" | It's not. Wrap with what operation failed. |
| "Table-driven tests are overkill for this" | They're not. Your future self will add cases. |
| "I'll document it later" | Document now or forever hold your peace. |
| "This clever solution saves lines" | "Clear is better than clever." Write the obvious version. |

---

## Review Checklist

When reviewing code, actively hunt for:

1. Error handling gaps
2. Missing context in wrapped errors
3. Goroutines without lifecycle management
4. Oversized interfaces
5. Generic package names
6. Stuttering in names
7. Tests without subtests for multiple cases
8. `init()` functions doing too much
```

---

## Implementation Notes

### File Structure for Marketplace Submission

```
idiomatic-go/
├── SKILL.md           # Main skill content (above)
└── README.md          # Optional: additional examples or FAQ
```

### Token Efficiency

The skill is comprehensive (~600 lines) but organized with clear headers for quick scanning. Consider splitting into supporting files if specific sections grow significantly.

### Integration with Other Skills

- Works alongside `test-driven-development` skill
- Complements `systematic-debugging` skill
- Can reference `go-code-reviewer` agent for post-implementation review
