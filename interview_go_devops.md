# Go Technical Interview Prep (DevOps Context)

## Core Go Concepts & Topics (Experienced Developer Focus)

### Language & Syntax
- Package structure (`main` vs library packages), module layout
- Go modules: semantic import versioning, vendoring, replace directives
- Visibility rules (exported identifiers), `init` functions
- Variables, constants, `iota`, short declaration nuances
- Control structures (`for` variations, `switch`, `select`)
- Functions, closures, variadic functions, `defer` semantics & pitfalls
- Error handling patterns: sentinel, wrapping, custom types, `errors.Is` / `errors.As`
- Pointers, value vs reference semantics, zero values
- Composite types: arrays, slices, maps, structs
- Slice internals (header: ptr, len, cap), capacity growth, reslicing pitfalls
- Memory allocation basics: stack vs heap, escape analysis
- Strings & runes: UTF-8, immutability, conversions
- Method sets; pointer vs value receivers
- Interfaces: implicit satisfaction, minimal design, dynamic type assertions
- Generics: type parameters, constraints (`any`, `comparable`), custom constraints & performance
- Concurrency: goroutines, channels, `select`, patterns (fan-in/out, worker pools, pipelines)
- Synchronization: `sync.Mutex`, `RWMutex`, `WaitGroup`, `Cond`, `Once`, `Map`, atomic operations
- Cancellation & timeouts: `context.Context` best practices
- Avoiding goroutine leaks & proper cleanup
- `sync/atomic` vs locks trade-offs
- Immutability patterns & safe publication
- `time` package, timers, tickers
- Panics & recover: appropriate vs inappropriate usage

### Toolchain & Build
- `go build`, `go run`, `go test`, `go vet`, `go fmt`, linters (e.g., golangci-lint)
- Testing (table-driven, golden files, examples, benchmarks)
- Profiling & tracing: `pprof`, CPU/memory/block profiles, `runtime/trace`
- Race detector (`go test -race`)
- Build constraints / tags: `//go:build`
- Cross-compilation: `GOOS`, `GOARCH`, CGO toggling
- Module proxy & checksum database, private modules
- Reproducible builds, `GODEBUG` flags

### Runtime & Performance
- GC basics (tri-color marking, STW reduction)
- Allocation reduction techniques, pooling (`sync.Pool`)
- Data structure selection for performance
- Avoiding false sharing, cache friendliness

### I/O & Networking
- `net/http` servers, middleware patterns
- Streaming responses, request context usage
- gRPC vs REST trade-offs
- TLS config, HTTP/2 considerations
- JSON encoding: streaming (`json.Decoder`), `omitempty`, custom marshalers
- Large file I/O, buffering (`bufio`)
- Using `net` with `context`

### Architecture & Design
- Dependency injection via constructors & interfaces
- Internal packages, module layout conventions
- Small interfaces, hexagonal/layered architecture
- Error boundaries & wrapping strategy
- Logging (structured, `log/slog`, levels)

### DevOps Integration
- Container-friendly services (static binary, multi-stage Docker builds)
- Graceful shutdown (signal handling + `Server.Shutdown`)
- Health/readiness/liveness endpoints
- Observability: metrics (Prometheus), tracing (OpenTelemetry), structured logs
- Configuration management (env vars, flags, config files)
- Secrets handling (env, vault integration)
- Backpressure & rate limiting (`x/time/rate`, bounded channels)
- Feature flags & rollout
- CI/CD caching modules, test & lint steps

### Security
- Input validation & sanitization
- Avoid unsafe concurrency patterns
- Timeouts & context enforcement
- Maintaining dependencies
- Avoid panic-based control flow
- Proper TLS/certificate management

---

## 25 Interview Questions With Strong Answers

### 1. What are Go modules and how do they differ from GOPATH-based dependency management?
Go modules provide per-project versioning with `go.mod` and `go.sum`, enabling reproducible builds and semantic import versioning (major v2+ requires `/vN` path suffix). Unlike GOPATH (global workspace), modules allow dependencies at explicit versions, optional vendoring via `go mod vendor`, local overrides via `replace`, and integration with the checksum database for integrity. This eliminates version conflicts and makes builds hermetic.

### 2. Explain how slices work internally and a common pitfall when reslicing.
A slice header contains: pointer to underlying array, length, and capacity. Reslicing creates a new header sharing the same array. Pitfalls: (1) Appending can mutate data seen by other references sharing the array. (2) Holding a small subslice of a very large array retains the whole array—copy needed portion to a new slice to allow GC. (3) Addresses of elements become invalid after append-induced reallocation.

