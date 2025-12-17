# Java 8 Interview Questions and Answers - Complete Guide

## Table of Contents
1. [Lambda Expressions](#lambda-expressions)
2. [Functional Interfaces](#functional-interfaces)
3. [Stream API](#stream-api)
4. [Optional Class](#optional-class)
5. [Default and Static Methods in Interfaces](#default-and-static-methods-in-interfaces)
6. [Method References](#method-references)
7. [Date/Time API](#datetime-api)
8. [Collectors](#collectors)
9. [Parallel Streams](#parallel-streams)
10. [Memory and Performance](#memory-and-performance)
11. [Miscellaneous Technical Questions](#miscellaneous-technical-questions)

---

## Lambda Expressions

### Q1: What are Lambda Expressions in Java 8? Why were they introduced?

**Answer:**
Lambda expressions are anonymous functions that provide a clear and concise way to implement single-method interfaces (functional interfaces). They were introduced to:

1. **Enable Functional Programming**: Java 8 embraced functional programming paradigm
2. **Reduce Boilerplate Code**: Eliminates verbose anonymous class syntax
3. **Support Parallel Processing**: Works seamlessly with Stream API for parallel operations
4. **Improve Collection Libraries**: Makes operations on collections more readable

**Syntax:**
```java
(parameters) -> expression
// or
(parameters) -> { statements; }
```

**Example - Before Java 8 (Anonymous Class):**
```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello World");
    }
};
```

**Example - With Lambda Expression:**
```java
Runnable runnable = () -> System.out.println("Hello World");
```

---

### Q2: What is the difference between Lambda Expression and Anonymous Class?

**Answer:**

| Aspect | Lambda Expression | Anonymous Class |
|--------|------------------|-----------------|
| **Syntax** | Concise, single expression | Verbose, requires full class definition |
| **Scope of `this`** | Refers to enclosing class | Refers to anonymous class itself |
| **Compilation** | Compiled using `invokedynamic` bytecode | Creates a separate .class file |
| **State** | Stateless (captures effectively final variables) | Can have instance variables/state |
| **Interface Type** | Only functional interfaces | Any interface or abstract class |
| **Performance** | Generally better (no extra class loading) | Slight overhead due to class creation |

**Code Example demonstrating `this` difference:**
```java
public class ThisExample {
    private String name = "Outer";
    
    public void demonstrate() {
        // Anonymous class - 'this' refers to anonymous class
        Runnable anonymous = new Runnable() {
            private String name = "Anonymous";
            @Override
            public void run() {
                System.out.println(this.name); // Prints "Anonymous"
            }
        };
        
        // Lambda - 'this' refers to enclosing class
        Runnable lambda = () -> {
            System.out.println(this.name); // Prints "Outer"
        };
    }
}
```

---

### Q3: What is "effectively final" in the context of Lambda expressions?

**Answer:**
A variable is **effectively final** if it's not explicitly declared as `final` but its value is never changed after initialization. Lambda expressions can only access local variables that are final or effectively final.

**Why this restriction?**
- Lambda expressions can be executed later (deferred execution)
- The variable might be out of scope when lambda executes
- Java copies the value to the lambda, so it must be immutable

**Example:**
```java
public void process() {
    String message = "Hello"; // effectively final
    int counter = 0;          // NOT effectively final
    
    // This works
    Runnable r1 = () -> System.out.println(message);
    
    // This causes compilation error
    // Runnable r2 = () -> System.out.println(counter++);
    
    counter++; // This makes 'counter' not effectively final
}
```

**Workaround using AtomicInteger:**
```java
AtomicInteger counter = new AtomicInteger(0);
Runnable r = () -> System.out.println(counter.incrementAndGet()); // Works!
```

---

### Q4: What is a Closure in Java Lambda expressions?

**Answer:**
A **Closure** is a lambda expression that captures variables from its enclosing scope. The lambda "closes over" these variables, maintaining access to them even when executed outside their original scope.

```java
public Function<Integer, Integer> createAdder(int addend) {
    // 'addend' is captured by the lambda - this creates a closure
    return number -> number + addend;
}

// Usage
Function<Integer, Integer> add5 = createAdder(5);
Function<Integer, Integer> add10 = createAdder(10);

System.out.println(add5.apply(3));  // Output: 8
System.out.println(add10.apply(3)); // Output: 13
```

---

## Functional Interfaces

### Q5: What is a Functional Interface? How is it different from a regular interface?

**Answer:**
A **Functional Interface** is an interface that contains exactly ONE abstract method. It can have multiple default and static methods.

**Key Characteristics:**
- Exactly one abstract method (SAM - Single Abstract Method)
- Can be annotated with `@FunctionalInterface` (optional but recommended)
- Target type for lambda expressions and method references
- Default methods don't count toward the single abstract method rule

```java
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);  // Single abstract method
    
    default void printResult(int result) {  // Default method - allowed
        System.out.println("Result: " + result);
    }
    
    static Calculator getAdder() {  // Static method - allowed
        return (a, b) -> a + b;
    }
}
```

---

### Q6: List and explain the built-in functional interfaces in java.util.function package.

**Answer:**

| Interface | Method | Description | Example |
|-----------|--------|-------------|---------|
| `Predicate<T>` | `boolean test(T t)` | Tests a condition | `x -> x > 10` |
| `Function<T,R>` | `R apply(T t)` | Transforms input to output | `s -> s.length()` |
| `Consumer<T>` | `void accept(T t)` | Consumes input, no return | `x -> System.out.println(x)` |
| `Supplier<T>` | `T get()` | Supplies a value | `() -> new ArrayList<>()` |
| `BiPredicate<T,U>` | `boolean test(T t, U u)` | Tests two inputs | `(a,b) -> a.equals(b)` |
| `BiFunction<T,U,R>` | `R apply(T t, U u)` | Transforms two inputs | `(a,b) -> a + b` |
| `BiConsumer<T,U>` | `void accept(T t, U u)` | Consumes two inputs | `(k,v) -> map.put(k,v)` |
| `UnaryOperator<T>` | `T apply(T t)` | Transforms same type | `x -> x * 2` |
| `BinaryOperator<T>` | `T apply(T t1, T t2)` | Combines two same types | `(a,b) -> a + b` |

**Practical Examples:**
```java
// Predicate - filter even numbers
Predicate<Integer> isEven = n -> n % 2 == 0;
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
numbers.stream().filter(isEven).forEach(System.out::println); // 2, 4

// Function - convert to uppercase
Function<String, String> toUpper = String::toUpperCase;
System.out.println(toUpper.apply("hello")); // HELLO

// Consumer - print with prefix
Consumer<String> printer = s -> System.out.println("Value: " + s);
printer.accept("Test"); // Value: Test

// Supplier - generate random number
Supplier<Double> randomSupplier = Math::random;
System.out.println(randomSupplier.get()); // 0.xxx

// BiFunction - concatenate strings with separator
BiFunction<String, String, String> concat = (a, b) -> a + "-" + b;
System.out.println(concat.apply("Hello", "World")); // Hello-World
```

---

### Q7: What is the difference between Predicate and Function?

**Answer:**

| Predicate<T> | Function<T, R> |
|--------------|----------------|
| Returns `boolean` only | Returns any type R |
| Used for testing/filtering | Used for transformation/mapping |
| Method: `test(T t)` | Method: `apply(T t)` |
| Has `and()`, `or()`, `negate()` | Has `compose()`, `andThen()` |

**Example:**
```java
// Predicate - checks condition
Predicate<String> isEmpty = String::isEmpty;
System.out.println(isEmpty.test("")); // true

// Function - transforms data
Function<String, Integer> length = String::length;
System.out.println(length.apply("Hello")); // 5

// Combining Predicates
Predicate<String> isNotEmpty = isEmpty.negate();
Predicate<String> hasLengthGreaterThan5 = s -> s.length() > 5;
Predicate<String> combined = isNotEmpty.and(hasLengthGreaterThan5);

// Chaining Functions
Function<String, String> trim = String::trim;
Function<String, String> toUpper = String::toUpperCase;
Function<String, String> combined2 = trim.andThen(toUpper);
System.out.println(combined2.apply("  hello  ")); // HELLO
```

---

### Q8: Can a Functional Interface extend another interface?

**Answer:**
Yes, a functional interface can extend another interface, BUT the total count of abstract methods must remain exactly ONE.

```java
// Valid - inherits abstract method, still only one
@FunctionalInterface
interface ChildInterface extends Runnable {
    // No new abstract method added - inherits run()
    default void doSomething() {
        System.out.println("Default method");
    }
}

// Invalid - would have two abstract methods
// @FunctionalInterface
// interface Invalid extends Runnable, Callable<String> { }

// Valid - overriding doesn't add new abstract method
@FunctionalInterface
interface MyPredicate extends Predicate<String> {
    // Still only test() method - valid
}
```

---

## Stream API

### Q9: What is Stream API? How is it different from Collections?

**Answer:**
Stream API is a sequence of elements supporting sequential and parallel aggregate operations. It's designed for functional-style operations on collections of elements.

| Aspect | Stream | Collection |
|--------|--------|------------|
| **Purpose** | Processing data | Storing data |
| **Modification** | Cannot modify source | Can add/remove elements |
| **Iteration** | Internal iteration | External iteration |
| **Consumption** | Single use (one-time traversal) | Multiple traversals |
| **Laziness** | Lazy evaluation | Eager operations |
| **Infinite** | Can be infinite | Must be finite |
| **Parallelism** | Built-in parallel support | Manual implementation |

**Key Characteristics of Streams:**
1. **Not a data structure** - doesn't store elements
2. **Functional** - operations produce results without modifying source
3. **Lazy** - intermediate operations are not evaluated until terminal operation
4. **Possibly unbounded** - can process infinite data
5. **Consumable** - elements visited only once

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

// Stream approach - declarative
List<String> filtered = names.stream()
    .filter(name -> name.startsWith("A"))
    .map(String::toUpperCase)
    .collect(Collectors.toList());

// Collection approach - imperative
List<String> result = new ArrayList<>();
for (String name : names) {
    if (name.startsWith("A")) {
        result.add(name.toUpperCase());
    }
}
```

---

### Q10: Explain the difference between Intermediate and Terminal operations in Streams.

**Answer:**

**Intermediate Operations:**
- Return another Stream
- Lazy - not executed until terminal operation
- Can be chained
- Stateless or Stateful

| Operation | Type | Description |
|-----------|------|-------------|
| `filter()` | Stateless | Filters elements based on predicate |
| `map()` | Stateless | Transforms elements |
| `flatMap()` | Stateless | Flattens nested structures |
| `distinct()` | Stateful | Removes duplicates |
| `sorted()` | Stateful | Sorts elements |
| `peek()` | Stateless | Performs action, returns same stream |
| `limit()` | Short-circuiting | Limits number of elements |
| `skip()` | Stateful | Skips first n elements |

**Terminal Operations:**
- Produce a result or side-effect
- Trigger stream processing
- Stream cannot be reused after

| Operation | Description |
|-----------|-------------|
| `forEach()` | Iterates each element |
| `collect()` | Accumulates elements into collection |
| `reduce()` | Reduces to single value |
| `count()` | Returns count of elements |
| `findFirst()` | Returns first element |
| `findAny()` | Returns any element |
| `allMatch()` | Tests if all match predicate |
| `anyMatch()` | Tests if any match predicate |
| `noneMatch()` | Tests if none match predicate |
| `min() / max()` | Returns min/max element |
| `toArray()` | Converts to array |

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Demonstrating lazy evaluation
Stream<Integer> stream = numbers.stream()
    .filter(n -> {
        System.out.println("Filtering: " + n);
        return n % 2 == 0;
    })
    .map(n -> {
        System.out.println("Mapping: " + n);
        return n * 2;
    });

// Nothing printed yet - lazy!
System.out.println("Terminal operation starting...");

// Terminal operation triggers processing
List<Integer> result = stream.limit(2).collect(Collectors.toList());
// Output:
// Terminal operation starting...
// Filtering: 1
// Filtering: 2
// Mapping: 2
// Filtering: 3
// Filtering: 4
// Mapping: 4
```

---

### Q11: What is the difference between map() and flatMap()?

**Answer:**

| `map()` | `flatMap()` |
|---------|-------------|
| One-to-one transformation | One-to-many transformation |
| Returns `Stream<R>` | Flattens nested streams |
| Used for simple mapping | Used when mapping produces streams |
| `Function<T, R>` | `Function<T, Stream<R>>` |

**Example:**
```java
// map() - one-to-one
List<String> words = Arrays.asList("Hello", "World");
List<Integer> lengths = words.stream()
    .map(String::length)
    .collect(Collectors.toList()); // [5, 5]

// Problem with map() for characters
List<Stream<String>> mapped = words.stream()
    .map(word -> Arrays.stream(word.split("")))
    .collect(Collectors.toList()); // List of Streams - not what we want!

// flatMap() - flattens the streams
List<String> characters = words.stream()
    .flatMap(word -> Arrays.stream(word.split("")))
    .distinct()
    .collect(Collectors.toList()); // [H, e, l, o, W, r, d]

// Real-world example - Orders with Items
class Order {
    List<Item> items;
}

List<Order> orders = getOrders();

// Get all items from all orders
List<Item> allItems = orders.stream()
    .flatMap(order -> order.getItems().stream())
    .collect(Collectors.toList());
```

---

### Q12: Explain reduce() operation with examples.

**Answer:**
`reduce()` is a terminal operation that combines elements of a stream into a single result using an associative accumulation function.

**Three Variants:**

```java
// 1. reduce(BinaryOperator) - returns Optional
Optional<Integer> sum = numbers.stream()
    .reduce((a, b) -> a + b);

// 2. reduce(identity, BinaryOperator) - returns value
Integer sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);

// 3. reduce(identity, BiFunction, BinaryOperator) - for parallel
Integer result = numbers.parallelStream()
    .reduce(0,
            (subtotal, element) -> subtotal + element,
            (subtotal1, subtotal2) -> subtotal1 + subtotal2);
```

**Practical Examples:**
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Sum
int sum = numbers.stream().reduce(0, Integer::sum); // 15

// Product
int product = numbers.stream().reduce(1, (a, b) -> a * b); // 120

// Max
Optional<Integer> max = numbers.stream().reduce(Integer::max); // 5

// Concatenation
List<String> words = Arrays.asList("Hello", " ", "World");
String sentence = words.stream().reduce("", String::concat); // "Hello World"

// Complex reduction - sum of squares
int sumOfSquares = numbers.stream()
    .reduce(0, (acc, n) -> acc + n * n, Integer::sum); // 55
```

---

### Q13: What is Short-Circuiting in Streams?

**Answer:**
Short-circuiting operations don't need to process all elements to produce a result. They terminate processing as soon as a condition is met.

**Short-Circuiting Operations:**
- **Intermediate:** `limit()`, `skip()`
- **Terminal:** `findFirst()`, `findAny()`, `anyMatch()`, `allMatch()`, `noneMatch()`

```java
// Without short-circuiting - processes all elements
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

// With short-circuiting - stops at first match
Optional<String> firstA = names.stream()
    .peek(n -> System.out.println("Processing: " + n))
    .filter(n -> n.startsWith("A"))
    .findFirst();
// Output: Processing: Alice (stops here)

// Infinite stream with limit
Stream.iterate(0, n -> n + 1)
    .limit(5)  // Short-circuits infinite stream
    .forEach(System.out::println); // 0, 1, 2, 3, 4

// anyMatch short-circuits on first true
boolean hasEven = numbers.stream()
    .peek(n -> System.out.println("Checking: " + n))
    .anyMatch(n -> n % 2 == 0);
// Stops at first even number
```

---

### Q14: How to create Streams in Java 8?

**Answer:**
```java
// 1. From Collection
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream1 = list.stream();
Stream<String> parallelStream = list.parallelStream();

// 2. From Array
String[] array = {"a", "b", "c"};
Stream<String> stream2 = Arrays.stream(array);
Stream<String> stream3 = Stream.of(array);

// 3. Stream.of() for values
Stream<String> stream4 = Stream.of("a", "b", "c");

// 4. Stream.generate() - infinite stream
Stream<Double> randoms = Stream.generate(Math::random).limit(5);

// 5. Stream.iterate() - infinite stream with seed
Stream<Integer> integers = Stream.iterate(0, n -> n + 2).limit(5); // 0, 2, 4, 6, 8

// 6. From String (chars)
IntStream chars = "Hello".chars();

// 7. From Files
Stream<String> lines = Files.lines(Paths.get("file.txt"));

// 8. Empty Stream
Stream<String> empty = Stream.empty();

// 9. Builder Pattern
Stream<String> stream5 = Stream.<String>builder()
    .add("a")
    .add("b")
    .add("c")
    .build();

// 10. IntStream, LongStream, DoubleStream
IntStream intStream = IntStream.range(1, 5);      // 1, 2, 3, 4
IntStream intStream2 = IntStream.rangeClosed(1, 5); // 1, 2, 3, 4, 5
```

---

### Q15: What happens if you try to reuse a Stream?

**Answer:**
Streams are **single-use**. Once a terminal operation is performed, the stream is closed and cannot be reused. Attempting to reuse throws `IllegalStateException`.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3);
Stream<Integer> stream = numbers.stream();

// First use - works
stream.forEach(System.out::println);

// Second use - throws exception
// stream.forEach(System.out::println);
// IllegalStateException: stream has already been operated upon or closed

// Solution: Create new stream each time
Supplier<Stream<Integer>> streamSupplier = () -> numbers.stream();
streamSupplier.get().forEach(System.out::println);
streamSupplier.get().filter(n -> n > 1).forEach(System.out::println);
```

---

## Optional Class

### Q16: What is Optional? Why was it introduced?

**Answer:**
`Optional<T>` is a container object that may or may not contain a non-null value. It was introduced to:

1. **Avoid NullPointerException**: Provides safer way to handle null
2. **Express Intent**: Makes it clear when a value might be absent
3. **Functional API**: Provides methods for functional-style operations
4. **Better API Design**: Return type signals possibility of absence

**Important Note:** Optional is designed for **return types**, not for:
- Class fields
- Method parameters
- Collection elements

```java
// Before Optional
public String getCity(User user) {
    if (user != null) {
        Address address = user.getAddress();
        if (address != null) {
            return address.getCity();
        }
    }
    return "Unknown";
}

// With Optional
public String getCity(User user) {
    return Optional.ofNullable(user)
        .map(User::getAddress)
        .map(Address::getCity)
        .orElse("Unknown");
}
```

---

### Q17: Explain the different ways to create Optional.

**Answer:**
```java
// 1. Optional.of() - throws NullPointerException if null
Optional<String> opt1 = Optional.of("value");
// Optional.of(null); // NullPointerException!

// 2. Optional.ofNullable() - allows null, creates empty Optional
Optional<String> opt2 = Optional.ofNullable("value");
Optional<String> opt3 = Optional.ofNullable(null); // Empty Optional

// 3. Optional.empty() - creates empty Optional
Optional<String> opt4 = Optional.empty();
```

---

### Q18: What are the key methods of Optional class?

**Answer:**

| Method | Description | Returns |
|--------|-------------|---------|
| `isPresent()` | Checks if value exists | boolean |
| `isEmpty()` (Java 11) | Checks if empty | boolean |
| `get()` | Returns value (throws if empty) | T |
| `orElse(T other)` | Returns value or default | T |
| `orElseGet(Supplier)` | Returns value or lazy default | T |
| `orElseThrow()` | Returns value or throws | T |
| `orElseThrow(Supplier)` | Returns value or custom exception | T |
| `ifPresent(Consumer)` | Executes if present | void |
| `ifPresentOrElse()` (Java 9) | Executes either way | void |
| `map(Function)` | Transforms value | Optional<R> |
| `flatMap(Function)` | Transforms to Optional | Optional<R> |
| `filter(Predicate)` | Filters value | Optional<T> |
| `or(Supplier)` (Java 9) | Alternative Optional | Optional<T> |

**Examples:**
```java
Optional<String> opt = Optional.of("Hello");

// isPresent / get
if (opt.isPresent()) {
    System.out.println(opt.get());
}

// orElse - always evaluates default
String value1 = opt.orElse("Default");

// orElseGet - lazy evaluation
String value2 = opt.orElseGet(() -> computeDefault());

// orElseThrow
String value3 = opt.orElseThrow(() -> new RuntimeException("Not found"));

// ifPresent
opt.ifPresent(v -> System.out.println("Value: " + v));

// map
Optional<Integer> length = opt.map(String::length);

// filter
Optional<String> filtered = opt.filter(s -> s.length() > 3);

// flatMap - when mapping returns Optional
Optional<String> result = opt.flatMap(this::findById);

// Chaining
String finalValue = Optional.ofNullable(getValue())
    .filter(v -> v.length() > 0)
    .map(String::toUpperCase)
    .orElse("DEFAULT");
```

---

### Q19: What is the difference between orElse() and orElseGet()?

**Answer:**
The key difference is **evaluation timing**.

| `orElse(T other)` | `orElseGet(Supplier<T>)` |
|-------------------|--------------------------|
| Default value always evaluated | Default value lazily evaluated |
| Even if Optional has value | Only when Optional is empty |
| Use for constants/simple values | Use for expensive computations |

```java
public String getDefault() {
    System.out.println("Computing default...");
    return "Default";
}

Optional<String> present = Optional.of("Value");
Optional<String> empty = Optional.empty();

// orElse - getDefault() is ALWAYS called
String result1 = present.orElse(getDefault());
// Output: Computing default...
// result1 = "Value"

// orElseGet - getDefault() called ONLY if empty
String result2 = present.orElseGet(this::getDefault);
// No output - method not called
// result2 = "Value"

String result3 = empty.orElseGet(this::getDefault);
// Output: Computing default...
// result3 = "Default"
```

**Performance Impact Example:**
```java
// BAD - expensive operation always runs
return optionalUser.orElse(userRepository.findDefault());

// GOOD - expensive operation runs only when needed
return optionalUser.orElseGet(() -> userRepository.findDefault());
```

---

### Q20: What is the difference between map() and flatMap() in Optional?

**Answer:**

| `map()` | `flatMap()` |
|---------|-------------|
| Wraps result in Optional | Expects function to return Optional |
| `Optional<Optional<R>>` if function returns Optional | `Optional<R>` directly |
| Use for regular transformations | Use when transformation returns Optional |

```java
class User {
    Optional<Address> getAddress() { ... }
}

class Address {
    String getCity() { ... }
}

Optional<User> user = findUser();

// map() wraps result - leads to nested Optional
Optional<Optional<Address>> nested = user.map(User::getAddress);

// flatMap() flattens the result
Optional<Address> address = user.flatMap(User::getAddress);

// Chained example
String city = user
    .flatMap(User::getAddress)  // Returns Optional<Address>
    .map(Address::getCity)       // Returns Optional<String>
    .orElse("Unknown");
```

---

## Default and Static Methods in Interfaces

### Q21: What are Default Methods in interfaces? Why were they introduced?

**Answer:**
Default methods are methods in interfaces with a default implementation, declared using the `default` keyword.

**Reasons for Introduction:**
1. **Backward Compatibility**: Add new methods to interfaces without breaking existing implementations
2. **Interface Evolution**: Extend functionality of existing interfaces (like `Collection.stream()`)
3. **Multiple Inheritance of Behavior**: Achieve behavior inheritance from multiple interfaces
4. **Optional Implementation**: Provide sensible defaults that implementers can override

```java
public interface Vehicle {
    void start();
    void stop();
    
    // Default method
    default void honk() {
        System.out.println("Beep beep!");
    }
    
    default void printInfo() {
        System.out.println("This is a vehicle");
    }
}

public class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car starting...");
    }
    
    @Override
    public void stop() {
        System.out.println("Car stopping...");
    }
    
    // Can override default method
    @Override
    public void honk() {
        System.out.println("Car horn: Honk!");
    }
    // printInfo() inherited as-is
}
```

---

### Q22: What happens when a class implements two interfaces with the same default method?

**Answer:**
This creates a **Diamond Problem**. Java 8 has specific rules to resolve this:

**Rules:**
1. **Class wins over interface**: Class method takes precedence
2. **Sub-interface wins**: More specific interface wins
3. **Explicit resolution required**: Otherwise, compilation error

```java
interface A {
    default void hello() {
        System.out.println("Hello from A");
    }
}

interface B {
    default void hello() {
        System.out.println("Hello from B");
    }
}

// Compilation error without explicit override
class C implements A, B {
    // MUST override to resolve conflict
    @Override
    public void hello() {
        // Option 1: Provide own implementation
        System.out.println("Hello from C");
        
        // Option 2: Call specific interface method
        A.super.hello();  // Calls A's version
        // or
        B.super.hello();  // Calls B's version
    }
}

// If B extends A, B's method wins
interface B extends A {
    @Override
    default void hello() {
        System.out.println("Hello from B");
    }
}

class D implements B {
    // Uses B's hello() automatically
}
```

---

### Q23: What are Static Methods in interfaces?

**Answer:**
Static methods in interfaces belong to the interface itself, not to implementing classes. They cannot be overridden and must be called using the interface name.

```java
public interface MathOperations {
    static int add(int a, int b) {
        return a + b;
    }
    
    static int multiply(int a, int b) {
        return a * b;
    }
    
    static boolean isPositive(int number) {
        return number > 0;
    }
}

// Usage - must use interface name
int sum = MathOperations.add(5, 3);
int product = MathOperations.multiply(4, 2);

// Cannot call through implementing class
class Calculator implements MathOperations { }
// Calculator.add(5, 3);  // Compilation error!
```

**Use Cases:**
- Factory methods
- Utility methods
- Replacing utility classes (like `Collections`)

---

## Method References

### Q24: What are Method References? What are the different types?

**Answer:**
Method references are shorthand syntax for lambda expressions that call existing methods. They use the `::` operator.

**Four Types:**

| Type | Syntax | Lambda Equivalent |
|------|--------|-------------------|
| Static method | `ClassName::staticMethod` | `x -> ClassName.staticMethod(x)` |
| Instance method (specific object) | `instance::instanceMethod` | `x -> instance.instanceMethod(x)` |
| Instance method (arbitrary object) | `ClassName::instanceMethod` | `(x, y) -> x.instanceMethod(y)` |
| Constructor | `ClassName::new` | `x -> new ClassName(x)` |

**Examples:**
```java
// 1. Static Method Reference
Function<String, Integer> parseInt = Integer::parseInt;
// Equivalent: s -> Integer.parseInt(s)

// 2. Instance Method of Particular Object
String prefix = "Hello ";
Function<String, String> concatenator = prefix::concat;
// Equivalent: s -> prefix.concat(s)

// 3. Instance Method of Arbitrary Object
Function<String, String> toUpper = String::toUpperCase;
// Equivalent: s -> s.toUpperCase()

BiPredicate<String, String> equals = String::equals;
// Equivalent: (s1, s2) -> s1.equals(s2)

// 4. Constructor Reference
Supplier<ArrayList<String>> listSupplier = ArrayList::new;
// Equivalent: () -> new ArrayList<>()

Function<Integer, ArrayList<String>> listWithSize = ArrayList::new;
// Equivalent: size -> new ArrayList<>(size)

// Practical Usage
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// Method reference
names.forEach(System.out::println);
// Equivalent lambda
names.forEach(name -> System.out.println(name));

// Sorting with method reference
names.sort(String::compareToIgnoreCase);

// Stream operations
List<String> upperNames = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

---

### Q25: Can you use method reference for methods with multiple parameters?

**Answer:**
Yes, but the functional interface must match the method signature.

```java
// BiFunction for two parameters
BiFunction<Integer, Integer, Integer> max = Math::max;
System.out.println(max.apply(5, 3)); // 5

// Instance method reference with receiver as first param
BiFunction<String, String, String> concat = String::concat;
System.out.println(concat.apply("Hello ", "World")); // Hello World

// Comparator (two parameters, returns int)
Comparator<String> comparator = String::compareTo;
List<String> sorted = names.stream()
    .sorted(String::compareToIgnoreCase)
    .collect(Collectors.toList());

// Custom functional interface
@FunctionalInterface
interface TriFunction<A, B, C, R> {
    R apply(A a, B b, C c);
}

class Calculator {
    public static int addThree(int a, int b, int c) {
        return a + b + c;
    }
}

TriFunction<Integer, Integer, Integer, Integer> add = Calculator::addThree;
System.out.println(add.apply(1, 2, 3)); // 6
```

---

## Date/Time API

### Q26: Why was a new Date/Time API introduced in Java 8?

**Answer:**
The old `java.util.Date` and `java.util.Calendar` had several issues:

| Problem with Old API | Solution in Java 8 |
|---------------------|-------------------|
| Mutable (not thread-safe) | Immutable and thread-safe |
| Poor design (months start at 0) | Intuitive API |
| Date class handles time too | Separate classes for date, time |
| No timezone support in Date | Built-in timezone handling |
| Hard to use API | Fluent, readable API |
| Difficult date arithmetic | Easy date manipulation |

---

### Q27: Explain the main classes in java.time package.

**Answer:**

| Class | Description | Example |
|-------|-------------|---------|
| `LocalDate` | Date without time | 2024-01-15 |
| `LocalTime` | Time without date | 10:30:45 |
| `LocalDateTime` | Date and time | 2024-01-15T10:30:45 |
| `ZonedDateTime` | Date, time with timezone | 2024-01-15T10:30:45+05:30[Asia/Kolkata] |
| `Instant` | Timestamp (epoch) | 2024-01-15T05:00:45Z |
| `Duration` | Time-based amount | PT2H30M (2 hours 30 min) |
| `Period` | Date-based amount | P1Y2M3D (1 year 2 months 3 days) |
| `ZoneId` | Timezone identifier | Asia/Kolkata |
| `ZoneOffset` | Timezone offset | +05:30 |
| `DateTimeFormatter` | Formatting/parsing | dd-MM-yyyy |

**Examples:**
```java
// LocalDate
LocalDate today = LocalDate.now();
LocalDate specific = LocalDate.of(2024, Month.JANUARY, 15);
LocalDate parsed = LocalDate.parse("2024-01-15");

// LocalTime
LocalTime now = LocalTime.now();
LocalTime specific = LocalTime.of(10, 30, 45);

// LocalDateTime
LocalDateTime dateTime = LocalDateTime.now();
LocalDateTime combined = LocalDateTime.of(today, now);

// ZonedDateTime
ZonedDateTime zoned = ZonedDateTime.now(ZoneId.of("Asia/Kolkata"));

// Instant - UTC timestamp
Instant instant = Instant.now();
long epochMilli = instant.toEpochMilli();

// Duration (time-based)
Duration duration = Duration.between(time1, time2);
Duration twoHours = Duration.ofHours(2);

// Period (date-based)
Period period = Period.between(date1, date2);
Period oneMonth = Period.ofMonths(1);

// Date arithmetic
LocalDate tomorrow = today.plusDays(1);
LocalDate lastMonth = today.minusMonths(1);
LocalDate nextYear = today.plusYears(1);

// Comparison
boolean isAfter = date1.isAfter(date2);
boolean isBefore = date1.isBefore(date2);

// Formatting
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd-MM-yyyy HH:mm:ss");
String formatted = dateTime.format(formatter);
LocalDateTime parsedDt = LocalDateTime.parse("15-01-2024 10:30:45", formatter);
```

---

### Q28: What is the difference between Instant and LocalDateTime?

**Answer:**

| Instant | LocalDateTime |
|---------|---------------|
| Point on timeline (UTC) | Date and time without timezone |
| Machine-oriented | Human-oriented |
| Absolute time | Relative/local time |
| Same instant worldwide | Different meaning per timezone |
| Use for: timestamps, comparisons | Use for: birthdays, appointments |
| Epoch-based (nanoseconds since 1970) | Calendar-based representation |

```java
// Instant - same moment everywhere
Instant instant = Instant.now();
System.out.println(instant); // 2024-01-15T05:00:00Z (UTC)

// LocalDateTime - local representation
LocalDateTime local = LocalDateTime.now();
System.out.println(local); // 2024-01-15T10:30:00 (local time)

// Converting between them
LocalDateTime ldt = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());
Instant inst = ldt.toInstant(ZoneOffset.UTC);

// ZonedDateTime - bridges both
ZonedDateTime zdt = instant.atZone(ZoneId.of("Asia/Kolkata"));
```

---

## Collectors

### Q29: Explain the Collectors class and its important methods.

**Answer:**
`Collectors` is a utility class providing various reduction operations for use with `collect()`.

**Common Collectors:**

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David", "Alice");
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 1. toList(), toSet()
List<String> list = names.stream().collect(Collectors.toList());
Set<String> set = names.stream().collect(Collectors.toSet());

// 2. toMap()
Map<String, Integer> nameLength = names.stream()
    .distinct()
    .collect(Collectors.toMap(
        name -> name,           // key mapper
        String::length          // value mapper
    ));

// Handle duplicate keys
Map<String, Integer> map = names.stream()
    .collect(Collectors.toMap(
        name -> name,
        String::length,
        (existing, replacement) -> existing  // merge function
    ));

// 3. joining()
String joined = names.stream()
    .collect(Collectors.joining(", ", "[", "]")); // [Alice, Bob, Charlie, David, Alice]

// 4. counting()
long count = names.stream().collect(Collectors.counting());

// 5. summarizingInt/Long/Double()
IntSummaryStatistics stats = numbers.stream()
    .collect(Collectors.summarizingInt(Integer::intValue));
// count=5, sum=15, min=1, average=3.0, max=5

// 6. averagingInt/Long/Double()
double avg = numbers.stream()
    .collect(Collectors.averagingInt(Integer::intValue)); // 3.0

// 7. summingInt/Long/Double()
int sum = numbers.stream()
    .collect(Collectors.summingInt(Integer::intValue)); // 15

// 8. groupingBy()
Map<Integer, List<String>> byLength = names.stream()
    .collect(Collectors.groupingBy(String::length));
// {5=[Alice, David, Alice], 3=[Bob], 7=[Charlie]}

// Downstream collector
Map<Integer, Long> countByLength = names.stream()
    .collect(Collectors.groupingBy(String::length, Collectors.counting()));

// 9. partitioningBy()
Map<Boolean, List<Integer>> partitioned = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
// {false=[1, 3, 5], true=[2, 4]}

// 10. maxBy/minBy()
Optional<String> longest = names.stream()
    .collect(Collectors.maxBy(Comparator.comparingInt(String::length)));

// 11. mapping()
Map<Integer, Set<Character>> firstCharByLength = names.stream()
    .collect(Collectors.groupingBy(
        String::length,
        Collectors.mapping(
            s -> s.charAt(0),
            Collectors.toSet()
        )
    ));

// 12. collectingAndThen()
List<String> unmodifiable = names.stream()
    .collect(Collectors.collectingAndThen(
        Collectors.toList(),
        Collections::unmodifiableList
    ));

// 13. reducing()
Optional<Integer> product = numbers.stream()
    .collect(Collectors.reducing((a, b) -> a * b));
```

---

### Q30: What is the difference between groupingBy() and partitioningBy()?

**Answer:**

| `groupingBy()` | `partitioningBy()` |
|----------------|-------------------|
| Groups by any classifier | Groups by boolean predicate |
| Multiple groups possible | Exactly two groups (true/false) |
| Uses Function | Uses Predicate |
| Map with any key type | Map<Boolean, List> |

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

// partitioningBy - exactly two groups
Map<Boolean, List<Integer>> evenOdd = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
// {false=[1, 3, 5], true=[2, 4, 6]}

// groupingBy - multiple groups
Map<Integer, List<Integer>> byRemainder = numbers.stream()
    .collect(Collectors.groupingBy(n -> n % 3));
// {0=[3, 6], 1=[1, 4], 2=[2, 5]}

// partitioningBy with downstream collector
Map<Boolean, Long> countEvenOdd = numbers.stream()
    .collect(Collectors.partitioningBy(
        n -> n % 2 == 0,
        Collectors.counting()
    ));
// {false=3, true=3}
```

---

## Parallel Streams

### Q31: What are Parallel Streams? When should you use them?

**Answer:**
Parallel streams divide the source data into multiple chunks and process them in parallel using the ForkJoinPool.

**When to Use:**
✅ Large datasets (1000+ elements)
✅ Computationally expensive operations
✅ Independent element processing
✅ No shared mutable state
✅ Stateless operations

**When NOT to Use:**
❌ Small datasets (parallel overhead > benefit)
❌ I/O operations (blocking operations)
❌ Order-sensitive operations
❌ Shared mutable state
❌ Operations with side effects

```java
List<Integer> numbers = IntStream.rangeClosed(1, 1000000)
    .boxed()
    .collect(Collectors.toList());

// Sequential
long seqStart = System.currentTimeMillis();
long seqSum = numbers.stream()
    .map(n -> n * n)
    .reduce(0L, Long::sum);
System.out.println("Sequential: " + (System.currentTimeMillis() - seqStart) + "ms");

// Parallel
long parStart = System.currentTimeMillis();
long parSum = numbers.parallelStream()
    .map(n -> n * n)
    .reduce(0L, Long::sum);
System.out.println("Parallel: " + (System.currentTimeMillis() - parStart) + "ms");

// Creating parallel streams
Stream<Integer> parallel1 = numbers.parallelStream();
Stream<Integer> parallel2 = numbers.stream().parallel();

// Check if parallel
boolean isParallel = parallel1.isParallel();

// Convert parallel to sequential
Stream<Integer> sequential = parallel1.sequential();
```

---

### Q32: What are the pitfalls of Parallel Streams?

**Answer:**

**1. Shared Mutable State (Race Condition)**
```java
// BAD - Race condition
List<Integer> result = new ArrayList<>();
numbers.parallelStream()
    .forEach(result::add);  // ArrayList is not thread-safe!

// GOOD - Use collect
List<Integer> result = numbers.parallelStream()
    .collect(Collectors.toList());
```

**2. Order Not Preserved**
```java
// Order might not be preserved
numbers.parallelStream()
    .forEach(System.out::println);  // Unpredictable order

// Use forEachOrdered for ordering
numbers.parallelStream()
    .forEachOrdered(System.out::println);  // Maintains order but slower
```

**3. Wrong Data Structure**
```java
// LinkedList - poor parallel performance
LinkedList<Integer> linkedList = new LinkedList<>(numbers);
linkedList.parallelStream()...  // Bad - O(n) for splitting

// ArrayList - good parallel performance
ArrayList<Integer> arrayList = new ArrayList<>(numbers);
arrayList.parallelStream()...  // Good - O(1) for splitting
```

**4. Stateful Operations**
```java
// limit() and skip() are expensive in parallel
numbers.parallelStream()
    .limit(100)  // Forces ordering, reduces parallelism
    .forEach(System.out::println);
```

**5. I/O Operations**
```java
// BAD - I/O blocks threads
urls.parallelStream()
    .map(this::fetchUrl)  // Blocks ForkJoinPool threads
    .collect(Collectors.toList());

// GOOD - Use CompletableFuture with custom executor
ExecutorService executor = Executors.newFixedThreadPool(10);
List<CompletableFuture<String>> futures = urls.stream()
    .map(url -> CompletableFuture.supplyAsync(() -> fetchUrl(url), executor))
    .collect(Collectors.toList());
```

---

### Q33: What is ForkJoinPool and how does it relate to Parallel Streams?

**Answer:**
`ForkJoinPool` is the default thread pool used by parallel streams. It uses a work-stealing algorithm for efficient parallel execution.

**Default Pool:**
- Common pool shared by all parallel streams
- Size = Runtime.getRuntime().availableProcessors() - 1
- Plus the calling thread

```java
// Check default parallelism
int parallelism = ForkJoinPool.commonPool().getParallelism();

// System property to change (set before any parallel operation)
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "10");

// Custom ForkJoinPool for specific stream
ForkJoinPool customPool = new ForkJoinPool(4);
try {
    long sum = customPool.submit(() ->
        numbers.parallelStream()
            .map(n -> n * n)
            .reduce(0L, Long::sum)
    ).get();
} finally {
    customPool.shutdown();
}
```

---

## Memory and Performance

### Q34: What is the memory footprint of Lambda expressions?

**Answer:**
Lambda expressions are more memory-efficient than anonymous classes:

1. **No separate .class file**: Lambdas use `invokedynamic`
2. **Lazy instantiation**: Created at runtime when needed
3. **Instance reuse**: Stateless lambdas can be reused
4. **No instance fields**: Unlike anonymous classes

```java
// Anonymous class - creates new .class file
Runnable anon = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};

// Lambda - uses invokedynamic, more efficient
Runnable lambda = () -> System.out.println("Hello");

// Method reference - even more efficient
Runnable methodRef = System.out::println;
```

**Note:** Lambdas that capture variables create new instances, while stateless lambdas may be cached.

---

### Q35: How are Lambda expressions compiled and executed?

**Answer:**
Lambda expressions use `invokedynamic` bytecode instruction for efficient runtime creation.

**Compilation Process:**
1. **Desugaring**: Lambda body becomes private static method
2. **Bootstrap Method**: LambdaMetafactory creates implementation class
3. **Linking**: At first invocation, creates and caches implementation
4. **Subsequent calls**: Use cached implementation

**Benefits:**
- No compile-time generated classes
- Flexibility for JVM optimizations
- Better performance over time (JIT can optimize)

```java
// Original lambda
Function<String, Integer> f = s -> s.length();

// Desugared (conceptually)
private static Integer lambda$main$0(String s) {
    return s.length();
}
// invokedynamic instruction creates the implementation at runtime
```

---

### Q36: What are the best practices for using Streams efficiently?

**Answer:**

**1. Prefer Primitive Streams**
```java
// BAD - boxing overhead
int sum = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();

// GOOD - use primitive stream
int sum = IntStream.range(1, 1000).sum();
```

**2. Avoid Stateful Operations When Possible**
```java
// Stateless - better parallelization
stream.filter(x -> x > 10)

// Stateful - requires more coordination
stream.distinct().sorted()
```

**3. Use Short-Circuiting**
```java
// GOOD - stops at first match
Optional<String> result = names.stream()
    .filter(name -> name.startsWith("A"))
    .findFirst();
```

**4. Avoid Side Effects**
```java
// BAD - side effects
List<String> results = new ArrayList<>();
names.stream().forEach(results::add);

// GOOD - functional approach
List<String> results = names.stream()
    .collect(Collectors.toList());
```

**5. Choose Right Data Source**
```java
// ArrayList, arrays - good for parallel
// LinkedList - poor for parallel (bad split)
// HashSet - good for distinct operations
```

**6. Measure Before Parallelizing**
```java
// Always benchmark! Parallel isn't always faster
```

---

## Miscellaneous Technical Questions

### Q37: What is StringJoiner and how is it used?

**Answer:**
`StringJoiner` is a utility class for constructing sequences of characters with delimiters, prefix, and suffix.

```java
// Basic usage
StringJoiner joiner = new StringJoiner(", ");
joiner.add("Alice").add("Bob").add("Charlie");
System.out.println(joiner); // Alice, Bob, Charlie

// With prefix and suffix
StringJoiner joiner2 = new StringJoiner(", ", "[", "]");
joiner2.add("1").add("2").add("3");
System.out.println(joiner2); // [1, 2, 3]

// Empty value
StringJoiner joiner3 = new StringJoiner(", ", "[", "]");
joiner3.setEmptyValue("EMPTY");
System.out.println(joiner3); // EMPTY

// Merging joiners
joiner.merge(joiner2);

// With Streams (preferred)
String result = Stream.of("a", "b", "c")
    .collect(Collectors.joining(", ", "[", "]")); // [a, b, c]
```

---

### Q38: What are Nashorn and its significance in Java 8?

**Answer:**
Nashorn is a JavaScript engine introduced in Java 8, replacing the older Rhino engine. It allows execution of JavaScript code within Java applications.

**Key Features:**
- 2-10x faster than Rhino
- ECMAScript 5.1 compliant
- Seamless Java-JavaScript interoperability
- Command-line tool: `jjs`

**Note:** Nashorn was deprecated in Java 11 and removed in Java 15.

```java
import javax.script.*;

public class NashornExample {
    public static void main(String[] args) throws ScriptException {
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("nashorn");
        
        // Execute JavaScript
        engine.eval("print('Hello from JavaScript')");
        
        // Evaluate expression
        Object result = engine.eval("10 + 20");
        System.out.println("Result: " + result);
        
        // Call Java from JavaScript
        engine.eval("var ArrayList = Java.type('java.util.ArrayList');" +
                   "var list = new ArrayList();" +
                   "list.add('Hello');" +
                   "print(list.get(0));");
    }
}
```

---

### Q39: What is Base64 encoding support in Java 8?

**Answer:**
Java 8 added `java.util.Base64` class for Base64 encoding/decoding without external libraries.

```java
import java.util.Base64;

// Basic Encoding/Decoding
String original = "Hello World";

// Encode
String encoded = Base64.getEncoder().encodeToString(original.getBytes());
System.out.println("Encoded: " + encoded); // SGVsbG8gV29ybGQ=

// Decode
byte[] decodedBytes = Base64.getDecoder().decode(encoded);
String decoded = new String(decodedBytes);
System.out.println("Decoded: " + decoded); // Hello World

// URL Encoder (safe for URLs)
String urlEncoded = Base64.getUrlEncoder().encodeToString(original.getBytes());

// MIME Encoder (adds line breaks for large data)
String mimeEncoded = Base64.getMimeEncoder().encodeToString(original.getBytes());

// Without padding
String noPadding = Base64.getEncoder().withoutPadding().encodeToString(original.getBytes());
```

---

### Q40: What is the CompletableFuture and its relation to Java 8?

**Answer:**
`CompletableFuture` provides an enhanced Future with asynchronous programming capabilities, introduced in Java 8.

```java
import java.util.concurrent.*;

// Creating CompletableFuture
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // Async computation
    return "Hello";
});

// Non-blocking result handling
future.thenAccept(result -> System.out.println("Result: " + result));

// Chaining
CompletableFuture<String> chained = future
    .thenApply(s -> s + " World")
    .thenApply(String::toUpperCase);

// Combining futures
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");

CompletableFuture<String> combined = future1.thenCombine(future2, (s1, s2) -> s1 + " " + s2);

// Exception handling
CompletableFuture<String> safe = future
    .exceptionally(ex -> "Default Value");

// Wait for multiple futures
CompletableFuture<Void> allOf = CompletableFuture.allOf(future1, future2);
CompletableFuture<Object> anyOf = CompletableFuture.anyOf(future1, future2);

// Blocking get (use sparingly)
String result = future.get(); // Blocks until complete
String resultWithTimeout = future.get(1, TimeUnit.SECONDS);

// Non-blocking check
boolean isDone = future.isDone();
```

---

### Q41: What is the diamond problem and how does Java 8 handle it with default methods?

**Answer:**
The diamond problem occurs when a class inherits from multiple sources that have the same method signature. Java 8 introduced default methods in interfaces, creating potential for this problem.

**Resolution Rules:**
1. **Classes always win** over interface default methods
2. **More specific interface wins** (subinterface over superinterface)
3. **Explicit override required** if neither rule applies

```java
interface A {
    default void hello() { System.out.println("A"); }
}

interface B extends A {
    default void hello() { System.out.println("B"); }
}

interface C extends A {
    default void hello() { System.out.println("C"); }
}

// B is more specific than A
class D implements A, B { }
// D uses B's hello()

// Must resolve explicitly
class E implements B, C {
    @Override
    public void hello() {
        B.super.hello(); // Explicit choice
    }
}
```

---

### Q42: Explain the forEach() method in Iterable interface.

**Answer:**
Java 8 added `forEach()` as a default method in `Iterable` interface, allowing functional iteration over collections.

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// Traditional for-each
for (String name : names) {
    System.out.println(name);
}

// Java 8 forEach
names.forEach(name -> System.out.println(name));

// Method reference
names.forEach(System.out::println);

// Map forEach (key-value)
Map<String, Integer> map = new HashMap<>();
map.put("One", 1);
map.put("Two", 2);

map.forEach((key, value) -> 
    System.out.println(key + " = " + value));

// With index (using AtomicInteger or IntStream)
AtomicInteger index = new AtomicInteger(0);
names.forEach(name -> 
    System.out.println(index.getAndIncrement() + ": " + name));
```

**Difference from Stream.forEach():**
- `Iterable.forEach()` - guaranteed to be sequential
- `Stream.forEach()` - order not guaranteed for parallel streams

---

### Q43: What are Spliterators in Java 8?

**Answer:**
`Spliterator` (Splittable Iterator) is an iterator designed for parallel traversal and partitioning of source elements.

**Key Characteristics:**
- `tryAdvance()` - process single element
- `trySplit()` - partition for parallel processing
- `estimateSize()` - estimated remaining elements
- `characteristics()` - properties like ORDERED, SORTED, SIZED

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

Spliterator<String> spliterator = names.spliterator();

// Check characteristics
boolean isOrdered = spliterator.hasCharacteristics(Spliterator.ORDERED);
boolean isSized = spliterator.hasCharacteristics(Spliterator.SIZED);

// Estimated size
long size = spliterator.estimateSize();

// Split for parallel processing
Spliterator<String> secondHalf = spliterator.trySplit();

// Traverse elements
spliterator.forEachRemaining(System.out::println);

// Custom Spliterator for special data sources
class CustomSpliterator implements Spliterator<String> {
    // Implementation...
}
```

---

### Q44: What is the difference between Collection.stream() and Collection.parallelStream()?

**Answer:**

| stream() | parallelStream() |
|----------|------------------|
| Sequential processing | Parallel processing |
| Single-threaded | Multi-threaded (ForkJoinPool) |
| Predictable order | Order may vary |
| Less overhead | More overhead for small data |
| Always safe | Requires stateless operations |
| Good for I/O | Good for CPU-intensive tasks |

```java
List<Integer> numbers = IntStream.rangeClosed(1, 100)
    .boxed()
    .collect(Collectors.toList());

// Sequential - single thread
numbers.stream()
    .map(n -> {
        System.out.println(Thread.currentThread().getName());
        return n * 2;
    })
    .collect(Collectors.toList());
// Output: main (all operations)

// Parallel - multiple threads
numbers.parallelStream()
    .map(n -> {
        System.out.println(Thread.currentThread().getName());
        return n * 2;
    })
    .collect(Collectors.toList());
// Output: ForkJoinPool.commonPool-worker-X (various threads)
```

---

### Q45: How to convert between Legacy Date API and new Date/Time API?

**Answer:**
```java
import java.time.*;
import java.util.Date;
import java.util.Calendar;

// Date to Instant
Date date = new Date();
Instant instant = date.toInstant();

// Instant to Date
Date fromInstant = Date.from(instant);

// Date to LocalDateTime
LocalDateTime ldt = date.toInstant()
    .atZone(ZoneId.systemDefault())
    .toLocalDateTime();

// LocalDateTime to Date
Date fromLdt = Date.from(ldt.atZone(ZoneId.systemDefault()).toInstant());

// Calendar to ZonedDateTime
Calendar calendar = Calendar.getInstance();
ZonedDateTime zdt = calendar.toInstant()
    .atZone(calendar.getTimeZone().toZoneId());

// ZonedDateTime to Calendar
Calendar fromZdt = GregorianCalendar.from(zdt);

// java.sql.Date to LocalDate
java.sql.Date sqlDate = java.sql.Date.valueOf(LocalDate.now());
LocalDate localDate = sqlDate.toLocalDate();

// java.sql.Timestamp to LocalDateTime
java.sql.Timestamp timestamp = java.sql.Timestamp.valueOf(LocalDateTime.now());
LocalDateTime fromTimestamp = timestamp.toLocalDateTime();
```

---

### Q46: What are the improvements in Collection Framework in Java 8?

**Answer:**

**1. New Methods in Collection Interface:**
```java
// removeIf - remove elements matching predicate
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
list.removeIf(n -> n % 2 == 0); // Removes 2, 4

// replaceAll - replace each element
list.replaceAll(n -> n * 2);

// sort - sort with comparator
list.sort(Comparator.reverseOrder());
```

**2. New Methods in Map Interface:**
```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);

// getOrDefault
int value = map.getOrDefault("C", 0); // Returns 0

// putIfAbsent
map.putIfAbsent("C", 3); // Adds only if absent

// compute
map.compute("A", (k, v) -> v + 10); // A=11

// computeIfAbsent
map.computeIfAbsent("D", k -> k.length()); // D=1

// computeIfPresent
map.computeIfPresent("A", (k, v) -> v * 2);

// merge
map.merge("A", 5, Integer::sum); // Adds or merges

// forEach
map.forEach((k, v) -> System.out.println(k + ":" + v));

// replaceAll
map.replaceAll((k, v) -> v * 10);
```

---

### Q47: What is Type Inference improvement in Java 8?

**Answer:**
Java 8 improved type inference, especially for generics with lambda expressions.

```java
// Before Java 8 - explicit type needed
List<String> list = Collections.<String>emptyList();

// Java 8 - compiler infers type
List<String> list = Collections.emptyList();

// Lambda type inference
// Compiler infers parameter types from functional interface
Comparator<String> comparator = (s1, s2) -> s1.compareTo(s2);
// Instead of: (String s1, String s2) -> s1.compareTo(s2)

// Method reference type inference
Function<String, Integer> parser = Integer::parseInt;
// Compiler knows parameter is String, return is Integer

// Diamond operator with anonymous class (Java 9 improvement)
// Java 8:
Comparator<String> comp = new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
};
```

---

### Q48: What is the peek() method in Streams and when should it be used?

**Answer:**
`peek()` is an intermediate operation that performs an action on each element without modifying the stream.

**Primary Use:** Debugging stream pipelines

```java
List<String> names = Arrays.asList("alice", "bob", "charlie");

// Debugging with peek
List<String> result = names.stream()
    .peek(n -> System.out.println("Original: " + n))
    .filter(n -> n.length() > 3)
    .peek(n -> System.out.println("Filtered: " + n))
    .map(String::toUpperCase)
    .peek(n -> System.out.println("Mapped: " + n))
    .collect(Collectors.toList());

// Output:
// Original: alice
// Filtered: alice
// Mapped: ALICE
// Original: bob
// Original: charlie
// Filtered: charlie
// Mapped: CHARLIE
```

**Warning:** Don't use `peek()` for side effects in production code - it violates the functional programming principle of pure functions.

---

### Q49: What are the key annotations introduced/enhanced in Java 8?

**Answer:**

**1. @FunctionalInterface**
```java
@FunctionalInterface
public interface MyFunction<T, R> {
    R apply(T t);
}
```

**2. Repeatable Annotations**
```java
@Repeatable(Schedules.class)
@interface Schedule {
    String day();
}

@interface Schedules {
    Schedule[] value();
}

@Schedule(day = "Monday")
@Schedule(day = "Wednesday")
public class RepeatableExample { }
```

**3. Type Annotations (on any type use)**
```java
// On type parameters
List<@NonNull String> names;

// On type cast
String s = (@NonNull String) object;

// On extends/implements
class MyList implements @Readonly List<String> { }

// On throws
void method() throws @Critical Exception { }
```

---

### Q50: Explain the PermGen to Metaspace change in Java 8.

**Answer:**
Java 8 removed PermGen (Permanent Generation) and introduced Metaspace for storing class metadata.

| PermGen (Pre-Java 8) | Metaspace (Java 8+) |
|---------------------|---------------------|
| Fixed size (default ~64MB-85MB) | Auto-growing (uses native memory) |
| Part of Java heap | Native memory (outside heap) |
| Caused OutOfMemoryError: PermGen | Causes OutOfMemoryError: Metaspace |
| Needed tuning: -XX:MaxPermSize | Optional tuning: -XX:MaxMetaspaceSize |
| GC in PermGen was problematic | Better GC behavior |
| Limited by heap | Limited by native memory |

**JVM Options:**
```bash
# Old (Java 7)
-XX:PermSize=256m -XX:MaxPermSize=512m

# New (Java 8+)
-XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m
```

**Benefits:**
1. No more PermGen OutOfMemory errors (common in redeploys)
2. More efficient memory usage
3. Automatic tuning by default
4. Better for dynamic class loading (frameworks, scripting)

---

## Quick Reference Summary

### Core Java 8 Features Checklist
- [ ] Lambda Expressions
- [ ] Functional Interfaces
- [ ] Stream API
- [ ] Optional Class
- [ ] Default & Static Methods in Interfaces
- [ ] Method References
- [ ] New Date/Time API
- [ ] Collectors
- [ ] Parallel Streams
- [ ] Nashorn JavaScript Engine
- [ ] Base64 Encoding
- [ ] CompletableFuture
- [ ] StringJoiner
- [ ] Type Annotations
- [ ] Repeatable Annotations
- [ ] PermGen to Metaspace

### Most Important Functional Interfaces
1. `Predicate<T>` - boolean test(T)
2. `Function<T,R>` - R apply(T)
3. `Consumer<T>` - void accept(T)
4. `Supplier<T>` - T get()
5. `BiFunction<T,U,R>` - R apply(T, U)

### Key Stream Operations
- **Intermediate:** filter, map, flatMap, sorted, distinct, peek, limit, skip
- **Terminal:** collect, forEach, reduce, count, findFirst, findAny, anyMatch, allMatch, noneMatch

---

## Interview Tips

1. **Understand the WHY**: Know why features were introduced, not just how to use them
2. **Code Examples**: Be ready to write working code during interviews
3. **Trade-offs**: Understand when to use parallel vs sequential streams
4. **Best Practices**: Know the dos and don'ts (especially with parallel streams)
5. **Real-world Usage**: Connect features to practical use cases
6. **Performance**: Understand memory and performance implications

---

*Document Version: 1.0 | Last Updated: December 2024*

