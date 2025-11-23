# Go Interview Questions & Answers

This document contains 25 comprehensive interview questions with detailed answers for Go developers in DevOps context.

## 25 Interview Questions & Answers

### 1. What is the difference between `make` and `new` in Go?

**Answer:**
- `new(T)` allocates zeroed storage for a new item of type T and returns a pointer `*T` to it. It only allocates memory and zeros it.
- `make(T, args)` is used only for slices, maps, and channels. It creates an initialized (not zeroed) instance of type T (not `*T`) and returns T itself.

Example:
```go
// new returns a pointer to a zeroed value
p := new([]int)  // *[]int, p is a pointer to a nil slice

// make returns an initialized slice ready to use
s := make([]int, 10)  // []int with length and capacity of 10
m := make(map[string]int)  // initialized empty map
ch := make(chan int, 5)  // buffered channel with capacity 5
```

### 2. Explain how goroutines are scheduled and what the GMP model is.

**Answer:**
Go uses an M:N threading model called GMP:
- **G (Goroutine)**: A goroutine, the basic unit of concurrency
- **M (Machine)**: An OS thread managed by the Go runtime
- **P (Processor)**: A logical processor that represents resources needed to execute Go code

The runtime multiplexes goroutines (G) onto OS threads (M) through logical processors (P). Each P has a local run queue of goroutines. This allows many goroutines to run on fewer OS threads, making concurrency lightweight. The scheduler performs work-stealing between Ps to balance load and uses async system calls to avoid blocking Ms when goroutines perform I/O.

### 3. What are the key differences between buffered and unbuffered channels?

**Answer:**
- **Unbuffered channels** (`make(chan T)`) have no capacity. Send blocks until a receiver is ready, and receive blocks until a sender is ready. They provide synchronization between goroutines.
- **Buffered channels** (`make(chan T, capacity)`) have a capacity. Sends block only when the buffer is full; receives block only when the buffer is empty. They decouple sender and receiver timing.

Example:
```go
// Unbuffered - synchronous
ch1 := make(chan int)
go func() { ch1 <- 1 }()  // blocks until received
val := <-ch1

// Buffered - can send without immediate receiver
ch2 := make(chan int, 3)
ch2 <- 1  // doesn't block
ch2 <- 2  // doesn't block
ch2 <- 3  // doesn't block
ch2 <- 4  // would block - buffer full
```

### 4. How do you prevent goroutine leaks?

**Answer:**
Goroutine leaks occur when goroutines are started but never terminate. Prevention strategies:

1. **Use context for cancellation**: Pass `context.Context` to goroutines and check for cancellation
2. **Always ensure channels have receivers**: Sending on a channel with no receiver will block forever
3. **Close channels when done**: Signal completion to waiting goroutines
4. **Use timeouts**: Don't wait indefinitely
5. **Use `sync.WaitGroup`**: Track goroutine completion

Example:
```go
func worker(ctx context.Context, jobs <-chan int) {
    for {
        select {
        case job := <-jobs:
            // process job
        case <-ctx.Done():
            return  // prevent leak by respecting cancellation
        }
    }
}
```

### 5. Explain the difference between value receivers and pointer receivers for methods.

**Answer:**
- **Value receiver** `func (t Type) Method()`: Operates on a copy of the value. Changes don't affect the original. Can be called on both values and pointers.
- **Pointer receiver** `func (t *Type) Method()`: Operates on the original value. Changes affect the original. More efficient for large structs.

Guidelines:
- Use pointer receivers when you need to modify the receiver
- Use pointer receivers for large structs to avoid copying
- Be consistent: if one method has a pointer receiver, all should (for interface satisfaction)

```go
type Counter struct {
    count int
}

func (c Counter) GetValue() int {
    return c.count  // value receiver - read-only
}

func (c *Counter) Increment() {
    c.count++  // pointer receiver - modifies original
}
```

### 6. What is the zero value in Go, and why is it important?

**Answer:**
In Go, all variables are initialized to their "zero value" if not explicitly initialized:
- Numbers: `0`
- Booleans: `false`
- Strings: `""`
- Pointers, slices, maps, channels, functions, interfaces: `nil`

