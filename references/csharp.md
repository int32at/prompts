# C# Code Review Reference

## Common Issues

### Security
- SQL injection via string concatenation
- Path traversal vulnerabilities
- XML External Entity (XXE) attacks
- Insecure deserialization
- Missing input validation
- Hardcoded credentials or connection strings
- Missing authentication/authorization attributes
- Weak cryptography (MD5, SHA1)
- Not using parameterized queries
- Cross-site scripting in MVC views

### Performance
- Missing async/await for I/O operations
- Using string concatenation in loops (not StringBuilder)
- Not disposing IDisposable objects
- Boxing/unboxing value types excessively
- LINQ queries not deferred properly
- N+1 database query problem
- Large object heap allocations
- Not using connection pooling
- Missing caching strategies

### Code Quality
- Catching generic Exception instead of specific types
- Methods longer than 50 lines
- Classes with too many responsibilities (SRP violation)
- Missing XML documentation for public APIs
- Improper use of static members
- Not following naming conventions (PascalCase, camelCase)
- Missing null checks (pre-C# 8)
- Not using nullable reference types (C# 8+)
- Magic numbers and strings

### Bugs
- Not disposing resources (memory leaks)
- Race conditions in multithreading
- Deadlocks from improper locking
- Incorrect null handling
- DateTime comparison issues (not using UTC)
- Collection modified during enumeration
- Off-by-one errors
- Integer overflow

### Best Practices
- Use async/await for I/O-bound operations
- Implement IDisposable with using statements
- Use nullable reference types (C# 8+)
- Use dependency injection
- Follow SOLID principles
- Use pattern matching (C# 7+)
- Use record types for immutable data (C# 9+)
- Use expression-bodied members
- Use var judiciously
- Handle exceptions appropriately

## Language-Specific Patterns

### Modern C# to Encourage
```csharp
// Good: Async/await
public async Task<User> GetUserAsync(int id)
{
    return await _repository.FindAsync(id);
}

// Good: Using statement
using var connection = new SqlConnection(connectionString);

// Good: Nullable reference types
public string? FindName(int id) // Explicit nullable

// Good: Pattern matching
return shape switch
{
    Circle c => c.Radius * c.Radius * Math.PI,
    Rectangle r => r.Width * r.Height,
    _ => 0
};

// Good: StringBuilder for loops
var sb = new StringBuilder();
foreach (var item in items)
{
    sb.AppendLine(item);
}

// Good: LINQ deferred execution
var query = items.Where(x => x.IsActive); // Not executed yet
```

### Anti-patterns to Flag
```csharp
// Bad: Catching generic Exception
try
{
    DoSomething();
}
catch (Exception ex) // Too broad
{
    // ...
}

// Bad: String concatenation in loop
string result = "";
foreach (var item in items)
{
    result += item; // Use StringBuilder
}

// Bad: Not disposing
var stream = File.OpenRead(path);
// Missing using or Dispose()

// Bad: Blocking async call
var result = GetDataAsync().Result; // Can cause deadlocks

// Bad: SQL injection
var query = $"SELECT * FROM Users WHERE Name = '{name}'";
```

## ASP.NET Core Specific

### Security Issues
- Missing [Authorize] attributes
- Not validating ModelState
- Missing CSRF protection
- Exposing detailed error messages
- Not sanitizing user inputs
- Missing HTTPS enforcement
- Weak CORS policy

### Best Practices
```csharp
// Good: Authorization
[Authorize(Roles = "Admin")]
public IActionResult AdminAction() { }

// Good: Model validation
if (!ModelState.IsValid)
{
    return BadRequest(ModelState);
}

// Good: Dependency injection
public class UserController : Controller
{
    private readonly IUserService _userService;
    
    public UserController(IUserService userService)
    {
        _userService = userService;
    }
}

// Good: Async actions
public async Task<IActionResult> GetUsers()
{
    var users = await _service.GetUsersAsync();
    return Ok(users);
}
```

## Entity Framework Core

### Common Issues
- N+1 query problem (missing Include/ThenInclude)
- Not using AsNoTracking for read-only queries
- Loading entire tables into memory
- Not using pagination
- Missing indexes on foreign keys
- Cartesian explosion from multiple Includes

### Patterns to Encourage
```csharp
// Good: Eager loading
var users = await _context.Users
    .Include(u => u.Orders)
    .ToListAsync();

// Good: No tracking for read-only
var users = await _context.Users
    .AsNoTracking()
    .ToListAsync();

// Good: Pagination
var page = await _context.Users
    .Skip((pageNum - 1) * pageSize)
    .Take(pageSize)
    .ToListAsync();

// Good: Parameterized queries
var user = await _context.Users
    .FirstOrDefaultAsync(u => u.Id == id);
```

## Dependency Injection

### Best Practices
- Register services with appropriate lifetime (Transient, Scoped, Singleton)
- Avoid service locator anti-pattern
- Use constructor injection
- Don't inject IServiceProvider directly
- Avoid circular dependencies

## Threading and Async

### Common Issues
```csharp
// Bad: Mixing sync and async
public void Process()
{
    var result = GetDataAsync().Result; // Deadlock risk
}

// Good: Async all the way
public async Task ProcessAsync()
{
    var result = await GetDataAsync();
}

// Bad: Async void (except event handlers)
public async void DoWork() // Don't do this

// Good: Async Task
public async Task DoWorkAsync()
```

## Memory Management

### IDisposable Pattern
```csharp
// Good: IDisposable implementation
public class ResourceHolder : IDisposable
{
    private bool _disposed;
    
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
    
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed && disposing)
        {
            // Dispose managed resources
        }
        _disposed = true;
    }
}
```
