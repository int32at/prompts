# Java Code Review Reference

## Common Issues

### Security
- SQL injection via string concatenation
- Path traversal vulnerabilities
- XML External Entity (XXE) attacks
- Insecure deserialization
- Missing input validation
- Hardcoded credentials
- Weak cryptography (MD5, SHA1)
- Missing authentication/authorization
- Command injection
- LDAP injection

### Performance
- Not using PreparedStatement (SQL injection + performance)
- Creating unnecessary objects in loops
- Using String concatenation instead of StringBuilder
- Not closing resources (memory leaks)
- Inefficient collections usage
- N+1 query problem
- Not using connection pooling
- Autoboxing overhead
- Missing caching strategies
- Synchronization bottlenecks

### Code Quality
- Catching generic Exception
- Methods longer than 50 lines
- Classes with too many responsibilities
- Missing JavaDoc for public APIs
- Not following naming conventions
- Magic numbers and strings
- Using raw types instead of generics
- Mutable static fields
- Not using Optional for nullable returns
- Deep inheritance hierarchies

### Bugs
- NullPointerException from null checks
- Resource leaks (not using try-with-resources)
- ConcurrentModificationException
- Race conditions in multithreading
- Incorrect equals/hashCode implementations
- Off-by-one errors
- Integer overflow
- Incorrect use of == for object comparison

### Best Practices
- Use try-with-resources for AutoCloseable
- Use Optional instead of null
- Follow SOLID principles
- Use dependency injection (Spring)
- Prefer immutability
- Use Streams API appropriately
- Use modern Java features (records, sealed classes)
- Use enums instead of constants
- Follow effective Java guidelines
- Use static analysis tools (SonarQube, SpotBugs)

## Language-Specific Patterns

### Modern Java to Encourage
```java
// Good: Try-with-resources
try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    return br.readLine();
} catch (IOException e) {
    throw new RuntimeException("Failed to read file", e);
}

// Good: Optional instead of null
public Optional<User> findUser(int id) {
    return Optional.ofNullable(users.get(id));
}

// Good: Streams API
List<String> names = users.stream()
    .filter(User::isActive)
    .map(User::getName)
    .collect(Collectors.toList());

// Good: StringBuilder in loops
StringBuilder sb = new StringBuilder();
for (String item : items) {
    sb.append(item);
}

// Good: Diamond operator
Map<String, List<Integer>> map = new HashMap<>();

// Good: Records (Java 14+)
public record User(String name, int age) {}

// Good: Switch expressions (Java 14+)
String result = switch (day) {
    case MONDAY, FRIDAY -> "Work";
    case SATURDAY, SUNDAY -> "Weekend";
    default -> "Midweek";
};
```

### Anti-patterns to Flag
```java
// Bad: Not using try-with-resources
BufferedReader br = new BufferedReader(new FileReader(path));
String line = br.readLine();
// Missing br.close()

// Bad: Catching generic Exception
try {
    doSomething();
} catch (Exception e) { // Too broad
    e.printStackTrace(); // Don't use printStackTrace
}

// Bad: String concatenation in loop
String result = "";
for (String item : items) {
    result += item; // Use StringBuilder
}

// Bad: Returning null
public User findUser(int id) {
    return null; // Use Optional
}

// Bad: Using == for strings
if (str == "test") { // Use .equals()

// Bad: Raw types
List list = new ArrayList(); // Use generics
```

## Spring Framework

### Common Issues
```java
// Bad: Field injection
@Autowired
private UserService userService; // Avoid

// Good: Constructor injection
private final UserService userService;

@Autowired
public UserController(UserService userService) {
    this.userService = userService;
}

// Good: Using @Transactional properly
@Transactional(readOnly = true)
public User getUser(Long id) {
    return userRepository.findById(id)
        .orElseThrow(() -> new UserNotFoundException(id));
}

// Good: Proper exception handling
@ExceptionHandler(UserNotFoundException.class)
@ResponseStatus(HttpStatus.NOT_FOUND)
public ErrorResponse handleUserNotFound(UserNotFoundException ex) {
    return new ErrorResponse(ex.getMessage());
}
```

### Security
```java
// Good: Using Spring Security
@PreAuthorize("hasRole('ADMIN')")
public void adminOperation() {
    // ...
}

// Good: CSRF protection enabled
// Good: Password encoding
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

## JPA/Hibernate

### Common Issues
```java
// Bad: N+1 query problem
List<Order> orders = orderRepository.findAll();
for (Order order : orders) {
    order.getItems().size(); // Triggers N queries
}