This is important because:
1. No undefined variables - safer code
2. Zero values are often useful (e.g., empty slice can be appended to)
3. Simplifies initialization logic
4. Makes structs immediately usable in many cases

```go
var i int        // 0
var s string     // ""
var b bool       // false
var slice []int  // nil, but can append to it
```

### 7. How do slices work internally, and what is their relationship to arrays?

**Answer:**
A slice is a descriptor containing:
- A pointer to an underlying array
- Length (number of elements in the slice)
- Capacity (number of elements in the underlying array from the slice's start)

Slices provide a dynamic, flexible view into arrays. Multiple slices can share the same underlying array. When capacity is exceeded during `append`, a new larger array is allocated, and elements are copied.

```go
arr := [5]int{1, 2, 3, 4, 5}
slice1 := arr[1:4]  // points to arr, len=3, cap=4
slice2 := arr[2:4]  // points to same arr, len=2, cap=3

// Modifying slice1 affects arr and potentially slice2
slice1[0] = 99  // arr[1] = 99, visible in original array
```

### 8. What is the purpose of the `defer` statement, and what are its pitfalls?

**Answer:**
`defer` schedules a function call to execute after the surrounding function returns. It's commonly used for cleanup (closing files, unlocking mutexes).

Pitfalls:
1. **Arguments are evaluated immediately**, not when deferred function runs
2. **Runs in LIFO order** (stack-based)
3. **Performance cost** - slight overhead for each defer
4. **Defer in loops** can accumulate many deferred calls

```go
// Pitfall: argument evaluated immediately
i := 0
defer fmt.Println(i)  // prints 0, not 1
i++

// Correct pattern for resource cleanup
file, err := os.Open("file.txt")
if err != nil {
    return err
}
defer file.Close()  // ensures cleanup

// Pitfall in loop
for _, file := range files {
    f, _ := os.Open(file)
    defer f.Close()  // accumulates, not closed until function exits
}
// Better: use explicit function call or close immediately
```

### 9. Explain Go's error handling philosophy and best practices.

**Answer:**
Go treats errors as values, not exceptions. Functions return error as the last return value.

Best practices:
1. **Check errors explicitly** - don't ignore them
2. **Add context when wrapping**: use `fmt.Errorf("context: %w", err)`
3. **Use sentinel errors** for expected conditions: `var ErrNotFound = errors.New("not found")`
4. **Use custom error types** for richer information
5. **Use `errors.Is`** for checking sentinel errors in wrapped chains
6. **Use `errors.As`** for extracting specific error types

```go
// Wrapping errors with context
if err := doSomething(); err != nil {
    return fmt.Errorf("failed to do something: %w", err)
}

// Checking for specific errors
if errors.Is(err, ErrNotFound) {
    // handle not found
}

// Extracting error type
var pathErr *os.PathError
if errors.As(err, &pathErr) {
    fmt.Println("Failed at path:", pathErr.Path)
}
```

### 10. What is the `select` statement used for, and how does it work?

**Answer:**
`select` allows a goroutine to wait on multiple channel operations simultaneously. It blocks until one case can proceed, then executes that case. If multiple cases are ready, one is chosen pseudo-randomly.

```go
select {
case msg := <-ch1:
    fmt.Println("Received from ch1:", msg)
case msg := <-ch2:
    fmt.Println("Received from ch2:", msg)
case ch3 <- value:
    fmt.Println("Sent to ch3")
case <-time.After(1 * time.Second):
    fmt.Println("Timeout")
default:
    fmt.Println("No channels ready")  // non-blocking select
}
```

Common patterns:
- Timeouts with `time.After`
- Non-blocking operations with `default`
- Cancellation with `context.Done()`
- Multiplexing multiple channels

### 11. What is the purpose of the `context` package?

**Answer:**
The `context` package provides a way to carry deadlines, cancellation signals, and request-scoped values across API boundaries and between goroutines.

Key uses:
1. **Cancellation propagation**: Cancel downstream operations when parent is canceled
2. **Timeout management**: Automatically cancel operations that take too long
3. **Request-scoped values**: Pass request-specific data (use sparingly)

```go
// Create context with timeout
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

// Pass to functions
result, err := fetchDataWithContext(ctx)

// In the function, respect cancellation
func fetchDataWithContext(ctx context.Context) (Data, error) {
    select {
    case <-ctx.Done():
        return Data{}, ctx.Err()  // context.DeadlineExceeded or context.Canceled
    case result := <-doWork():
        return result, nil
    }
}
```

### 12. How do you handle graceful shutdown in a Go application?

**Answer:**
Graceful shutdown involves:
1. **Catching OS signals** (SIGTERM, SIGINT)
2. **Stopping acceptance of new requests**
3. **Completing in-flight requests**
4. **Cleaning up resources**

```go
func main() {
    server := &http.Server{Addr: ":8080"}
    
    // Start server in goroutine
    go func() {
        if err := server.ListenAndServe(); err != http.ErrServerClosed {
            log.Fatal(err)
        }
    }()
    
    // Wait for interrupt signal
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    
    log.Println("Shutting down server...")
    
    // Graceful shutdown with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    if err := server.Shutdown(ctx); err != nil {
        log.Fatal("Server forced to shutdown:", err)
    }
    
    log.Println("Server exited")
}
```

### 13. What are interfaces in Go, and how do they differ from other languages?

**Answer:**
Interfaces in Go are implicitly satisfied - a type implements an interface by implementing its methods, without explicit declaration. This is "structural typing" vs "nominal typing."

Advantages:
1. **Decoupling**: No dependency between interface definition and implementation
2. **Testability**: Easy to create mocks
3. **Composition**: Small interfaces are composable

```go
// Define interface
type Reader interface {
    Read(p []byte) (n int, err error)
}

// Any type with this method signature satisfies Reader
type MyReader struct{}

func (m MyReader) Read(p []byte) (int, error) {
    // implementation
    return 0, nil
}

// No explicit "implements" declaration needed
var r Reader = MyReader{}  // works automatically
```

Best practice: "Accept interfaces, return concrete types"

### 14. Explain the difference between `sync.Mutex` and `sync.RWMutex`.

**Answer:**
- **`sync.Mutex`**: Mutual exclusion lock. Only one goroutine can hold the lock at a time, for both reads and writes.
- **`sync.RWMutex`**: Reader/Writer lock. Multiple readers can hold the lock simultaneously, but writers get exclusive access.

Use `RWMutex` when reads are frequent and writes are rare for better performance.

```go
var mu sync.RWMutex
var data map[string]int

// Multiple readers can execute concurrently
func getValue(key string) int {
    mu.RLock()
    defer mu.RUnlock()
    return data[key]
}

// Writer gets exclusive access
func setValue(key string, value int) {
    mu.Lock()
    defer mu.Unlock()
    data[key] = value
}
```

### 15. What is the empty interface `interface{}` (or `any` in Go 1.18+), and when should you use it?

**Answer:**
The empty interface `interface{}` (now `any`) can hold values of any type because every type implements zero methods.

Use cases:
- Generic data structures before Go 1.18 generics
- JSON unmarshaling to unknown structures
- Printf-like variadic functions

However, avoid overuse:
- Loss of type safety
- Requires type assertions/switches
- Go 1.18+ generics are often better

```go
func PrintAny(v any) {
    fmt.Println(v)  // accepts anything
}

// Type assertion
var i any = "hello"
s := i.(string)  // type assertion, panics if wrong type
s, ok := i.(string)  // safe type assertion

// Type switch
switch v := i.(type) {
case string:
    fmt.Println("string:", v)
case int:
    fmt.Println("int:", v)
default:
    fmt.Println("unknown type")
}
```

### 16. How do you write effective tests in Go?

**Answer:**
Best practices for Go testing:

1. **Table-driven tests**: Test multiple cases with one test function
2. **Use subtests**: `t.Run()` for better organization and isolation
3. **Test naming**: `Test<FunctionName>` convention
4. **Helper functions**: Use `t.Helper()` to mark helper functions
5. **Parallel tests**: Use `t.Parallel()` when safe
6. **Mocking**: Use interfaces for dependency injection

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive numbers", 2, 3, 5},
        {"negative numbers", -2, -3, -5},
        {"mixed", -2, 3, 1},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("Add(%d, %d) = %d; want %d", 
                    tt.a, tt.b, result, tt.expected)
            }
        })
    }
}
```

### 17. What is a race condition, and how do you detect and prevent them in Go?

**Answer:**
A race condition occurs when multiple goroutines access shared memory concurrently, and at least one access is a write, without proper synchronization.

**Detection**:
- Use Go's race detector: `go test -race` or `go run -race`

**Prevention**:
1. **Use mutexes**: Protect shared data with `sync.Mutex` or `sync.RWMutex`
2. **Use channels**: Communicate, don't share memory
3. **Avoid shared state**: Design goroutines to be independent
4. **Use `sync/atomic`**: For simple counters and flags

```go
// Race condition
var counter int
for i := 0; i < 10; i++ {
    go func() {
        counter++  // RACE!
    }()
}

