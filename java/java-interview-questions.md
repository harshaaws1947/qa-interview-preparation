# Java Interview Questions for QA/SDET

## Core Java Basics

### 1. What are the main features of Java?
- **Platform Independent**: Write once, run anywhere (WORA)
- **Object-Oriented**: Supports encapsulation, inheritance, polymorphism, abstraction
- **Robust**: Strong memory management, exception handling
- **Secure**: No explicit pointers, bytecode verification
- **Multithreaded**: Built-in support for concurrent programming

### 2. What is the difference between JDK, JRE, and JVM?
| Component | Description |
|-----------|-------------|
| **JVM** | Java Virtual Machine - executes bytecode |
| **JRE** | JVM + libraries to run Java programs |
| **JDK** | JRE + development tools (compiler, debugger) |

### 3. Explain the main() method signature
```java
public static void main(String[] args)
```
- `public`: Accessible from anywhere
- `static`: Can be called without creating an object
- `void`: Returns nothing
- `String[] args`: Command line arguments

### 4. What are primitive data types in Java?
| Type | Size | Default |
|------|------|---------|
| byte | 1 byte | 0 |
| short | 2 bytes | 0 |
| int | 4 bytes | 0 |
| long | 8 bytes | 0L |
| float | 4 bytes | 0.0f |
| double | 8 bytes | 0.0d |
| char | 2 bytes | '\u0000' |
| boolean | 1 bit | false |

### 5. What is the difference between == and equals()?
- `==` compares **references** (memory addresses)
- `equals()` compares **content/values**

```java
String s1 = new String("hello");
String s2 = new String("hello");
s1 == s2;      // false (different objects)
s1.equals(s2); // true (same content)
```

## OOP Concepts

### 6. What are the four pillars of OOP?

**1. Encapsulation**
- Binding data and methods together
- Using private variables with public getters/setters

**2. Inheritance**
- Child class inherits properties from parent class
- `extends` keyword for classes, `implements` for interfaces

**3. Polymorphism**
- Same method behaves differently
- Compile-time (overloading) and Runtime (overriding)

**4. Abstraction**
- Hiding implementation details
- Using abstract classes and interfaces

### 7. What is the difference between method overloading and overriding?

| Overloading | Overriding |
|-------------|------------|
| Same class | Different classes (parent-child) |
| Different parameters | Same parameters |
| Compile-time polymorphism | Runtime polymorphism |
| Return type can differ | Return type must be same or covariant |

### 8. What is an abstract class vs interface?

| Abstract Class | Interface |
|----------------|-----------|
| Can have abstract and concrete methods | All methods abstract (until Java 8) |
| Can have constructors | No constructors |
| Can have instance variables | Only static final constants |
| `extends` (single) | `implements` (multiple) |

### 9. What is the difference between `this` and `super`?
- `this`: Refers to current class instance
- `super`: Refers to parent class instance

## Strings

### 10. Why is String immutable in Java?
- **Security**: Used in class loading, network connections
- **Caching**: String pool enables sharing
- **Thread Safety**: Immutable objects are thread-safe
- **Hashcode**: Can be cached for HashMap keys

### 11. What is the String Pool?
- Special memory area in heap for String literals
- Strings with same content share the same memory

```java
String s1 = "hello";  // Goes to pool
String s2 = "hello";  // Reuses from pool
String s3 = new String("hello"); // New object in heap
```

### 12. StringBuilder vs StringBuffer
| StringBuilder | StringBuffer |
|---------------|--------------|
| Not thread-safe | Thread-safe (synchronized) |
| Faster | Slower |
| Use in single-threaded | Use in multi-threaded |

## Collections Framework

### 13. List vs Set vs Map

| Collection | Duplicates | Order | Null |
|------------|------------|-------|------|
| List | Allowed | Maintains insertion order | Allowed |
| Set | Not allowed | HashSet-no order, LinkedHashSet-ordered | One null (HashSet) |
| Map | Keys unique, values can duplicate | HashMap-no order | One null key |

### 14. ArrayList vs LinkedList