### 3. When should you use a pointer receiver vs a value receiver on methods?
Use pointer receivers when mutating state, avoiding copies of large structs, or ensuring consistent method sets for interface satisfaction. Use value receivers for small, immutable structs where copying is cheap (e.g., `time.Time`). Mixed receivers can be confusing; if any method must be pointer-based, often all should for uniformity unless conceptual immutability is desired.

### 4. Describe Go’s interface satisfaction and why small interfaces are encouraged.
Interface implementation is implicit: any type with required methods satisfies the interface—no explicit declarations. Small interfaces (single or few methods) improve testability, reduce coupling, and encourage composition (e.g., `io.Reader`). They minimize accidental large surface areas and make mocks/adapters straightforward.

### 5. How do you handle and classify errors in Go effectively?
Use simple errors for local issues, sentinel errors for well-known states (`io.EOF`), wrapping (`fmt.Errorf("context: %w", err)`) for call stack context, and custom types when structured data is needed. Callers use `errors.Is` for sentinel checks and `errors.As` for type extraction. Log errors once at a boundary; avoid panics except for programmer invariants.

### 6. What is the purpose of the context package and common misuse patterns?
`context.Context` provides cancellation, deadlines, and lightweight request-scoped values. Misuses: placing large objects/global config in context, ignoring `ctx.Done()` in loops, using `context.Background()` deep inside instead of propagating, and stuffing unrelated metadata (context is not a generic map).

### 7. Explain the role of `select` with channels and a pattern for implementing timeouts.
`select` waits on multiple channel operations, enabling multiplexing, cancellation, and timeout handling. Timeout pattern:

```go
select {
case v := <-ch:
    handle(v)
case <-ctx.Done():
    return ctx.Err()
case <-time.After(timeout):
    return errors.New("timeout")
}
```

Prefer context deadlines (`context.WithTimeout`) to unify cancellation API.

### 8. How do you prevent goroutine leaks in server code?
Tie goroutine lifetimes to contexts or channel closures; always check `<-ctx.Done()>` within loops. Avoid blocked sends/receives with no consumer. Close outbound channels when producers finish. Use `pprof` goroutine profiles and `go test -race` to detect lingering concurrency issues.

### 9. Compare using a channel vs a mutex for concurrency control.
Mutexes protect shared state with minimal overhead—best for quick critical sections. Channels excel at conveying ownership or events. Using channels to serialize state access can be overcomplicated; choose channels for communication patterns (pipelines), mutexes for shared mutable state. Combine when signaling plus protected state is needed.

### 10. When and why would you use `sync.WaitGroup`?
To wait for a set of goroutines to complete. Increment the counter before launching each goroutine; each goroutine calls `Done`. Caller blocks on `Wait`. Not for passing results—use channels or error aggregation separately. Avoid calling `Add` concurrently after goroutines start calling `Done` (race risk).

### 11. What are generics in Go and a practical DevOps example?
Generics allow writing algorithms over any type constrained by an interface or union. Example: a retry helper that returns typed results:

```go
func Retry[T any](n int, fn func() (T, error)) (T, error) {
    var zero T
    for i := 0; i < n; i++ {
        v, err := fn()
        if err == nil {
            return v, nil
        }
    }
    return zero, fmt.Errorf("failed after %d attempts", n)
}
```

Useful for reusable utilities (sets, maps, retry/backoff). Keep constraints minimal for clarity and performance.

### 12. How do you structure and write table-driven tests?
Define a slice of cases with fields for name, input, expected output/error, and iterate with `t.Run`:

```go
cases := []struct {
    name string
    in   string
    want Output
    wantErr bool
} {
    {"valid", "abc", Output{Val: "abc"}, false},
    {"empty", "", Output{}, true},
}

for _, tc := range cases {
    t.Run(tc.name, func(t *testing.T) {
        got, err := Parse(tc.in)
        if (err != nil) != tc.wantErr {
            t.Fatalf("error mismatch")
        }
        if !reflect.DeepEqual(got, tc.want) {
            t.Fatalf("got %+v want %+v", got, tc.want)
        }
    })
}
```

Promotes clarity and easy expansion.

### 13. How do you benchmark code and interpret results?
Use `testing` benchmarks:

```go
func BenchmarkProcess(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Process(sample)
    }
}
```

Run: `go test -bench=. -benchmem`. Metrics: `ns/op` (speed), `B/op` and `allocs/op` (allocation pressure). Compare different implementations; use consistent environment; combine with profiling (`-cpuprofile`, `-memprofile`) to identify hotspots.

### 14. What is the race detector and when should it be used?
Run with `go test -race` or `go run -race`. It instruments access patterns to detect data races at runtime. Use in CI and locally for concurrent code. It increases resource usage; not suitable for production performance baselines. A clean run increases confidence in thread safety.

