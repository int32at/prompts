# Go Code Review Reference

## Common Issues

### Security
- SQL injection via string concatenation
- Command injection in exec.Command
- Path traversal vulnerabilities
- Missing input validation
- Hardcoded credentials
- Insecure randomness (math/rand for security)
- Missing authentication checks
- Weak cryptography
- Race conditions in concurrent code

### Performance
- Not reusing HTTP clients (creating new client per request)
- Not using connection pooling for databases
- Allocating in hot paths unnecessarily
- Not using sync.Pool for temporary objects
- Missing buffered channels causing blocking
- Goroutine leaks (not properly terminating)
- Not using strings.Builder for concatenation
- Copying large structs by value

### Code Quality
- Not checking errors
- Ignoring context cancellation
- Missing defer for cleanup
- Goroutines without proper error handling
- Not using idiomatic Go patterns
- Too many levels of indentation
- Functions longer than 50 lines
- Missing documentation for exported functions
- Not following Go naming conventions

### Bugs
- Race conditions (run go build -race)
- Goroutine leaks
- Nil pointer dereferences
- Channel deadlocks
- Panic in goroutines not recovered
- Context not propagated
- Resources not cleaned up (missing defer)
- Improper mutex usage

### Best Practices
- Check every error explicitly
- Use defer for cleanup
- Use context for cancellation and timeouts
- Follow effective Go guidelines
- Use gofmt/goimports
- Run golangci-lint
- Avoid global variables
- Use interfaces for abstraction
- Prefer composition over inheritance
- Keep interfaces small

## Language-Specific Patterns

### Idiomatic Go to Encourage
```go
// Good: Error handling
result, err := doSomething()
if err != nil {
    return fmt.Errorf("doing something: %w", err)
}

// Good: Defer for cleanup
file, err := os.Open("file.txt")
if err != nil {
    return err
}
defer file.Close()

// Good: Context propagation
func Process(ctx context.Context) error {
    select {
    case <-ctx.Done():
        return ctx.Err()
    case result := <-ch:
        return process(result)
    }
}

// Good: Table-driven tests
tests := []struct {
    name string
    input int
    want int
}{
    {"case1", 1, 2},
    {"case2", 2, 4},
}
for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        got := double(tt.input)
        if got != tt.want {
            t.Errorf("got %d, want %d", got, tt.want)
        }
    })
}

// Good: Buffered channels when needed
ch := make(chan int, 100)
```

### Anti-patterns to Flag
```go
// Bad: Ignoring errors
result, _ := doSomething() // Don't ignore errors

// Bad: Not using defer
file, err := os.Open("file.txt")
// ... code ...
file.Close() // May not be reached if panic/return

// Good: Use defer
defer file.Close()

// Bad: Creating HTTP client per request
client := &http.Client{}
resp, err := client.Get(url) // Create once, reuse

// Bad: Panic for normal errors
if err != nil {
    panic(err) // Use panic only for truly exceptional cases
}

// Bad: Not checking context
func Process(ctx context.Context) {
    // Long operation without checking ctx.Done()
}

// Bad: Goroutine leak
go func() {
    // Infinite loop with no way to stop
    for {
        doWork()
    }
}()
```

## Concurrency

### Common Issues
```go
// Bad: Race condition
var counter int
for i := 0; i < 10; i++ {
    go func() {
        counter++ // Race!
    }()
}

// Good: Using mutex
var (
    counter int
    mu sync.Mutex
)
for i := 0; i < 10; i++ {
    go func() {
        mu.Lock()
        counter++
        mu.Unlock()
    }()
}

// Good: Using atomic
var counter int64
for i := 0; i < 10; i++ {
    go func() {
        atomic.AddInt64(&counter, 1)
    }()
}

// Bad: Not propagating context
func fetchData() ([]byte, error) {
    resp, err := http.Get(url) // No timeout/cancellation
    // ...
}

// Good: Using context
func fetchData(ctx context.Context) ([]byte, error) {
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }
    resp, err := client.Do(req)
    // ...
}
```

### Goroutine Management
```go
// Bad: Goroutine leak
func process() {
    ch := make(chan int)
    go func() {
        for v := range ch {
            process(v)
        }
    }() // No way to stop this goroutine
}

// Good: Proper shutdown
func process(ctx context.Context) {
    ch := make(chan int)
    done := make(chan struct{})
    
    go func() {
        defer close(done)
        for {
            select {
            case v := <-ch:
                process(v)
            case <-ctx.Done():
                return
            }
        }
    }()
    
    // Later: wait for shutdown
    <-done
}
```

## Error Handling

### Patterns to Encourage
```go
// Good: Wrap errors with context
if err != nil {
    return fmt.Errorf("reading config: %w", err)
}

// Good: Custom error types
type ValidationError struct {
    Field string
    Msg   string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("%s: %s", e.Field, e.Msg)
}

// Good: Checking error types
var validationErr *ValidationError
if errors.As(err, &validationErr) {
    // Handle validation error
}

// Good: Sentinel errors
var ErrNotFound = errors.New("not found")

if errors.Is(err, ErrNotFound) {
    // Handle not found
}
```

## HTTP Servers

### Best Practices
```go
// Good: Proper HTTP server setup
srv := &http.Server{
    Addr:         ":8080",
    Handler:      router,
    ReadTimeout:  15 * time.Second,
    WriteTimeout: 15 * time.Second,
    IdleTimeout:  60 * time.Second,
}

// Good: Graceful shutdown
go func() {
    if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
        log.Fatalf("listen: %s\n", err)
    }
}()

// Wait for interrupt signal
<-ctx.Done()

shutdownCtx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

if err := srv.Shutdown(shutdownCtx); err != nil {
    log.Fatal("Server forced to shutdown:", err)
}
```

## Database Access

### Common Issues
```go
// Bad: SQL injection
query := fmt.Sprintf("SELECT * FROM users WHERE name = '%s'", name)

// Good: Parameterized queries
query := "SELECT * FROM users WHERE name = ?"
rows, err := db.Query(query, name)

// Bad: Not closing rows
rows, err := db.Query(query)
// Missing rows.Close()

// Good: Defer close
rows, err := db.Query(query)
if err != nil {
    return err
}
defer rows.Close()

// Good: Check rows.Err()
for rows.Next() {
    // ...
}
if err := rows.Err(); err != nil {
    return err
}
```

## Testing

### Best Practices
```go
// Good: Table-driven tests
func TestAdd(t *testing.T) {
    tests := []struct {
        name string
        a, b int
        want int
    }{
        {"positive", 1, 2, 3},
        {"negative", -1, -2, -3},
        {"zero", 0, 0, 0},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := Add(tt.a, tt.b)
            if got != tt.want {
                t.Errorf("Add(%d, %d) = %d, want %d", 
                    tt.a, tt.b, got, tt.want)
            }
        })
    }
}

// Good: Using testify for assertions
assert.Equal(t, expected, actual)
require.NoError(t, err)
```

## Resource Management

### Cleanup Patterns
```go
// Good: Multiple defers in order
func process(path string) error {
    f, err := os.Open(path)
    if err != nil {
        return err
    }
    defer f.Close()
    
    r := bufio.NewReader(f)
    // Work with r
    
    return nil
}

// Good: Defer with error check
defer func() {
    if err := f.Close(); err != nil {
        log.Printf("error closing file: %v", err)
    }
}()
```
