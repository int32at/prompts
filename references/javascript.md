# JavaScript/TypeScript Code Review Reference

## Common Issues

### Security
- XSS via innerHTML or dangerouslySetInnerHTML
- SQL injection in dynamic queries
- Command injection in child_process
- Path traversal in file operations
- Missing input sanitization
- Eval or Function constructor with user input
- Insecure randomness (Math.random for security)
- Missing CORS configuration
- JWT without proper validation
- Prototype pollution vulnerabilities

### Performance
- Missing key prop in React lists
- Unnecessary re-renders in React
- Memory leaks (event listeners not cleaned up)
- Blocking the event loop with heavy computations
- Not using memoization (useMemo, useCallback)
- Inefficient array operations (nested loops)
- Missing pagination or virtualization for long lists
- Not using Web Workers for heavy tasks
- Excessive DOM manipulation

### Code Quality
- Missing null/undefined checks
- Implicit type coercion bugs
- Callback hell (missing async/await)
- Not using const/let (using var)
- Missing error boundaries in React
- Inconsistent naming conventions
- Functions longer than 50 lines
- Missing JSDoc or TypeScript types
- Unused imports or variables
- Console.log statements in production code

### TypeScript-Specific
- Using 'any' type excessively
- Missing return types for functions
- Not enabling strict mode
- Type assertions instead of proper typing
- Missing null checks despite strict null checks
- Ignoring TypeScript errors with @ts-ignore

### Bugs
- Race conditions in async operations
- Missing error handling in promises
- Incorrect this binding
- Closure scope issues
- Memory leaks (uncleaned intervals/subscriptions)
- Missing dependency arrays in hooks
- Stale closures in React hooks
- Incorrect equality checks (== vs ===)

### Best Practices
- Use async/await instead of callbacks
- Use const by default, let when needed, never var
- Use strict equality (===)
- Use optional chaining (?.)
- Use nullish coalescing (??)
- Destructure objects and arrays
- Use arrow functions appropriately
- Handle promise rejections
- Use ESLint and Prettier
- Use TypeScript strict mode

## Language-Specific Patterns

### Modern JavaScript to Encourage
```javascript
// Good: Async/await
async function fetchData() {
  try {
    const response = await fetch(url);
    return await response.json();
  } catch (error) {
    console.error(error);
  }
}

// Good: Optional chaining
const value = obj?.property?.nested;

// Good: Nullish coalescing
const result = value ?? defaultValue;

// Good: Destructuring
const { name, age } = person;

// Good: Array methods
const filtered = items.filter(x => x.active);
```

### Anti-patterns to Flag
```javascript
// Bad: Callback hell
getData(function(a) {
  getMoreData(a, function(b) {
    getEvenMore(b, function(c) {
      // ...
    });
  });
});

// Bad: Using var
var x = 10; // Use const or let

// Bad: Missing error handling
fetch(url).then(res => res.json()); // No .catch()

// Bad: innerHTML with user data
element.innerHTML = userInput; // XSS risk

// Bad: Loose equality
if (x == null) // Use ===
```

## React-Specific

### Common Issues
- Missing key props in lists
- Mutating state directly
- Not using useCallback/useMemo appropriately
- Missing cleanup in useEffect
- Prop drilling instead of context
- Too many re-renders
- Large component files (>300 lines)
- Missing error boundaries

### Patterns to Encourage
```javascript
// Good: Proper hooks usage
useEffect(() => {
  const subscription = subscribe();
  return () => subscription.unsubscribe();
}, [dependency]);

// Good: Memoization
const memoized = useMemo(() => expensive(a, b), [a, b]);

// Good: Key props
{items.map(item => <Item key={item.id} {...item} />)}
```

## Node.js-Specific

### Security Issues
- Missing rate limiting on APIs
- Not validating environment variables
- Exposing stack traces to clients
- Missing helmet middleware
- Not sanitizing user inputs
- Missing authentication middleware

### Performance
- Not using connection pooling
- Synchronous operations in request handlers
- Not using streams for large files
- Missing caching strategy
- Not using cluster mode

## TypeScript Best Practices

### Type Safety
```typescript
// Good: Explicit types
function process(data: User): Result {
  // ...
}

// Good: Union types instead of any
type Status = 'pending' | 'active' | 'completed';

// Good: Proper null handling
function find(id: string): User | null {
  // ...
}

// Bad: Using any
function process(data: any): any { // Avoid this
  // ...
}
```
