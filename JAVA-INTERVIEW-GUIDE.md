<p align="center">
  <img src="https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white" alt="Java"/>
</p>

<h1 align="center">☕ Java Interview Guide</h1>

<p align="center">
  <a href="README.md">← Back to Main Guide</a>
</p>

---

## 📚 Section Overview

| # | Topic | Difficulty |
|---|-------|------------|
| 1 | [Core Java Fundamentals](#1-core-java-fundamentals) | ⭐⭐ |
| 2 | [Object-Oriented Programming](#2-object-oriented-programming-oop) | ⭐⭐ |
| 3 | [Exception Handling](#3-exception-handling) | ⭐⭐ |
| 4 | [Collections Framework](#4-collections-framework) | ⭐⭐⭐ |
| 5 | [Multithreading & Concurrency](#5-multithreading--concurrency) | ⭐⭐⭐⭐ |
| 6 | [Java 8+ Features](#6-java-8-features) | ⭐⭐⭐ |
| 7 | [Java 9-21 Features](#7-java-9-21-features) | ⭐⭐⭐ |
| 8 | [Generics](#8-generics) | ⭐⭐⭐ |
| 9 | [Java I/O & NIO](#9-java-io--nio) | ⭐⭐ |
| 10 | [Design Patterns](#10-design-patterns) | ⭐⭐⭐ |
| 11 | [SOLID Principles](#11-solid-principles) | ⭐⭐⭐ |

---

## 1. Core Java Fundamentals

### 🔷 1.1 JDK vs JRE vs JVM

```
┌─────────────────────────────────────────────────────────────────┐
│                          JDK                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                        JRE                                │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │                      JVM                            │  │  │
│  │  │  • Executes bytecode                                │  │  │
│  │  │  • Platform independence                            │  │  │
│  │  │  • Memory management                                │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  │  + Core Libraries (rt.jar, java.lang, java.util)          │  │
│  └───────────────────────────────────────────────────────────┘  │
│  + Development Tools (javac, javadoc, jar, debugger)            │
└─────────────────────────────────────────────────────────────────┘
```

| Component | Full Form | Purpose |
|-----------|-----------|---------|
| **JVM** | Java Virtual Machine | Executes bytecode, provides platform independence |
| **JRE** | Java Runtime Environment | JVM + Core Libraries (for running Java apps) |
| **JDK** | Java Development Kit | JRE + Development Tools (for developing Java apps) |

### 🔷 1.2 JVM Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                        JVM ARCHITECTURE                        │
├────────────────────────────────────────────────────────────────┤
│                    CLASS LOADER SUBSYSTEM                      │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────────────┐ │
│  │  Bootstrap   │→ │  Extension   │→ │  Application/System   │ │
│  │  ClassLoader │  │  ClassLoader │  │  ClassLoader          │ │
│  │  (rt.jar)    │  │  (ext/*.jar) │  │  (classpath)          │ │
│  └──────────────┘  └──────────────┘  └───────────────────────┘ │
├────────────────────────────────────────────────────────────────┤
│                    RUNTIME DATA AREAS                          │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌───────────┐ │
│  │  Method     │ │    Heap     │ │   Stack     │ │    PC     │ │
│  │  Area       │ │  (Objects)  │ │  (per       │ │  Register │ │
│  │  (Shared)   │ │  (Shared)   │ │  thread)    │ │           │ │
│  └─────────────┘ └─────────────┘ └─────────────┘ └───────────┘ │
├────────────────────────────────────────────────────────────────┤
│                     EXECUTION ENGINE                           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐   │
│  │ Interpreter │ │ JIT Compiler│ │   Garbage Collector     │   │
│  └─────────────┘ └─────────────┘ └─────────────────────────┘   │
└────────────────────────────────────────────────────────────────┘
```

### 🔷 1.3 Memory Management

#### Heap Structure

```
┌──────────────────────────────────────────────────────────────────┐
│                           HEAP MEMORY                            │
├────────────────────────────────┬─────────────────────────────────┤
│        YOUNG GENERATION        │        OLD GENERATION           │
├──────────┬─────────┬───────────┤                                 │
│   Eden   │   S0    │    S1     │        Tenured Space            │
│  Space   │ (From)  │   (To)    │                                 │
├──────────┴─────────┴───────────┴─────────────────────────────────┤
│                          METASPACE                               │
│              (Class metadata - Native Memory)                    │
└──────────────────────────────────────────────────────────────────┘
```

| Memory Area | Storage | Thread Safety |
|-------------|---------|---------------|
| **Eden Space** | Newly created objects | Shared |
| **Survivor (S0/S1)** | Objects surviving Minor GC | Shared |
| **Old Generation** | Long-lived objects | Shared |
| **Metaspace** | Class metadata (Java 8+) | Shared |
| **Stack** | Local variables, method calls | Per Thread |
| **PC Register** | Current instruction address | Per Thread |

### 🔷 1.4 Garbage Collection

#### GC Algorithms Comparison

| GC Type | Description | Use Case | Java Version |
|---------|-------------|----------|--------------|
| 🔹 **Serial GC** | Single-threaded, stop-the-world | Small apps, single CPU | All |
| 🔹 **Parallel GC** | Multi-threaded for throughput | Batch processing | Default until Java 8 |
| 🔹 **G1 GC** | Region-based, low pause times | General purpose | Default Java 9+ |
| 🔹 **ZGC** | Sub-millisecond pauses | Large heaps (TB scale) | Java 11+ |
| 🔹 **Shenandoah** | Low pause, concurrent | Low latency apps | Java 12+ |

#### GC Process Flow

```
NEW OBJECT → EDEN → (Minor GC) → SURVIVOR → (Age > threshold) → OLD GEN
                                    ↓
                              (Major GC/Full GC)
                                    ↓
                              GARBAGE COLLECTED
```

### 🔷 1.5 Pass by Value vs Reference

> ⚠️ **Important**: Java is ALWAYS pass-by-value!

```java
// Primitives - value is copied
void modifyPrimitive(int x) {
    x = 100;  // Does NOT affect original
}

// Objects - reference VALUE is copied
void modifyObject(StringBuilder sb) {
    sb.append(" World");  // DOES affect original (same object)
    sb = new StringBuilder("New");  // Does NOT affect original reference
}
```

---

## 2. Object-Oriented Programming (OOP)

### 🔷 2.1 Four Pillars of OOP

```
┌─────────────────────────────────────────────────────────────────┐
│                    FOUR PILLARS OF OOP                          │
├────────────────┬────────────────┬─────────────┬─────────────────┤
│  ENCAPSULATION │  INHERITANCE   │ POLYMORPHISM│   ABSTRACTION   │
│                │                │             │                 │
│  🔒 Data      |   📦 Code      │  🔄 Many    │  🎭 Hide       │
│  Hiding        │  Reuse         │  Forms      │  Complexity     │
│                │                │             │                 │
│  private +     │  extends,      │  Overload,  │  abstract,      │
│  getters/      │  implements    │  Override   │  interface      │
│  setters       │                │             │                 │
└────────────────┴────────────────┴─────────────┴─────────────────┘
```

#### Encapsulation

```java
public class BankAccount {
    private double balance;  // 🔒 Hidden data
    
    public void deposit(double amount) {  // ✅ Controlled access
        if (amount > 0) {
            balance += amount;
        }
    }
    
    public double getBalance() {
        return balance;
    }
}
```

#### Inheritance

```java
public class Animal {
    protected String name;
    public void eat() { 
        System.out.println("Eating..."); 
    }
}

public class Dog extends Animal {
    @Override
    public void eat() { 
        System.out.println("Dog eating..."); 
    }
    
    public void bark() { 
        System.out.println("Barking..."); 
    }
}
```

#### Polymorphism

```java
// 📌 Compile-time (Method Overloading)
public int add(int a, int b) { return a + b; }
public double add(double a, double b) { return a + b; }
public int add(int a, int b, int c) { return a + b + c; }

// 📌 Runtime (Method Overriding)
Animal animal = new Dog();  // Reference: Animal, Object: Dog
animal.eat();  // Calls Dog's eat() method at runtime
```

#### Abstraction

```java
// Abstract Class - partial abstraction
public abstract class Shape {
    abstract double calculateArea();  // No implementation
    
    public void display() {  // Can have concrete methods
        System.out.println("Area: " + calculateArea());
    }
}

// Interface - full abstraction (before Java 8)
public interface Drawable {
    void draw();
    default void print() {  // Java 8+
        System.out.println("Printing..."); 
    }
}
```

### 🔷 2.2 Abstract Class vs Interface

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| **Methods** | Abstract + Concrete | Abstract (+ default/static Java 8+) |
| **Variables** | Any type | `public static final` only |
| **Inheritance** | Single | Multiple |
| **Constructor** | ✅ Yes | ❌ No |
| **Access Modifiers** | Any | `public` (implicit) |
| **When to Use** | IS-A with shared code | CAN-DO capability |

### 🔷 2.3 Method Overloading vs Overriding

| Aspect | Overloading | Overriding |
|--------|-------------|------------|
| **Class** | Same class | Parent-Child |
| **Method Signature** | Different parameters | Same |
| **Return Type** | Can be different | Same or covariant |
| **Access Modifier** | Any | Same or more permissive |
| **Binding** | Compile-time (Static) | Runtime (Dynamic) |
| **Exceptions** | Any | Same or narrower |

### 🔷 2.4 Object Cloning

```java
// Shallow Copy - copies references
public class Address implements Cloneable {
    String city;
    
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();  // Shallow copy
    }
}

// Deep Copy - creates new objects
public class Person implements Cloneable {
    String name;
    Address address;
    
    @Override
    protected Object clone() throws CloneNotSupportedException {
        Person cloned = (Person) super.clone();
        cloned.address = (Address) address.clone();  // Deep copy
        return cloned;
    }
}
```

---

## 3. Exception Handling

### 🔷 3.1 Exception Hierarchy

```
                        Throwable
                            │
            ┌───────────────┴───────────────┐
            │                               │
         Error                          Exception
      (Unchecked)                           │
            │                   ┌───────────┴───────────┐
    ┌───────┴───────┐           │                       │
 OutOfMemory  StackOverflow  RuntimeException     Checked Exceptions
    Error        Error        (Unchecked)              │
                                  │           ┌────────┴────────┐
                          ┌───────┴───────┐  IOException  SQLException
                   NullPointer  ArrayIndex
                   Exception   OutOfBounds
```

### 🔷 3.2 Checked vs Unchecked Exceptions

| Type | Checked | Unchecked |
|------|---------|-----------|
| **Compile Check** | ✅ Must handle | ❌ No requirement |
| **Extends** | Exception | RuntimeException |
| **Examples** | IOException, SQLException | NullPointerException, ArrayIndexOutOfBounds |
| **When** | Recoverable situations | Programming errors |

### 🔷 3.3 Exception Handling Syntax

```java
// try-catch-finally
try {
    // Risky code
    int result = 10 / 0;
} catch (ArithmeticException e) {
    // Handle specific exception
    System.out.println("Cannot divide by zero");
} catch (Exception e) {
    // Handle general exception
    System.out.println("Error: " + e.getMessage());
} finally {
    // Always executes (cleanup)
    System.out.println("Cleanup");
}

// try-with-resources (Java 7+)
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line = br.readLine();
} catch (IOException e) {
    e.printStackTrace();
}  // br.close() called automatically
```

### 🔷 3.4 throw vs throws

```java
// throw - to explicitly throw an exception
public void validateAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("Age cannot be negative");
    }
}

// throws - declare method can throw exception
public void readFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    // ...
}
```

### 🔷 3.5 Custom Exceptions

```java
public class BusinessException extends Exception {
    private String errorCode;
    
    public BusinessException(String message, String errorCode) {
        super(message);
        this.errorCode = errorCode;
    }
    
    public String getErrorCode() {
        return errorCode;
    }
}

// Usage
throw new BusinessException("User not found", "ERR_001");
```

---

## 4. Collections Framework

### 🔷 4.1 Collection Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  LEGEND:  🔷 Interface    🔶 Class                                          │
└─────────────────────────────────────────────────────────────────────────────┘

🔷 Iterable
        │
        └── 🔷 Collection
                │
                ├── 🔷 List
                │       ├── 🔶 ArrayList
                │       ├── 🔶 LinkedList ─────────────────────┐
                │       ├── 🔶 Vector                          │
                │       │       └── 🔶 Stack                   │
                │       └── 🔶 CopyOnWriteArrayList            │
                │                                              │
                ├── 🔷 Set                                     │
                │       ├── 🔶 HashSet                         │
                │       │       └── 🔶 LinkedHashSet           │
                │       ├── 🔷 SortedSet                       │
                │       │       └── 🔷 NavigableSet            │
                │       │               └── 🔶 TreeSet         │
                │       ├── 🔶 EnumSet                         │
                │       ├── 🔶 CopyOnWriteArraySet             │
                │       └── 🔶 ConcurrentSkipListSet           │
                │                                              │
                └── 🔷 Queue ◄─────────────────────────────────┘
                        │          (LinkedList implements Queue)
                        ├── 🔶 PriorityQueue
                        ├── 🔷 Deque
                        │       ├── 🔶 ArrayDeque
                        │       ├── 🔶 LinkedList
                        │       └── 🔶 ConcurrentLinkedDeque
                        │
                        └── 🔷 BlockingQueue
                                ├── 🔶 ArrayBlockingQueue
                                ├── 🔶 LinkedBlockingQueue
                                ├── 🔶 PriorityBlockingQueue
                                ├── 🔶 SynchronousQueue
                                ├── 🔶 DelayQueue
                                └── 🔷 BlockingDeque
                                        └── 🔶 LinkedBlockingDeque


═══════════════════════════════════════════════════════════════════════════════
                            MAP HIERARCHY (Separate)
═══════════════════════════════════════════════════════════════════════════════

🔷 Map
        │
        ├── 🔶 HashMap
        │       └── 🔶 LinkedHashMap
        │
        ├── 🔷 SortedMap
        │       └── 🔷 NavigableMap
        │               └── 🔶 TreeMap
        │
        ├── 🔶 Hashtable (Legacy)
        │       └── 🔶 Properties
        │
        ├── 🔶 WeakHashMap
        ├── 🔶 IdentityHashMap
        ├── 🔶 EnumMap
        │
        └── 🔷 ConcurrentMap
                ├── 🔶 ConcurrentHashMap
                └── 🔷 ConcurrentNavigableMap
                        └── 🔶 ConcurrentSkipListMap
```

### 🔷 4.2 List Implementations

| Feature | ArrayList | LinkedList | Vector |
|---------|-----------|------------|--------|
| **Data Structure** | Dynamic Array | Doubly Linked List | Dynamic Array |
| **Thread Safe** | ❌ No | ❌ No | ✅ Yes (synchronized) |
| **get(index)** | O(1) 🚀 | O(n) 🐢 | O(1) |
| **add(element)** | O(1)* amortized | O(1) | O(1)* |
| **add(index)** | O(n) | O(n) | O(n) |
| **remove** | O(n) | O(1) if at position | O(n) |
| **Memory** | Less | More (node overhead) | Less |
| **Best For** | Random access | Frequent add/remove | Legacy/thread-safe |

### 🔷 4.3 Set Implementations

| Feature | HashSet | LinkedHashSet | TreeSet |
|---------|---------|---------------|---------|
| **Ordering** | ❌ No order | ✅ Insertion order | ✅ Sorted (natural/comparator) |
| **Null** | ✅ One null | ✅ One null | ❌ No null (NPE) |
| **Data Structure** | HashMap | HashMap + LinkedList | Red-Black Tree |
| **add/remove/contains** | O(1) | O(1) | O(log n) |

### 🔷 4.4 HashMap Internal Working

#### Structure (Java 8+)

```
┌─────────────────────────────────────────────────────────────────┐
│                        HashMap<K,V>                             │
├─────────────────────────────────────────────────────────────────┤
│  Initial Capacity: 16    Load Factor: 0.75    Threshold: 12     │
├─────────────────────────────────────────────────────────────────┤
│  buckets[] (Node<K,V>[])                                        │
│  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐              │
│  │  0  │  1  │  2  │  3  │  4  │  5  │  6  │ ... │              │
│  └──┬──┴─────┴──┬──┴─────┴─────┴─────┴─────┴─────┘              │
│     │           │                                               │
│     ▼           ▼                                               │
│  [Node]      [Node]                                             │
│     │           │                                               │
│     ▼           ▼                                               │
│  [Node]      [TreeNode] ← Converts when > 8 nodes               │
│     │           │                                               │
│   null       [TreeNode]                                         │
│                 │                                               │
│              [TreeNode]                                         │
└─────────────────────────────────────────────────────────────────┘
```

#### How put() Works

```java
// Step 1: Calculate hash
int hash = key.hashCode() ^ (key.hashCode() >>> 16);

// Step 2: Find bucket index
int index = (capacity - 1) & hash;

// Step 3: Handle collision
// - Bucket empty → Create new Node
// - Key exists → Replace value
// - Collision → Add to LinkedList (< 8) or Tree (>= 8)
```

#### Key Points for Interview

| Concept | Value/Condition |
|---------|-----------------|
| Default capacity | 16 |
| Load factor | 0.75 |
| Treeify threshold | 8 nodes AND table size >= 64 |
| Untreeify threshold | 6 nodes |
| Resize | When size > capacity × load factor |

### 🔷 4.5 HashMap vs ConcurrentHashMap vs Hashtable

| Feature | HashMap | Hashtable | ConcurrentHashMap |
|---------|---------|-----------|-------------------|
| **Thread-safe** | ❌ No | ✅ Yes (whole table lock) | ✅ Yes (segment/bucket lock) |
| **Null keys** | ✅ One | ❌ No | ❌ No |
| **Null values** | ✅ Multiple | ❌ No | ❌ No |
| **Performance** | 🚀 Fastest (single-thread) | 🐢 Slowest | ⚡ Best for concurrent |
| **Locking** | None | Entire table | Bucket-level (Java 8+) |
| **Iterator** | Fail-fast | Fail-fast | Weakly consistent |

### 🔷 4.6 Comparable vs Comparator

```java
// Comparable - natural ordering (in the class itself)
public class Employee implements Comparable<Employee> {
    private String name;
    private int salary;
    
    @Override
    public int compareTo(Employee other) {
        return this.salary - other.salary;  // Natural ordering by salary
    }
}

// Comparator - custom ordering (external)
Comparator<Employee> byName = (e1, e2) -> e1.getName().compareTo(e2.getName());
Comparator<Employee> bySalaryDesc = (e1, e2) -> e2.getSalary() - e1.getSalary();

// Usage
Collections.sort(employees);  // Uses Comparable
Collections.sort(employees, byName);  // Uses Comparator
employees.sort(Comparator.comparing(Employee::getName).thenComparing(Employee::getSalary));
```

### 🔷 4.7 fail-fast vs fail-safe Iterators

| Feature | fail-fast | fail-safe |
|---------|-----------|-----------|
| **Behavior** | Throws ConcurrentModificationException | Works on copy, no exception |
| **Collections** | ArrayList, HashMap, HashSet | CopyOnWriteArrayList, ConcurrentHashMap |
| **Performance** | Better (no copy needed) | Overhead of copy |
| **Modification during iteration** | ❌ Not allowed | ✅ Allowed |

### 🔷 4.8 equals() and hashCode() Contract

```java
public class Person {
    private String name;
    private int age;
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

**Contract Rules:**
1. If `a.equals(b)` is true, then `a.hashCode() == b.hashCode()` must be true
2. If `a.hashCode() != b.hashCode()`, then `a.equals(b)` must be false
3. Two objects with same hashCode are NOT necessarily equal

---

## 5. Multithreading & Concurrency

### 🔷 5.1 Thread Creation Methods

```java
// Method 1: Extend Thread class
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Running in: " + Thread.currentThread().getName());
    }
}
new MyThread().start();

// Method 2: Implement Runnable interface
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Running...");
    }
}
new Thread(new MyRunnable()).start();

// Method 3: Implement Callable interface (returns value)
class MyCallable implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        return 42;
    }
}
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(new MyCallable());
Integer result = future.get();

// Method 4: Lambda expression (Java 8+)
Thread t = new Thread(() -> System.out.println("Lambda thread"));
t.start();
```

### 🔷 5.2 Thread Lifecycle


```
                          ┌──────────────── RUNNABLE ─────────────────┐
                          │                                           │
                          │      scheduled                            │
   start()                │    ┌──────────────────►                   │   exit
(NEW) ──────────────────► │ (Ready to Run)      (Running) ├───────────┼──────────► (TERMINATED)
                          │    ◄───────────────────┘│                 │
                          │      suspended          │                 │
                          │                         │                 │
                          └─────────────────────────┼─────────────────┘
                                                    │           ▲
                          ┌──── NON-RUNNABLE ───────┼───────────┼──────┐
                          │                         │           │      │
                          │                         ▼           │      │
                          │  sleep(time)      ┌───────────┐     │      │
                          │  wait(timeout)    │   TIMED   │─────┘      │
                          │  join(timeout) ──►│  WAITING  │Time elapsed│
                          │  parkNanos()      └───────────┘     |      │
                          │  parkUntil()            │           |      │
                          │                         │ notify()  |      │
                          │                         │ notifyAll()      │
                          │                         ▼           |      │
                          │  wait()           ┌───────────┐     |      │
                          │  join()        ──►│  WAITING  │─────┤      │
                          │  park()           └───────────┘     |      │
                          │                         │ notify()  |      │
                          │                         │ notifyAll()      │
                          │                         ▼           |      │
                          │  Waiting for      ┌───────────┐     |      │
                          │  Object Monitor──►│  BLOCKED  │─────┘      │
                          │                   └───────────┘            │
                          │                   Monitor lock acquired    │
                          └────────────────────────────────────────────┘
```


## Thread States Summary

| State | Description |
|-------|-------------|
| **NEW** | Thread created but `start()` not yet called |
| **RUNNABLE** | Thread is ready to run or currently running |
| **BLOCKED** | Waiting to acquire a monitor lock |
| **WAITING** | Waiting indefinitely for another thread (wait, join, park) |
| **TIMED_WAITING** | Waiting for a specified time period |
| **TERMINATED** | Thread has completed execution |



### 🔷 5.3 Synchronization

```java
// Method-level synchronization
public synchronized void method() { 
    // Only one thread can execute at a time
}

// Block-level synchronization
public void method() {
    synchronized(this) {
        // Critical section
    }
}

// Static synchronization (class-level lock)
public static synchronized void staticMethod() {
    // Lock on Class object
}

// ReentrantLock (more flexible)
private final ReentrantLock lock = new ReentrantLock();

public void method() {
    lock.lock();
    try {
        // Critical section
    } finally {
        lock.unlock();  // Always unlock in finally
    }
}
```

### 🔷 5.4 wait(), notify(), notifyAll()

```java
// Producer-Consumer Pattern
class SharedBuffer {
    private Queue<Integer> queue = new LinkedList<>();
    private int capacity = 10;
    
    public synchronized void produce(int item) throws InterruptedException {
        while (queue.size() == capacity) {
            wait();  // Release lock and wait
        }
        queue.add(item);
        notifyAll();  // Wake up waiting consumers
    }
    
    public synchronized int consume() throws InterruptedException {
        while (queue.isEmpty()) {
            wait();  // Release lock and wait
        }
        int item = queue.poll();
        notifyAll();  // Wake up waiting producers
        return item;
    }
}
```

### 🔷 5.5 Deadlock, Livelock, Starvation

| Problem | Description | Solution |
|---------|-------------|----------|
| **Deadlock** | Two threads waiting for each other's locks forever | Lock ordering, timeout, tryLock() |
| **Livelock** | Threads keep responding to each other without progress | Random delays, backoff strategy |
| **Starvation** | Thread never gets CPU time due to priority | Fair locks, balanced priorities |

```java
// Deadlock Example
Thread 1: lock(A) → waiting for B
Thread 2: lock(B) → waiting for A

// Solution: Lock Ordering
Thread 1: lock(A) → lock(B)
Thread 2: lock(A) → lock(B)  // Same order
```

### 🔷 5.6 Executor Framework & Thread Pools

```java
// Fixed Thread Pool
ExecutorService fixed = Executors.newFixedThreadPool(10);

// Cached Thread Pool (creates threads as needed)
ExecutorService cached = Executors.newCachedThreadPool();

// Single Thread Executor
ExecutorService single = Executors.newSingleThreadExecutor();

// Scheduled Thread Pool
ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(5);

// Custom Thread Pool
ThreadPoolExecutor custom = new ThreadPoolExecutor(
    5,                      // corePoolSize
    10,                     // maxPoolSize
    60L, TimeUnit.SECONDS,  // keepAliveTime
    new LinkedBlockingQueue<>(100)  // workQueue
);
```

### 🔷 5.7 Future vs CompletableFuture

```java
// Future - blocking
ExecutorService executor = Executors.newFixedThreadPool(10);
Future<Integer> future = executor.submit(() -> {
    Thread.sleep(1000);
    return 42;
});
Integer result = future.get();  // ⚠️ Blocks until complete

// CompletableFuture - non-blocking, chainable
CompletableFuture.supplyAsync(() -> fetchData())
    .thenApply(data -> process(data))          // Transform
    .thenAccept(result -> save(result))        // Consume
    .thenRun(() -> log("Completed"))           // Run after
    .exceptionally(ex -> handleError(ex));     // Handle error

// Combine multiple futures
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");

CompletableFuture<String> combined = future1.thenCombine(future2, 
    (s1, s2) -> s1 + " " + s2);  // "Hello World"

// Wait for all
CompletableFuture.allOf(future1, future2).join();

// Wait for any
CompletableFuture.anyOf(future1, future2).join();
```

### 🔷 5.8 volatile vs synchronized

| Feature | volatile | synchronized |
|---------|----------|--------------|
| **Visibility** | ✅ Yes | ✅ Yes |
| **Atomicity** | ❌ No (only read/write) | ✅ Yes |
| **Mutual Exclusion** | ❌ No | ✅ Yes |
| **Use Case** | Flags, status variables | Compound operations |
| **Performance** | Faster | Slower (lock overhead) |

```java
// volatile - visibility only
private volatile boolean running = true;

public void stop() {
    running = false;  // Visible to all threads immediately
}

// synchronized - visibility + atomicity
private int count = 0;

public synchronized void increment() {
    count++;  // Atomic: Read + Modify + Write
}
```

### 🔷 5.9 ThreadLocal

```java
public class UserContext {
    private static ThreadLocal<User> userHolder = new ThreadLocal<>();
    
    public static void setUser(User user) {
        userHolder.set(user);
    }
    
    public static User getUser() {
        return userHolder.get();
    }
    
    public static void clear() {
        userHolder.remove();  // Important to avoid memory leaks!
    }
}
```

### 🔷 5.10 Fork/Join Framework

```java
public class SumTask extends RecursiveTask<Long> {
    private final long[] array;
    private final int start, end;
    private static final int THRESHOLD = 10000;
    
    public SumTask(long[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }
    
    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            // Compute directly
            long sum = 0;
            for (int i = start; i < end; i++) {
                sum += array[i];
            }
            return sum;
        } else {
            // Split into subtasks
            int mid = (start + end) / 2;
            SumTask left = new SumTask(array, start, mid);
            SumTask right = new SumTask(array, mid, end);
            
            left.fork();  // Execute asynchronously
            Long rightResult = right.compute();  // Execute synchronously
            Long leftResult = left.join();  // Wait for result
            
            return leftResult + rightResult;
        }
    }
}

// Usage
ForkJoinPool pool = new ForkJoinPool();
Long result = pool.invoke(new SumTask(array, 0, array.length));
```

---

## 6. Java 8+ Features

### 🔷 6.1 Lambda Expressions

```java
// Before Java 8
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};

// After Java 8
Runnable r = () -> System.out.println("Hello");

// With parameters
Comparator<String> c = (s1, s2) -> s1.compareTo(s2);

// Multi-line
Comparator<String> c = (s1, s2) -> {
    int result = s1.length() - s2.length();
    return result != 0 ? result : s1.compareTo(s2);
};
```

### 🔷 6.2 Functional Interfaces

| Interface | Method | Description | Example |
|-----------|--------|-------------|---------|
| `Predicate<T>` | `boolean test(T)` | Test a condition | Filtering |
| `Function<T,R>` | `R apply(T)` | Transform input to output | Mapping |
| `Consumer<T>` | `void accept(T)` | Consume a value | Printing, saving |
| `Supplier<T>` | `T get()` | Supply a value | Factory, lazy init |
| `BiFunction<T,U,R>` | `R apply(T,U)` | Two inputs to output | Combining |
| `UnaryOperator<T>` | `T apply(T)` | Same type in/out | Increment |
| `BinaryOperator<T>` | `T apply(T,T)` | Two same types to one | Sum, max |

```java
Predicate<String> isEmpty = String::isEmpty;
Function<String, Integer> length = String::length;
Consumer<String> print = System.out::println;
Supplier<LocalDate> today = LocalDate::now;
BinaryOperator<Integer> sum = (a, b) -> a + b;
```

### 🔷 6.3 Method References

```java
// Static method reference
Function<String, Integer> parseInt = Integer::parseInt;

// Instance method of particular object
Consumer<String> printer = System.out::println;

// Instance method of arbitrary object
Function<String, String> toUpper = String::toUpperCase;

// Constructor reference
Supplier<ArrayList<String>> listSupplier = ArrayList::new;
Function<Integer, int[]> arrayCreator = int[]::new;
```

### 🔷 6.4 Stream API

```java
List<Employee> employees = getEmployees();

// Filter, Map, Collect
List<String> names = employees.stream()
    .filter(e -> e.getSalary() > 50000)
    .map(Employee::getName)
    .sorted()
    .collect(Collectors.toList());

// Reduce
int totalSalary = employees.stream()
    .mapToInt(Employee::getSalary)
    .sum();

// Grouping
Map<Department, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// Partitioning
Map<Boolean, List<Employee>> partitioned = employees.stream()
    .collect(Collectors.partitioningBy(e -> e.getSalary() > 50000));

// flatMap - flatten nested collections
List<String> allSkills = employees.stream()
    .flatMap(e -> e.getSkills().stream())
    .distinct()
    .collect(Collectors.toList());

// Parallel Stream
long count = employees.parallelStream()
    .filter(e -> e.getSalary() > 50000)
    .count();
```

#### Stream Operations

| Intermediate (Lazy) | Terminal (Eager) |
|---------------------|------------------|
| `filter()`, `map()`, `flatMap()` | `collect()`, `forEach()` |
| `sorted()`, `distinct()` | `reduce()`, `count()`, `sum()` |
| `limit()`, `skip()` | `findFirst()`, `findAny()` |
| `peek()` | `anyMatch()`, `allMatch()`, `noneMatch()` |

### 🔷 6.5 Optional Class

```java
Optional<User> user = findUserById(id);

// Safe value extraction
String name = user.map(User::getName).orElse("Unknown");

// Throw if empty
User u = user.orElseThrow(() -> new UserNotFoundException(id));

// Conditional execution
user.ifPresent(u -> sendEmail(u));

// Java 9+
user.ifPresentOrElse(
    u -> sendEmail(u),
    () -> log.warn("User not found")
);

// Chaining with flatMap
String city = user
    .flatMap(User::getAddress)
    .flatMap(Address::getCity)
    .orElse("Unknown");

// Stream conversion (Java 9+)
List<User> users = optionalUser.stream().collect(Collectors.toList());
```

### 🔷 6.6 Date/Time API (java.time)

```java
// Current date/time
LocalDate date = LocalDate.now();           // 2024-12-18
LocalTime time = LocalTime.now();           // 14:30:45.123
LocalDateTime dateTime = LocalDateTime.now();
ZonedDateTime zoned = ZonedDateTime.now();
Instant instant = Instant.now();            // UTC timestamp

// Creating specific date/time
LocalDate birthday = LocalDate.of(1990, Month.JANUARY, 15);
LocalTime meetingTime = LocalTime.of(14, 30);

// Parsing
LocalDate parsed = LocalDate.parse("2024-12-18");
LocalDateTime parsedDT = LocalDateTime.parse("2024-12-18T14:30:00");

// Formatting
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd-MM-yyyy");
String formatted = date.format(formatter);

// Manipulation
LocalDate tomorrow = date.plusDays(1);
LocalDate nextMonth = date.plusMonths(1);
LocalDate lastYear = date.minusYears(1);

// Period & Duration
Period period = Period.between(startDate, endDate);
Duration duration = Duration.between(startTime, endTime);
```

---

## 7. Java 9-21 Features

### 🔷 7.1 Java 9 - Modules (JPMS)

```java
// module-info.java
module com.myapp.core {
    requires java.sql;
    requires transitive com.myapp.utils;
    
    exports com.myapp.core.api;
    exports com.myapp.core.model to com.myapp.web;
    
    opens com.myapp.core.internal to com.google.gson;
    
    provides com.myapp.core.api.Service 
        with com.myapp.core.impl.ServiceImpl;
    
    uses com.myapp.core.api.Plugin;
}
```

### 🔷 7.2 Java 10 - var (Local Variable Type Inference)

```java
// Instead of explicit types
var list = new ArrayList<String>();      // ArrayList<String>
var map = new HashMap<String, Integer>(); // HashMap<String, Integer>
var stream = list.stream();               // Stream<String>

// Cannot use with:
// - Class fields
// - Method parameters
// - null initialization
// - Lambda parameters (until Java 11)
```

### 🔷 7.3 Java 14+ - Records

```java
// Before: verbose POJO
public class Person {
    private final String name;
    private final int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // getters, equals, hashCode, toString...
}

// After: concise record
public record Person(String name, int age) {
    // Compact constructor for validation
    public Person {
        if (age < 0) throw new IllegalArgumentException("Age cannot be negative");
    }
    
    // Additional methods
    public String greeting() {
        return "Hello, " + name;
    }
}

Person person = new Person("John", 30);
String name = person.name();  // Getter (not getName())
```

### 🔷 7.4 Java 15+ - Sealed Classes

```java
// Restrict which classes can extend
public sealed class Shape 
    permits Circle, Rectangle, Triangle {
    // ...
}

// Must be final, sealed, or non-sealed
public final class Circle extends Shape { }
public sealed class Rectangle extends Shape permits Square { }
public non-sealed class Triangle extends Shape { }
```

### 🔷 7.5 Java 16+ - Pattern Matching for instanceof

```java
// Before
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// After
if (obj instanceof String s) {
    System.out.println(s.length());
}

// With negation
if (!(obj instanceof String s)) {
    return;
}
// s is in scope here
```

### 🔷 7.6 Java 17+ - Pattern Matching for switch

```java
// Enhanced switch with patterns
String result = switch (obj) {
    case Integer i -> "Integer: " + i;
    case String s -> "String: " + s;
    case Long l -> "Long: " + l;
    case null -> "Null value";
    default -> "Unknown type";
};

// With guards
String result = switch (obj) {
    case String s when s.length() > 10 -> "Long string";
    case String s -> "Short string";
    default -> "Not a string";
};
```

### 🔷 7.7 Java 15+ - Text Blocks

```java
// Multi-line strings
String html = """
    <html>
        <body>
            <h1>Hello, World!</h1>
        </body>
    </html>
    """;

String json = """
    {
        "name": "John",
        "age": 30,
        "city": "New York"
    }
    """;

String sql = """
    SELECT id, name, email
    FROM users
    WHERE status = 'ACTIVE'
    ORDER BY name
    """;
```

### 🔷 7.8 Java 21 - Virtual Threads

```java
// Platform thread (traditional)
Thread platformThread = new Thread(() -> {
    System.out.println("Running on: " + Thread.currentThread());
});

// Virtual thread (lightweight)
Thread virtualThread = Thread.ofVirtual().start(() -> {
    System.out.println("Running on virtual thread");
});

// Virtual thread per task executor
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 10_000).forEach(i -> {
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1));
            return i;
        });
    });
}  // Completes all tasks before closing

// Benefits:
// - Millions of virtual threads possible
// - Low memory footprint
// - Automatic scheduling
// - No thread pooling needed
```

---

## 8. Generics

### 🔷 8.1 Generic Classes and Methods

```java
// Generic class
public class Box<T> {
    private T content;
    
    public void set(T content) { this.content = content; }
    public T get() { return content; }
}

// Generic method
public <T> T getFirst(List<T> list) {
    return list.isEmpty() ? null : list.get(0);
}

// Multiple type parameters
public class Pair<K, V> {
    private K key;
    private V value;
    
    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }
}
```

### 🔷 8.2 Bounded Type Parameters

```java
// Upper bound - T must extend Number
public <T extends Number> double sum(List<T> list) {
    return list.stream().mapToDouble(Number::doubleValue).sum();
}

// Multiple bounds
public <T extends Comparable<T> & Serializable> T max(T a, T b) {
    return a.compareTo(b) > 0 ? a : b;
}
```

### 🔷 8.3 Wildcards

```java
// Unbounded wildcard - unknown type
public void printList(List<?> list) {
    for (Object item : list) {
        System.out.println(item);
    }
}

// Upper bounded - ? extends Type (producer)
public double sumOfNumbers(List<? extends Number> list) {
    return list.stream().mapToDouble(Number::doubleValue).sum();
}

// Lower bounded - ? super Type (consumer)
public void addIntegers(List<? super Integer> list) {
    list.add(1);
    list.add(2);
}
```

### 🔷 8.4 PECS Principle

> **Producer Extends, Consumer Super**

```java
// Producer - read from it (use extends)
public void copy(List<? extends Number> source,   // Producer
                 List<? super Number> dest) {     // Consumer
    for (Number n : source) {
        dest.add(n);
    }
}
```

### 🔷 8.5 Type Erasure

```java
// At compile time:
List<String> strings = new ArrayList<String>();
List<Integer> integers = new ArrayList<Integer>();

// At runtime (after erasure):
List strings = new ArrayList();
List integers = new ArrayList();
// Both become raw List type!

// This is why you can't:
// - Use instanceof with generics
// - Create arrays of generic types
// - Catch generic exceptions
```

---

## 9. Java I/O & NIO

### 🔷 9.1 I/O Streams Hierarchy

```
                      InputStream                    Reader
                          │                            │
        ┌─────────────────┼─────────────────┐    ┌─────┴─────┐
   FileInputStream  ByteArrayInputStream    │  FileReader  StringReader
                          │                 │
               BufferedInputStream    ObjectInputStream


                      OutputStream                   Writer
                          │                            │
        ┌─────────────────┼─────────────────┐    ┌─────┴─────┐
   FileOutputStream ByteArrayOutputStream   │  FileWriter  StringWriter
                          │                 │
               BufferedOutputStream   ObjectOutputStream
```

### 🔷 9.2 File Operations

```java
// Reading file (Java 7+)
String content = Files.readString(Path.of("file.txt"));
List<String> lines = Files.readAllLines(Path.of("file.txt"));

// Writing file
Files.writeString(Path.of("file.txt"), content);
Files.write(Path.of("file.txt"), lines);

// Buffered reading
try (BufferedReader reader = Files.newBufferedReader(Path.of("file.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
}

// Stream lines
try (Stream<String> lines = Files.lines(Path.of("file.txt"))) {
    lines.filter(line -> line.contains("error"))
         .forEach(System.out::println);
}
```

### 🔷 9.3 Serialization

```java
public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    
    private String name;
    private transient String password;  // Not serialized
    private int age;
}

// Serialize
try (ObjectOutputStream oos = new ObjectOutputStream(
        new FileOutputStream("user.ser"))) {
    oos.writeObject(user);
}

// Deserialize
try (ObjectInputStream ois = new ObjectInputStream(
        new FileInputStream("user.ser"))) {
    User user = (User) ois.readObject();
}
```

### 🔷 9.4 NIO.2 (Java 7+)

```java
// Path and Files
Path path = Path.of("folder", "file.txt");
Path path = Paths.get("folder/file.txt");

boolean exists = Files.exists(path);
boolean isDir = Files.isDirectory(path);
long size = Files.size(path);

// Create directories
Files.createDirectory(Path.of("newDir"));
Files.createDirectories(Path.of("parent/child/grandchild"));

// Copy and Move
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);

// Walk file tree
try (Stream<Path> paths = Files.walk(Path.of("rootDir"))) {
    paths.filter(Files::isRegularFile)
         .filter(p -> p.toString().endsWith(".java"))
         .forEach(System.out::println);
}
```

---

## 10. Design Patterns

### 🔷 10.1 Creational Patterns

#### Singleton

```java
// Thread-safe Singleton (Bill Pugh)
public class Singleton {
    private Singleton() {}
    
    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}

// Enum Singleton (Joshua Bloch recommended)
public enum Singleton {
    INSTANCE;
    
    public void doSomething() { }
}
```

#### Factory Method

```java
public interface Vehicle { void drive(); }
public class Car implements Vehicle { public void drive() { } }
public class Bike implements Vehicle { public void drive() { } }

public class VehicleFactory {
    public static Vehicle create(String type) {
        return switch (type) {
            case "car" -> new Car();
            case "bike" -> new Bike();
            default -> throw new IllegalArgumentException("Unknown type");
        };
    }
}
```

#### Builder

```java
public class User {
    private final String name;
    private final String email;
    private final int age;
    
    private User(Builder builder) {
        this.name = builder.name;
        this.email = builder.email;
        this.age = builder.age;
    }
    
    public static class Builder {
        private String name;
        private String email;
        private int age;
        
        public Builder name(String name) { this.name = name; return this; }
        public Builder email(String email) { this.email = email; return this; }
        public Builder age(int age) { this.age = age; return this; }
        
        public User build() { return new User(this); }
    }
}

// Usage
User user = new User.Builder()
    .name("John")
    .email("john@example.com")
    .age(30)
    .build();
```

### 🔷 10.2 Structural Patterns

#### Adapter

```java
// Target interface
public interface MediaPlayer {
    void play(String filename);
}

// Adaptee
public class VlcPlayer {
    public void playVlc(String filename) { }
}

// Adapter
public class MediaAdapter implements MediaPlayer {
    private VlcPlayer vlcPlayer = new VlcPlayer();
    
    @Override
    public void play(String filename) {
        vlcPlayer.playVlc(filename);
    }
}
```

#### Decorator

```java
public interface Coffee {
    double getCost();
    String getDescription();
}

public class SimpleCoffee implements Coffee {
    public double getCost() { return 2.0; }
    public String getDescription() { return "Simple Coffee"; }
}

public class MilkDecorator implements Coffee {
    private Coffee coffee;
    
    public MilkDecorator(Coffee coffee) { this.coffee = coffee; }
    
    public double getCost() { return coffee.getCost() + 0.5; }
    public String getDescription() { return coffee.getDescription() + ", Milk"; }
}

// Usage
Coffee coffee = new MilkDecorator(new SimpleCoffee());
```

### 🔷 10.3 Behavioral Patterns

#### Strategy

```java
public interface PaymentStrategy {
    void pay(double amount);
}

public class CreditCardPayment implements PaymentStrategy {
    public void pay(double amount) { /* Credit card logic */ }
}

public class PayPalPayment implements PaymentStrategy {
    public void pay(double amount) { /* PayPal logic */ }
}

public class ShoppingCart {
    private PaymentStrategy paymentStrategy;
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public void checkout(double amount) {
        paymentStrategy.pay(amount);
    }
}
```

#### Observer

```java
public interface Observer {
    void update(String message);
}

public class Subject {
    private List<Observer> observers = new ArrayList<>();
    
    public void addObserver(Observer o) { observers.add(o); }
    public void removeObserver(Observer o) { observers.remove(o); }
    
    public void notifyObservers(String message) {
        observers.forEach(o -> o.update(message));
    }
}
```

---

## 11. SOLID Principles

### 🔷 S - Single Responsibility Principle

> A class should have only one reason to change.

```java
// ❌ Bad: Multiple responsibilities
public class User {
    public void saveToDatabase() { }
    public void sendEmail() { }
    public void generateReport() { }
}

// ✅ Good: Single responsibility
public class User { /* Only user data */ }
public class UserRepository { public void save(User user) { } }
public class EmailService { public void send(Email email) { } }
public class ReportGenerator { public Report generate() { } }
```

### 🔷 O - Open/Closed Principle

> Open for extension, closed for modification.

```java
// ❌ Bad: Must modify class to add new shapes
public class AreaCalculator {
    public double calculate(Object shape) {
        if (shape instanceof Circle) { /* ... */ }
        else if (shape instanceof Rectangle) { /* ... */ }
        // Must add new else-if for each shape
    }
}

// ✅ Good: Extend without modification
public interface Shape {
    double calculateArea();
}

public class Circle implements Shape {
    public double calculateArea() { return Math.PI * radius * radius; }
}

public class Rectangle implements Shape {
    public double calculateArea() { return width * height; }
}
```

### 🔷 L - Liskov Substitution Principle

> Subtypes must be substitutable for their base types.

```java
// ❌ Bad: Square breaks Rectangle behavior
public class Rectangle {
    public void setWidth(int w) { width = w; }
    public void setHeight(int h) { height = h; }
}

public class Square extends Rectangle {
    public void setWidth(int w) { width = height = w; }  // Breaks expectation!
}

// ✅ Good: Use composition or separate hierarchy
public interface Shape {
    double getArea();
}
```

### 🔷 I - Interface Segregation Principle

> Clients should not be forced to depend on interfaces they don't use.

```java
// ❌ Bad: Fat interface
public interface Worker {
    void work();
    void eat();
    void sleep();
}

public class Robot implements Worker {
    public void work() { }
    public void eat() { }   // Robot doesn't eat!
    public void sleep() { } // Robot doesn't sleep!
}

// ✅ Good: Segregated interfaces
public interface Workable { void work(); }
public interface Eatable { void eat(); }
public interface Sleepable { void sleep(); }

public class Robot implements Workable {
    public void work() { }
}
```

### 🔷 D - Dependency Inversion Principle

> High-level modules should not depend on low-level modules. Both should depend on abstractions.

```java
// ❌ Bad: Direct dependency on low-level module
public class EmailNotifier {
    public void notify(String message) { /* ... */ }
}

public class UserService {
    private EmailNotifier notifier = new EmailNotifier(); // Tight coupling!
}

// ✅ Good: Depend on abstraction
public interface Notifier {
    void notify(String message);
}

public class EmailNotifier implements Notifier {
    public void notify(String message) { /* ... */ }
}

public class UserService {
    private Notifier notifier;
    
    public UserService(Notifier notifier) { // Dependency Injection
        this.notifier = notifier;
    }
}
```

---

<p align="center">
  <a href="README.md">← Back to Main Guide</a>
</p>