// Fixed with mutex
var counter int
var mu sync.Mutex
for i := 0; i < 10; i++ {
    go func() {
        mu.Lock()
        counter++
        mu.Unlock()
    }()
}

// Fixed with atomic
var counter int64
for i := 0; i < 10; i++ {
    go func() {
        atomic.AddInt64(&counter, 1)
    }()
}
```

### 18. Explain Go modules and semantic import versioning.

**Answer:**
Go modules are Go's dependency management system (introduced in Go 1.11, default since 1.16).

Key concepts:
- **`go.mod`**: Declares module path and dependencies
- **`go.sum`**: Contains cryptographic hashes of dependencies
- **Semantic versioning**: v1.2.3 (major.minor.patch)
- **Minimal version selection**: Uses oldest allowed version

**Semantic import versioning**: Major versions v2+ must be included in import path.

```go
// go.mod
module github.com/myuser/myproject

go 1.21

require (
    github.com/pkg/errors v0.9.1
    github.com/lib/pq v1.10.9
)

// For v2+, import path includes version
import "github.com/myuser/mylib/v2"
```

Commands:
- `go mod init`: Initialize module
- `go mod tidy`: Add missing and remove unused dependencies
- `go mod vendor`: Copy dependencies to vendor/

### 19. What are common patterns for worker pools in Go?

**Answer:**
Worker pools limit concurrent operations and reuse goroutines for efficiency.

```go
func workerPool(jobs <-chan Job, results chan<- Result, numWorkers int) {
    var wg sync.WaitGroup
    
    // Start worker goroutines
    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            for job := range jobs {
                result := process(job)
                results <- result
            }
        }(i)
    }
    
    // Wait for all workers to finish
    wg.Wait()
    close(results)
}