// Good: Fetch join
@Query("SELECT o FROM Order o JOIN FETCH o.items")
List<Order> findAllWithItems();

// Bad: Not using pagination
List<User> users = userRepository.findAll(); // Loads everything

// Good: Pagination
Page<User> users = userRepository.findAll(
    PageRequest.of(0, 20, Sort.by("name")));

// Good: Proper entity relationships
@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
@JoinColumn(name = "order_id")
private List<OrderItem> items = new ArrayList<>();
```

## Concurrency

### Thread Safety
```java
// Bad: Race condition
private int counter;

public void increment() {
    counter++; // Not thread-safe
}

// Good: Using AtomicInteger
private final AtomicInteger counter = new AtomicInteger();

public void increment() {
    counter.incrementAndGet();
}

// Good: Using synchronized
private int counter;

public synchronized void increment() {
    counter++;
}

// Good: Using ReentrantLock
private final Lock lock = new ReentrantLock();
private int counter;

public void increment() {
    lock.lock();
    try {
        counter++;
    } finally {
        lock.unlock();
    }
}

// Good: Using ConcurrentHashMap
private final Map<String, User> users = new ConcurrentHashMap<>();

// Bad: Double-checked locking (broken in old Java)
// Use AtomicReference or synchronized instead
```

## Collections

### Best Practices
```java
// Good: Initialize with capacity if known
List<String> items = new ArrayList<>(1000);

// Good: Use appropriate collection
Set<String> unique = new HashSet<>(); // For uniqueness
List<String> ordered = new ArrayList<>(); // For order
Map<String, User> lookup = new HashMap<>(); // For lookup

// Good: Immutable collections
List<String> immutable = List.of("a", "b", "c");
Set<String> immutableSet = Set.of("a", "b", "c");
Map<String, Integer> immutableMap = Map.of("a", 1, "b", 2);

// Good: Use Streams appropriately
List<String> filtered = items.stream()
    .filter(s -> s.length() > 5)
    .collect(Collectors.toList());

// Bad: Modifying collection during iteration
for (String item : items) {
    if (condition) {
        items.remove(item); // ConcurrentModificationException
    }
}

// Good: Use iterator or removeIf
items.removeIf(item -> condition);
```

## Exception Handling

### Best Practices
```java
// Good: Specific exceptions
public User getUser(Long id) throws UserNotFoundException {
    return userRepository.findById(id)
        .orElseThrow(() -> new UserNotFoundException(id));
}

// Good: Custom exceptions
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(Long id) {
        super("User not found: " + id);
    }
}

// Bad: Swallowing exceptions
try {
    doSomething();
} catch (Exception e) {
    // Empty catch - don't do this
}

// Bad: Using exceptions for flow control
try {
    return map.get(key);
} catch (NullPointerException e) {
    return defaultValue; // Use Optional instead
}

// Good: Proper logging
try {
    doSomething();
} catch (SpecificException e) {
    logger.error("Error doing something", e);
    throw new ApplicationException("Failed to do something", e);
}
```

## Testing

### Best Practices
```java
// Good: JUnit 5 tests
@Test
void shouldReturnUserWhenFound() {
    // Given
    User expected = new User("John", 30);
    when(repository.findById(1L)).thenReturn(Optional.of(expected));
    
    // When
    User actual = service.getUser(1L);
    
    // Then
    assertEquals(expected.getName(), actual.getName());
    verify(repository).findById(1L);
}

// Good: Parameterized tests
@ParameterizedTest
@ValueSource(strings = {"", "  ", "\t", "\n"})
void shouldRejectBlankInput(String input) {
    assertThrows(ValidationException.class, 
        () -> service.process(input));
}

// Good: Mocking with Mockito
@Mock
private UserRepository repository;

@InjectMocks
private UserService service;
```

## Immutability

### Patterns to Encourage
```java
// Good: Immutable class
public final class User {
    private final String name;
    private final int age;
    
    public User(String name, int age) {
        this.name = Objects.requireNonNull(name);
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
}

// Good: Defensive copying
public class Order {
    private final List<Item> items;
    
    public Order(List<Item> items) {
        this.items = new ArrayList<>(items); // Copy
    }
    
    public List<Item> getItems() {
        return new ArrayList<>(items); // Copy
    }
}

// Good: Using records (Java 14+)
public record Point(int x, int y) {}
```

## Equals and HashCode

### Correct Implementation
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    User user = (User) o;
    return age == user.age && 
           Objects.equals(name, user.name);
}

@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```