### 15. Explain build tags and a use case in a DevOps scenario.
Build tags (`//go:build linux && !cgo`) control file inclusion per platform or feature. Example: separate implementations for system metrics on Linux vs Windows, or enabling CGO-only code behind `cgo` tag while default build remains static. Tags support building minimal container images by excluding unused OS-specific code.

### 16. How do you create a minimal Docker image for a Go service?
Use a multi-stage build, disable CGO, strip symbols:

```Dockerfile
FROM golang:1.23 AS build
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -trimpath -ldflags="-s -w" -o server ./cmd/server

FROM gcr.io/distroless/static
COPY --from=build /app/server /server
USER nonroot:nonroot
ENTRYPOINT ["/server"]
```

Result: small attack surface, fast startup.

### 17. How to implement graceful shutdown in an HTTP server?
Listen for signals, call `Server.Shutdown` with timeout context:

```go
srv := &http.Server{Addr: ":8080", Handler: mux}
ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
def defer stop()

go func() {
    if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
        log.Fatalf("listen error: %v", err)
    }
}()

<-ctx.Done()
shutdownCtx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
def cancel()
if err := srv.Shutdown(shutdownCtx); err != nil {
    log.Printf("shutdown error: %v", err)
}
```

Ensures in-flight requests finish, new ones rejected.

### 18. How do you expose metrics from a Go service?
Use Prometheus client:

```go
var ops = prometheus.NewCounter(prometheus.CounterOpts{
    Name: "app_operations_total",
    Help: "Total operations processed.",
})
func init() { prometheus.MustRegister(ops) }
http.Handle("/metrics", promhttp.Handler())
```

Increment counters, use histograms for latency. Avoid high-cardinality labels (user IDs). Integrate with alerts (error rates, latency SLOs).

### 19. What strategies help reduce memory allocations in Go?
Reuse buffers, preallocate slice capacity, minimize `[]byte` ↔ `string` conversions, avoid reflection in tight loops, use pooling for ephemeral objects (`sync.Pool`), prevent retaining large backing arrays via small subslices, profile first (`pprof`) to target real allocation hotspots.

### 20. How does `sync.Pool` work and when is it appropriate?
`sync.Pool` stores temporary objects that may be discarded at any GC cycle. Appropriate for high-frequency, throwaway allocations (e.g., buffers during encoding). Not a cache—don’t store essential or long-lived resources. Measure performance; premature pooling can reduce clarity with minimal benefit.

### 21. How to implement a worker pool with backpressure?
Use a bounded buffered channel; producers block when full:

```go
jobs := make(chan Task, 100)
wg := sync.WaitGroup()

for i := 0; i < workers; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        for task := range jobs {
            process(task)
        }
    }()
}

for _, t := range inputTasks {
    jobs <- t // blocks when channel full -> backpressure
}
close(jobs)
wg.Wait()
```

Enhance with context cancellation, error channel, or rate limiter.

### 22. Explain `errors.Is` vs `errors.As`.
`errors.Is(err, target)` checks if any error in the unwrap chain matches a sentinel (via equality or custom `Is`). `errors.As(err, &targetType)` extracts the first error of a given concrete type. Use `Is` for identity checks, `As` for structured data (e.g., `*net.OpError`).

### 23. What are common causes of high GC pause or memory usage in Go services?
Excess short-lived allocations, retaining large backing arrays via small slices, unbounded caches/maps, goroutine leaks holding references, reflection-heavy serialization. Mitigation: profile (`alloc_space`, `inuse_space`), reuse buffers, implement eviction, ensure goroutines terminate, reduce reflection.

### 24. How do you secure an HTTP service in Go (basic considerations)?
Enable TLS (cert rotation), apply timeouts (`ReadHeaderTimeout`, `IdleTimeout`), sanitize inputs, avoid exposing internal stack traces, implement rate limiting, structured logging without secrets, run as non-root, use least privilege for config/secrets, keep dependencies updated, verify user input size bounds, add CSRF/auth layers where needed.

### 25. Discuss designing a minimal interface for a pluggable storage component.
Start from consumer needs—define smallest behavior set:

```go
type Store interface {
    Get(ctx context.Context, key string) ([]byte, error)
    Put(ctx context.Context, key string, value []byte) error
}
```

Use a sentinel `ErrNotFound`. Avoid premature expansion; add optional smaller extension interfaces (e.g. `Deleter`) later. Context permits cancellation for network/disk operations. Keeps implementations simple and promotes testability.

---

## PDF Generation Instructions
To regenerate PDF locally:
```bash
pandoc --toc --toc-depth=2 interview_go_devops.md -V geometry:margin=1in -V fontsize=11pt -o interview_go_devops.pdf
```