| ArrayList | LinkedList |
|-----------|------------|
| Dynamic array | Doubly-linked list |
| Fast random access O(1) | Slow random access O(n) |
| Slow insertion/deletion | Fast insertion/deletion |
| Less memory | More memory (node pointers) |

### 15. HashMap vs HashTable vs ConcurrentHashMap

| HashMap | HashTable | ConcurrentHashMap |
|---------|-----------|-------------------|
| Not synchronized | Synchronized | Partially synchronized |
| Allows null key/value | No nulls | No nulls |
| Fast | Slow | Fast |
| Single-threaded | Multi-threaded | Multi-threaded |

### 16. What is the difference between fail-fast and fail-safe iterators?
- **Fail-fast**: Throws ConcurrentModificationException if collection modified during iteration (ArrayList, HashMap)
- **Fail-safe**: Works on copy, no exception (CopyOnWriteArrayList, ConcurrentHashMap)

## Exception Handling

### 17. What is the difference between Checked and Unchecked Exceptions?

| Checked | Unchecked |
|---------|-----------|
| Compile-time | Runtime |
| Must be handled | Optional handling |
| IOException, SQLException | NullPointerException, ArrayIndexOutOfBoundsException |

### 18. Explain try-catch-finally
```java
try {
    // Code that may throw exception
} catch (ExceptionType e) {
    // Handle exception
} finally {
    // Always executes (cleanup)
}
```

### 19. What is try-with-resources?
- Automatically closes resources implementing AutoCloseable
```java
try (FileReader fr = new FileReader("file.txt")) {
    // Use resource
} // Auto-closed
```

## Multithreading

### 20. How to create a thread in Java?
**Method 1: Extend Thread**
```java
class MyThread extends Thread {
    public void run() { }
}
```

**Method 2: Implement Runnable**
```java
class MyRunnable implements Runnable {
    public void run() { }
}
```

### 21. What is the difference between wait() and sleep()?

| wait() | sleep() |
|--------|---------|
| Object class | Thread class |
| Releases lock | Holds lock |
| Must be in synchronized block | Can be anywhere |
| Woken by notify() | Wakes after time |

### 22. What is synchronization?
- Prevents multiple threads from accessing shared resources simultaneously
- Using `synchronized` keyword or Lock objects

## Java 8 Features

### 23. What are Lambda Expressions?
- Anonymous functions with concise syntax
```java
// Before Java 8
Runnable r = new Runnable() {
    public void run() { System.out.println("Hello"); }
};

// Java 8 Lambda
Runnable r = () -> System.out.println("Hello");
```

### 24. What is a Functional Interface?
- Interface with exactly one abstract method
- `@FunctionalInterface` annotation
- Examples: Runnable, Comparator, Predicate

### 25. What is Stream API?
- Process collections in a functional style
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int sum = numbers.stream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * 2)
    .reduce(0, Integer::sum);
```

### 26. What is Optional?
- Container to avoid NullPointerException
```java
Optional<String> opt = Optional.ofNullable(getValue());
String result = opt.orElse("default");
```

## Interview Coding Questions

### 27. Reverse a String
```java
String reversed = new StringBuilder(str).reverse().toString();
```

### 28. Check if String is Palindrome
```java
boolean isPalindrome = str.equals(new StringBuilder(str).reverse().toString());
```

### 29. Find duplicates in an array
```java
Set<Integer> seen = new HashSet<>();
for (int num : arr) {
    if (!seen.add(num)) {
        System.out.println("Duplicate: " + num);
    }
}
```

### 30. Sort a HashMap by values
```java
map.entrySet().stream()
    .sorted(Map.Entry.comparingByValue())
    .collect(Collectors.toLinkedHashMap());
```

---

## Quick Tips for Interview

1. **Know your basics** - Collections, OOP, Exception handling
2. **Practice coding** - Strings, Arrays, HashMap problems
3. **Understand complexity** - Big O notation
4. **Java 8 features** - Lambdas, Streams, Optional
5. **Design patterns** - Singleton, Factory, Page Object Model