func main() {
    jobs := make(chan Job, 100)
    results := make(chan Result, 100)
    
    // Start worker pool
    go workerPool(jobs, results, 5)
    
    // Send jobs
    for _, job := range allJobs {
        jobs <- job
    }
    close(jobs)
    
    // Collect results
    for result := range results {
        // handle result
    }
}
```

### 20. How does Go handle HTTP requests, and what is important for production use?

**Answer:**
Go's `net/http` package provides a production-ready HTTP server.

Production considerations:
1. **Timeouts**: Always set read, write, and idle timeouts
2. **Graceful shutdown**: Handle SIGTERM/SIGINT
3. **Middleware**: Use for logging, auth, recovery
4. **Context**: Propagate request context
5. **Error handling**: Don't expose internal errors to clients

```go
server := &http.Server{
    Addr:         ":8080",
    Handler:      router,
    ReadTimeout:  15 * time.Second,
    WriteTimeout: 15 * time.Second,
    IdleTimeout:  60 * time.Second,
}

http.HandleFunc("/api/data", func(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    
    // Use context for downstream calls
    data, err := fetchData(ctx)
    if err != nil {
        http.Error(w, "Internal Server Error", http.StatusInternalServerError)
        log.Printf("Error: %v", err)  // Log internally
        return
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(data)
})
```

### 21. What is `iota`, and how is it used?

**Answer:**
`iota` is a predeclared identifier that represents successive untyped integer constants within a `const` block. It resets to 0 whenever `const` appears and increments by 1 for each subsequent const.

```go
// Simple enumeration
const (
    Sunday = iota     // 0
    Monday            // 1
    Tuesday           // 2
    Wednesday         // 3
)

// Powers of 2 (bit flags)
const (
    Read = 1 << iota  // 1 << 0 = 1
    Write             // 1 << 1 = 2
    Execute           // 1 << 2 = 4
)

// Skip values
const (
    _ = iota          // skip 0
    KB = 1 << (10 * iota)  // 1024
    MB                     // 1048576
    GB                     // 1073741824
)
```

### 22. Explain struct tags and their common uses.

**Answer:**
Struct tags are metadata attached to struct fields using backticks. They're used by packages via reflection to control behavior.

Common uses:
- **JSON encoding/decoding**: Control field names, omit empty
- **Validation**: Validate field values
- **ORM mapping**: Map to database columns
- **Configuration**: Map to environment variables

```go
type User struct {
    ID        int    `json:"id" db:"user_id"`
    Name      string `json:"name" validate:"required,min=3"`
    Email     string `json:"email" validate:"required,email"`
    Password  string `json:"-"`  // never marshal to JSON
    Age       int    `json:"age,omitempty"`  // omit if zero value
}

// Access tags via reflection
field, _ := reflect.TypeOf(User{}).FieldByName("Name")
tag := field.Tag.Get("json")  // "name"
```

### 23. How do you build a minimal Docker image for a Go application?

**Answer:**
Use multi-stage builds to create small, secure images:

```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder

WORKDIR /app

# Copy go mod files
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Build binary
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o myapp .

# Final stage
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copy binary from builder
COPY --from=builder /app/myapp .

EXPOSE 8080

CMD ["./myapp"]
```

Benefits:
- Small image size (alpine base)
- No build tools in final image
- Static binary (`CGO_ENABLED=0`)
- Security (minimal attack surface)

### 24. What are some performance optimization techniques in Go?

**Answer:**
1. **Profiling first**: Use `pprof` to identify bottlenecks
   ```go
   import _ "net/http/pprof"
   go http.ListenAndServe(":6060", nil)
   ```

2. **Reduce allocations**:
   - Reuse objects with `sync.Pool`
   - Preallocate slices with capacity
   - Use pointers for large structs

3. **Concurrency patterns**:
   - Worker pools to limit goroutines
   - Buffered channels to reduce blocking

4. **Algorithm optimization**:
   - Use appropriate data structures
   - Avoid unnecessary work

5. **Benchmarking**:
   ```go
   func BenchmarkFunction(b *testing.B) {
       for i := 0; i < b.N; i++ {
           Function()
       }
   }
   ```

6. **Compiler optimizations**:
   - Inline small functions
   - Escape analysis awareness

### 25. How do you implement logging in a Go application for production?

**Answer:**
Production logging best practices:

1. **Structured logging**: Use JSON format for easy parsing
2. **Log levels**: DEBUG, INFO, WARN, ERROR
3. **Context**: Include request IDs, user IDs
4. **Performance**: Async logging, sampling for high-volume
5. **Libraries**: `zap`, `zerolog`, or `logrus`

```go
import "go.uber.org/zap"

// Initialize logger
logger, _ := zap.NewProduction()
defer logger.Sync()

// Structured logging
logger.Info("user login",
    zap.String("user_id", "123"),
    zap.String("ip", "192.168.1.1"),
    zap.Duration("latency", time.Millisecond*100),
)

// With context
logger.With(
    zap.String("request_id", requestID),
).Error("failed to process request",
    zap.Error(err),
)

// Log levels
logger.Debug("debug message")
logger.Info("info message")
logger.Warn("warning message")
logger.Error("error message")
```

Configuration considerations:
- Output to stdout (captured by container orchestration)
- JSON format for log aggregation systems
- Include timestamps
- Avoid logging sensitive data (passwords, tokens)

---

## Using This Guide

- Review all 25 questions and answers
- Practice explaining concepts out loud
- Focus on understanding, not memorization
- Relate concepts to your DevOps experience
- Be ready to discuss real-world applications

## Additional Resources

- [Go Documentation](https://go.dev/doc/)
- [Effective Go](https://go.dev/doc/effective_go)
- [Go Blog](https://go.dev/blog/)

---

**PDF Generation (Optional)**

Pandoc:
```bash
pandoc interview_go_devops.md -o interview_go_devops.pdf
```
Add TOC:
```bash
pandoc --toc --toc-depth=2 interview_go_devops.md -o interview_go_devops.pdf
```