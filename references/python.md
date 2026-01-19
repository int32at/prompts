# Python Code Review Reference

## Common Issues

### Security
- SQL injection via string concatenation instead of parameterized queries
- Command injection via os.system() or subprocess without input sanitization
- Path traversal via unchecked file paths
- Pickle deserialization of untrusted data
- Eval/exec with user input
- Hardcoded credentials or API keys
- Missing authentication/authorization checks
- Weak cryptography (MD5, SHA1 for passwords)

### Performance
- Inefficient loops (nested iterations over large datasets)
- Missing list comprehensions where applicable
- Repeated database queries (N+1 problem)
- Missing connection pooling
- Loading entire files into memory unnecessarily
- Missing pagination for large result sets
- Not using generators for large data processing

### Code Quality
- Overly broad exception catching (bare except)
- Missing type hints for function signatures
- Functions longer than 50 lines
- Deeply nested code (>3 levels)
- Magic numbers without constants
- Mutable default arguments
- Missing docstrings for public APIs
- Inconsistent naming conventions (PEP 8)
- Global variables instead of proper encapsulation

### Bugs
- Race conditions in multithreading
- Resource leaks (unclosed files, connections)
- Off-by-one errors in slicing
- Incorrect None comparisons (using == instead of is)
- Missing error handling for external calls
- Type mismatches (comparing str to int)
- Incorrect dictionary key checks

### Best Practices
- Use context managers (with statements) for resources
- Use pathlib instead of os.path
- Use dataclasses or attrs for data objects
- Use enumerate instead of range(len())
- Use collections.defaultdict/Counter where appropriate
- Use logging instead of print statements
- Use virtual environments
- Follow PEP 8 style guide
- Use f-strings instead of % or .format()
- Use list/dict/set comprehensions appropriately

## Language-Specific Patterns

### Pythonic Patterns to Encourage
```python
# Good: List comprehension
squares = [x**2 for x in range(10)]

# Good: Context manager
with open('file.txt') as f:
    content = f.read()

# Good: Enumerate
for i, item in enumerate(items):
    process(i, item)

# Good: Dictionary get with default
value = d.get('key', default_value)

# Good: Unpacking
x, y, *rest = values
```

### Anti-patterns to Flag
```python
# Bad: Not using context manager
f = open('file.txt')
content = f.read()
# Missing f.close()

# Bad: Manual indexing
for i in range(len(items)):
    item = items[i]

# Bad: Mutable default argument
def append_to(element, lst=[]):  # Bug!
    lst.append(element)
    return lst

# Bad: Bare except
try:
    risky_operation()
except:  # Too broad
    pass
```

## Frameworks

### Django
- Check for SQL injection in raw queries
- Verify CSRF protection is enabled
- Check for proper authentication decorators
- Ensure DEBUG=False in production
- Verify SECRET_KEY is not hardcoded

### Flask
- Check for SQL injection in raw SQL
- Verify CSRF protection
- Check session security configuration
- Ensure proper error handling
- Verify input validation

### FastAPI/Pydantic
- Check request validation with Pydantic models
- Verify dependency injection usage
- Check async/await consistency
- Ensure proper exception handlers
