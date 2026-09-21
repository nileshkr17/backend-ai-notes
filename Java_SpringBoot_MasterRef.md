# Java + Spring Boot: Complete Reference
### Fresher → SDE-3 → Architect · Single Doc · A to Z

---

## TL;DR
> One doc covering everything: Java type system → OOP → Collections → JVM internals → GC → Concurrency → Spring Boot → AOP → Transactions → Security → Production patterns. Read top to bottom as a fresher. Use as a reference as a senior.

---

# PHASE 1 — Core Java

---

## 1. Type System

### Primitives

| Type | Size | Range | Default |
|---|---|---|---|
| `byte` | 1 byte | -128 to 127 | 0 |
| `short` | 2 bytes | -32,768 to 32,767 | 0 |
| `int` | 4 bytes | -2^31 to 2^31-1 | 0 |
| `long` | 8 bytes | -2^63 to 2^63-1 | 0L |
| `float` | 4 bytes | ~±3.4e38 (7 decimal digits) | 0.0f |
| `double` | 8 bytes | ~±1.8e308 (15 decimal digits) | 0.0d |
| `char` | 2 bytes | 0 to 65,535 (Unicode) | '\u0000' |
| `boolean` | JVM-defined | true/false | false |

### Autoboxing Traps

```java
// Integer cache: -128 to 127 are cached
Integer a = 127, b = 127;
System.out.println(a == b);   // true — same cached object

Integer c = 128, d = 128;
System.out.println(c == d);   // false — different heap objects
System.out.println(c.equals(d)); // true — always use equals()

// NPE trap
Integer x = null;
int y = x;  // NullPointerException — unboxing null
```

### Type Promotion
```java
byte a = 10, b = 20;
// byte c = a + b;  // COMPILE ERROR — a+b is promoted to int
int c = a + b;      // OK

// Why? JVM arithmetic works on int minimum. byte/short/char
// are widened to int before any arithmetic operation.
```

### var (Java 10+)
```java
var list = new ArrayList<String>(); // inferred: ArrayList<String>
var map = new HashMap<String, Integer>();

// Cannot: var x;            — no initializer
// Cannot: var x = null;     — can't infer type from null
// Cannot as field/param/return type — local variables ONLY
```

---

## 2. String Deep Dive

### Why String is Immutable — 4 Pillars

```java
// Pillar 1: final class — no subclassing
public final class String { }

// Pillar 2: private final char[] (Java 8) / byte[] (Java 9+)
private final byte[] value;  // can't reassign reference, but...

// Pillar 3: No method exposes the array — defensive copy
public char[] toCharArray() {
    return Arrays.copyOf(value, value.length); // copy, not original
}

// Pillar 4: All "mutating" methods return NEW strings
String s = "hello";
String upper = s.toUpperCase(); // new String "HELLO", s unchanged
```

### String Pool
```java
String a = "hello";           // goes to String pool
String b = "hello";           // same pool object
String c = new String("hello"); // new heap object, NOT pool

System.out.println(a == b);        // true
System.out.println(a == c);        // false
System.out.println(a.equals(c));   // true
System.out.println(a == c.intern()); // true — intern() returns pool ref
```

### Java 9 Compact Strings
```java
// Java 8: char[] — 2 bytes per character always
// Java 9+: byte[] — 1 byte for Latin-1 chars, 2 bytes for others
// "hello" uses 5 bytes instead of 10 — ~50% memory saving for ASCII
// Transparent — no API change, JVM handles it internally
```

### StringBuilder vs String concatenation
```java
// Loop with + : creates new String object every iteration — O(n²)
String result = "";
for (int i = 0; i < 1000; i++) result += i; // BAD

// StringBuilder: single mutable buffer — O(n)
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) sb.append(i); // GOOD
String result = sb.toString();

// Compile-time concatenation of literals is fine — compiler optimizes
String s = "hello" + " " + "world"; // compiled to "hello world"
```

---

## 3. OOP Complete

### Encapsulation
```java
public class BankAccount {
    private double balance; // hidden state

    public void deposit(double amount) {
        if (amount <= 0) throw new IllegalArgumentException("Amount must be positive");
        balance += amount;
    }
    public double getBalance() { return balance; } // controlled read
    // No setter for balance — only deposit/withdraw allowed
}
// Without encapsulation: account.balance = -9999 — nothing stops it
```

### Inheritance + Dynamic Dispatch
```java
class Animal {
    public String sound() { return "..."; }
}
class Dog extends Animal {
    @Override
    public String sound() { return "Woof"; }
}

Animal a = new Dog(); // reference type: Animal, runtime type: Dog
a.sound(); // "Woof" — JVM uses runtime type (vtable lookup)
// This is runtime polymorphism / dynamic dispatch
```

### Method Overloading vs Overriding

| | Overloading | Overriding |
|---|---|---|
| Resolved at | Compile time | Runtime |
| Signature | Different params | Same params |
| Return type | Can differ | Must be same (or covariant) |
| Access | Any | Can't reduce visibility |
| static | Can overload static | Can't override static (hiding) |

```java
// Overloading trap — compile time resolution
void print(Object o) { System.out.println("Object"); }
void print(String s) { System.out.println("String"); }

Object o = "hello"; // reference type: Object
print(o); // prints "Object" — resolved at compile time using reference type
```

### Constructor Chaining
```java
class Person {
    String name; int age;

    Person() { this("Unknown", 0); } // calls 2-arg constructor
    Person(String name) { this(name, 0); }
    Person(String name, int age) {
        this.name = name;
        this.age = age;
        // super() implicitly called here if no explicit super()
    }
}
// Rule: this() or super() must be FIRST statement in constructor
```

### Inner Classes
```java
// Static nested — no reference to outer class, like a helper class
class Outer {
    static class StaticNested { } // Outer.StaticNested

    // Non-static inner — has implicit reference to outer instance
    class Inner { } // new Outer().new Inner()

    void method() {
        // Local class — inside method, can access effectively final locals
        class Local { }

        // Anonymous class — instantiate and define in one shot
        Runnable r = new Runnable() {
            @Override public void run() { System.out.println("run"); }
        };
        // Java 8+: replace anonymous Runnable with lambda
        Runnable r2 = () -> System.out.println("run");
    }
}
```

### Enums
```java
public enum Status {
    ACTIVE("Active"), INACTIVE("Inactive"), PENDING("Pending");

    private final String label;
    Status(String label) { this.label = label; }
    public String getLabel() { return label; }
}

// Enum in switch
switch (status) {
    case ACTIVE -> process();
    case INACTIVE -> skip();
}

// EnumSet — much faster than HashSet for enums (uses bit vector)
EnumSet<Status> active = EnumSet.of(Status.ACTIVE, Status.PENDING);

// EnumMap — faster than HashMap for enum keys
EnumMap<Status, String> labels = new EnumMap<>(Status.class);
```

---

## 4. Interfaces vs Abstract Classes

| | Abstract Class | Interface |
|---|---|---|
| State | Yes (instance fields) | `public static final` only |
| Constructor | Yes | No |
| Methods | Abstract + concrete | Abstract + default + static (Java 8) + private (Java 9) |
| Extends/implements | Single | Multiple |
| Use when | IS-A with shared state | CAN-DO contract |

```java
// Default method — diamond problem resolution
interface A { default void hello() { System.out.println("A"); } }
interface B { default void hello() { System.out.println("B"); } }

class C implements A, B {
    @Override
    public void hello() {
        A.super.hello(); // must explicitly resolve — pick one
    }
}
```

### Functional Interfaces
```java
@FunctionalInterface // exactly 1 abstract method
interface Transformer<T, R> {
    R transform(T input);
    // can have default/static methods — still functional
}

// Java built-in functional interfaces:
Function<String, Integer>   // T → R
BiFunction<T, U, R>         // T, U → R
Predicate<T>                // T → boolean
Consumer<T>                 // T → void
Supplier<T>                 // () → T
UnaryOperator<T>            // T → T
BinaryOperator<T>           // T, T → T
```

---

## 5. Generics

### PECS — Producer Extends, Consumer Super
```java
// ? extends T — you READ from it (it produces T values)
List<? extends Number> numbers = new ArrayList<Integer>();
Number n = numbers.get(0); // OK — guaranteed to be Number or subtype
// numbers.add(1);          // ERROR — don't know exact type

// ? super T — you WRITE to it (it consumes T values)
List<? super Integer> ints = new ArrayList<Number>();
ints.add(1);               // OK — Integer is valid
// Integer i = ints.get(0); // ERROR — could return Object
```

### Type Erasure
```java
// At compile time:
List<String> strings = new ArrayList<>();
List<Integer> ints = new ArrayList<>();

// At runtime: both are just List (raw type)
strings.getClass() == ints.getClass(); // true — both ArrayList.class

// You cannot:
// new T()                     — type unknown at runtime
// if (x instanceof List<String>) // can't check generic type
// T[] arr = new T[10]         // can't create generic array

// Heap pollution: raw type bypasses generic safety
List rawList = new ArrayList<String>();
rawList.add(42);                        // no compile error
List<String> stringList = rawList;      // unchecked warning
String s = stringList.get(0);          // ClassCastException at runtime
```

---

## 6. Collections — Internals

### ArrayList
```java
// Backed by Object[] array
// Default initial capacity: 10
// Growth: newCapacity = oldCapacity + (oldCapacity >> 1)  → x1.5
// Add at end: O(1) amortized (occasional O(n) copy)
// Add at index: O(n) — shifts elements right
// Get by index: O(1)
// Contains/remove by value: O(n)

List<String> list = new ArrayList<>(100); // pre-size if you know count
```

### HashMap Internals
```java
// Structure: array of Node<K,V>[] (buckets)
// Default capacity: 16, load factor: 0.75
// Resize trigger: size > capacity * loadFactor
// On resize: capacity doubles, all entries rehashed

// Hash function — reduces collisions:
// h = key.hashCode()
// index = (h ^ (h >>> 16)) & (capacity - 1)

// Collision handling:
// Java 7: linked list chaining
// Java 8+: linked list → Red-Black tree when chain length > 8
//          tree → linked list when chain length < 6 (untreeify)
// Why 8? Probability of 8 collisions with good hash is 0.00000006

// Performance:
// get/put: O(1) average, O(log n) worst case (tree bucket)
// Null key: allowed (stored at bucket 0)
```

### LinkedHashMap
```java
// Extends HashMap + doubly-linked list through all entries
// Two modes:
// 1. Insertion order (default)
// 2. Access order (LRU): new LinkedHashMap<>(16, 0.75f, true)

// LRU Cache using LinkedHashMap:
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    LRUCache(int capacity) {
        super(capacity, 0.75f, true); // accessOrder = true
        this.capacity = capacity;
    }
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity; // auto-evict when over capacity
    }
}
```

### TreeMap
```java
// Red-Black tree (self-balancing BST)
// O(log n) for get/put/remove
// Keys sorted by natural order or Comparator
// NavigableMap methods: floorKey, ceilingKey, higherKey, lowerKey, subMap, headMap, tailMap

TreeMap<String, Integer> map = new TreeMap<>();
map.put("banana", 2); map.put("apple", 1); map.put("cherry", 3);
map.firstKey();               // "apple"
map.subMap("apple", "cherry"); // apple, banana (exclusive end)
```

### ConcurrentHashMap
```java
// Java 7: Segment-based locking (16 segments, each is a mini HashMap)
// Java 8+: CAS + synchronized on individual bucket head
//   - Reads: completely lock-free
//   - Writes: only lock the affected bucket
//   - No null keys or values allowed

ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// Atomic compound operations:
map.putIfAbsent("key", 1);
map.computeIfAbsent("key", k -> expensiveCompute(k));
map.merge("key", 1, Integer::sum); // atomic increment
map.compute("key", (k, v) -> v == null ? 1 : v + 1); // atomic update
```

### PriorityQueue
```java
// Min-heap by default (smallest element at head)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());

// O(log n) offer/poll, O(1) peek
// NOT thread-safe — use PriorityBlockingQueue for concurrent use

// Top-K pattern:
PriorityQueue<Integer> topK = new PriorityQueue<>(k); // min-heap of size k
for (int num : nums) {
    topK.offer(num);
    if (topK.size() > k) topK.poll(); // remove smallest
}
// topK now contains k largest elements
```

---

## 7. equals() and hashCode() Contract

```java
// THE CONTRACT:
// 1. a.equals(a) == true                    (reflexive)
// 2. a.equals(b) == b.equals(a)             (symmetric)
// 3. a.equals(b) && b.equals(c) → a.equals(c) (transitive)
// 4. a.equals(b) consistent across calls    (consistent)
// 5. a.equals(null) == false                (null-safe)
// 6. a.equals(b) → a.hashCode() == b.hashCode()  THE KEY RULE

// Correct implementation:
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof User)) return false; // handles null too
    User other = (User) o;
    return Objects.equals(email, other.email)
        && Objects.equals(id, other.id);
}

@Override
public int hashCode() {
    return Objects.hash(email, id); // same fields as equals
}
```

---

## 8. Functional Programming (Java 8+)

### Streams
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "Dave");

// Pipeline: source → intermediate ops (lazy) → terminal op (triggers execution)
List<String> result = names.stream()
    .filter(n -> n.length() > 3)      // intermediate — lazy
    .map(String::toUpperCase)          // intermediate — lazy
    .sorted()                          // intermediate — lazy
    .collect(Collectors.toList());     // terminal — triggers everything

// flatMap — flatten nested structures
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2), Arrays.asList(3, 4));
List<Integer> flat = nested.stream()
    .flatMap(Collection::stream)       // flatten
    .collect(Collectors.toList());     // [1, 2, 3, 4]

// Collectors
Map<Integer, List<String>> byLength = names.stream()
    .collect(Collectors.groupingBy(String::length));

Map<Boolean, List<String>> partitioned = names.stream()
    .collect(Collectors.partitioningBy(n -> n.length() > 3));

String joined = names.stream()
    .collect(Collectors.joining(", ", "[", "]")); // [Alice, Charlie, Dave]

// reduce
int sum = IntStream.rangeClosed(1, 100).sum(); // 5050
Optional<Integer> product = Stream.of(1,2,3,4).reduce((a, b) -> a * b);
```

### Optional
```java
// Purpose: explicit signal that value may be absent — no more NPE surprises
Optional<User> user = userRepo.findByEmail("nilesh@sap.com");

// BAD — don't use get() without check
user.get(); // throws NoSuchElementException if empty

// GOOD patterns:
user.orElse(defaultUser);                    // return default if empty
user.orElseGet(() -> createDefaultUser());   // lazy default
user.orElseThrow(() -> new UserNotFoundException("not found"));
user.ifPresent(u -> process(u));             // only if present
user.map(User::getName).orElse("Anonymous"); // transform if present
user.filter(u -> u.isActive()).ifPresent(this::notify);

// Anti-patterns:
// Optional as field type — use null instead
// Optional in collections — List<Optional<T>> is pointless
// Optional.get() without isPresent() — defeats the purpose
```

---

## 9. Exception Handling

```java
// Hierarchy:
// Throwable
//   ├── Error (don't catch: OutOfMemoryError, StackOverflowError)
//   └── Exception
//       ├── Checked (must handle: IOException, SQLException)
//       └── RuntimeException (unchecked: NPE, IllegalArgument, ArrayIndexOutOfBounds)

// Try-with-resources (Java 7) — auto-closes AutoCloseable
try (Connection conn = dataSource.getConnection();
     PreparedStatement ps = conn.prepareStatement(sql)) {
    ps.executeQuery();
} // conn and ps closed automatically, even if exception thrown

// Multi-catch (Java 7)
try {
    riskyOperation();
} catch (IOException | SQLException e) {
    log.error("IO or SQL error", e);
}

// Exception chaining — ALWAYS preserve original cause
try {
    Files.readAllBytes(path);
} catch (IOException e) {
    throw new ServiceException("Failed to read config", e); // chain it
}

// finally — always runs EXCEPT: System.exit(), JVM crash, infinite loop
try {
    return 1;
} finally {
    return 2; // overrides the return 1 — gotcha!
}
```

---

## 10. Sorting

### All Algorithms

| Algorithm | Best | Average | Worst | Space | Stable |
|---|---|---|---|---|---|
| Bubble | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Selection | O(n²) | O(n²) | O(n²) | O(1) | No |
| Insertion | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Tim | O(n) | O(n log n) | O(n log n) | O(n) | Yes |
| Counting | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes |

### QuickSort Internals
```java
void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pivot = partition(arr, low, high);
        quickSort(arr, low, pivot - 1);
        quickSort(arr, pivot + 1, high);
    }
}

int partition(int[] arr, int low, int high) {
    int pivot = arr[high]; // last element as pivot
    int i = low - 1;
    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            int temp = arr[i]; arr[i] = arr[j]; arr[j] = temp;
        }
    }
    int temp = arr[i+1]; arr[i+1] = arr[high]; arr[high] = temp;
    return i + 1;
}
// Worst case: already sorted + last pivot → O(n²)
// Fix: random pivot, or median-of-three
```

### Java's Sort Strategy
```java
Arrays.sort(int[])    // Dual-pivot QuickSort (Yaroslavskiy) — O(n log n) avg
Arrays.sort(Object[]) // TimSort — stable, O(n) if already sorted
Collections.sort()    // delegates to TimSort
// Why different? Objects need stability (equal elements keep order).
// Primitives don't box, so no stability need — QuickSort is faster.
```

### Comparator
```java
List<Person> people = ...;

// Comparator.comparing + chaining
people.sort(Comparator
    .comparing(Person::getLastName)
    .thenComparing(Person::getFirstName)
    .thenComparingInt(Person::getAge)
    .reversed());

// nullsFirst / nullsLast
Comparator<String> nullSafe = Comparator.nullsFirst(String::compareTo);
```

---

# PHASE 2 — JVM Internals

---

## 11. JVM Architecture

```
┌─────────────────────────────────────────────────┐
│                    JVM                           │
│  ┌──────────────┐  ┌──────────────────────────┐ │
│  │ Class Loader │  │   Runtime Data Areas      │ │
│  │  Subsystem   │  │  ┌────────┐ ┌──────────┐ │ │
│  └──────────────┘  │  │ Method │ │   Heap   │ │ │
│                    │  │  Area  │ │          │ │ │
│  ┌──────────────┐  │  └────────┘ └──────────┘ │ │
│  │  Execution   │  │  ┌────────┐ ┌──────────┐ │ │
│  │   Engine     │  │  │ Stack  │ │ PC Reg.  │ │ │
│  │ (JIT/Interp) │  │  │(thread)│ │(thread)  │ │ │
│  └──────────────┘  │  └────────┘ └──────────┘ │ │
└─────────────────────────────────────────────────┘
```

### JVM Memory Areas

**Metaspace (replaced PermGen in Java 8):**
- Class metadata, method bytecode, static variables
- In native memory — no fixed limit by default
- `-XX:MaxMetaspaceSize=256m` to cap it
- `OutOfMemoryError: Metaspace` if too many classes loaded (classloader leak)

**Heap:**
```
┌──────────────────────────────────────────────────┐
│                     HEAP                         │
│  ┌────────────────────────┐  ┌────────────────┐  │
│  │     Young Generation   │  │  Old Generation│  │
│  │  ┌──────┐ ┌──┐ ┌──┐  │  │   (Tenured)    │  │
│  │  │ Eden │ │S0│ │S1│  │  │                │  │
│  │  └──────┘ └──┘ └──┘  │  │                │  │
│  │   8   :   1  :  1    │  │                │  │
│  └────────────────────────┘  └────────────────┘  │
└──────────────────────────────────────────────────┘
```
- New objects → Eden
- Minor GC: Eden + S0 → S1 (or promote to Old if age > threshold)
- Objects surviving 15 GCs (default) → Old Generation
- TLAB (Thread Local Allocation Buffer): each thread gets a slice of Eden for fast local allocation

**Stack (per thread):**
- Stack frames: local variables, operand stack, return address
- `-Xss512k` to configure thread stack size
- `StackOverflowError` on deep recursion

---

## 12. Class Loading

### Three Phases
```
Loading → Linking (Verify + Prepare + Resolve) → Initialization

Loading:   Find .class file, read bytes, create Class object in heap
Verify:    Bytecode is valid, no security violations
Prepare:   Allocate memory for static fields, set to defaults (0, null, false)
Resolve:   Symbolic references → direct references
Initialize: Run static initializers and static field assignments
```

### Class Loader Hierarchy
```
Bootstrap ClassLoader (JVM built-in, C++)
    └── Platform ClassLoader (Java 9+, was Extension)
            └── Application ClassLoader (classpath)
                    └── Custom ClassLoader
```

**Parent delegation:** when asked to load a class, always ask parent first. Only loads it yourself if parent can't find it.
- **Why:** prevents rogue code from replacing `java.lang.String` with a malicious version.

```java
// Custom ClassLoader for hot reload / plugin isolation
public class PluginClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] bytes = loadBytesFromPlugin(name); // read from plugin jar
        return defineClass(name, bytes, 0, bytes.length);
    }
}
```

---

## 13. JIT Compilation

```
Execution path:
  Bytecode → Interpreter (slow) → C1 Compiler (fast compile, basic opt)
           → C2 Compiler (slow compile, heavy opt) for "hot" methods

Hot method: called > 10,000 times (CompileThreshold)
```

**Key JIT Optimizations:**
- **Method inlining:** replaces `helper()` call with helper's body — eliminates call overhead
- **Escape analysis:** if object doesn't escape current method, allocate on STACK not heap → no GC pressure
- **Loop unrolling:** expand loop body to reduce loop overhead
- **Dead code elimination:** remove code that can never execute
- **Scalar replacement:** decompose object into individual fields if it doesn't escape

```bash
# JVM flags for JIT insight:
-XX:+PrintCompilation          # see which methods get JIT compiled
-XX:+UnlockDiagnosticVMOptions
-XX:+PrintInlining             # see inlining decisions
```

---

## 14. Garbage Collection

### GC Roots
Objects reachable from GC roots are alive. Everything else is garbage.
- Local variables on stack
- Static fields
- JNI references
- Thread objects

### GC Algorithms Comparison

| GC | Pause | Throughput | Heap Size | Java Default |
|---|---|---|---|---|
| Serial | High (single-thread) | Low | Small (<4GB) | |
| Parallel | High (multi-thread) | High | Medium | Java 8 |
| G1 | Medium (predictable) | Good | Medium-Large | Java 9+ |
| ZGC | Sub-ms | Good | Any (up to TB) | Java 15+ option |
| Shenandoah | Sub-ms | Good | Any | Java 12+ option |

### G1GC (Default since Java 9)
```
Heap divided into ~2048 equal regions (1–32MB each)
Each region tagged as: Eden / Survivor / Old / Humongous

Collection cycle:
1. Young-only phase: minor GCs collect Eden + Survivor regions
2. Concurrent marking: runs alongside app, marks live objects in Old
3. Space reclamation: mixed GC — collects young + old regions with most garbage

Key flags:
-XX:MaxGCPauseMillis=200      # soft pause target (default 200ms)
-XX:G1HeapRegionSize=4m       # region size
-XX:InitiatingHeapOccupancyPercent=45  # start concurrent mark at 45% full
```

### ZGC (Java 15+ production)
```
Goals: pause < 1ms regardless of heap size
How: concurrent relocation — GC moves objects while app runs
     colored pointers — GC metadata in pointer bits
     load barriers — intercept reads to handle concurrent move

-XX:+UseZGC
-Xmx16g  # works fine with very large heaps
```

### GC Tuning Essentials
```bash
# Always set Xms == Xmx to avoid resize pauses
java -Xms4g -Xmx4g -XX:+UseG1GC MyApp

# GC logging (Java 9+)
-Xlog:gc*:file=gc.log:time,uptime:filecount=5,filesize=20m

# Common OOM types:
# OutOfMemoryError: Java heap space          → heap too small or memory leak
# OutOfMemoryError: Metaspace               → too many classes (classloader leak)
# OutOfMemoryError: GC overhead limit exceeded → 98% time in GC, <2% freed
```

---

## 15. Java Memory Model (JMM)

### The Visibility Problem
```java
// Thread A writes:
boolean ready = false;
int value = 0;

// Thread A:
value = 42;
ready = true;  // Thread B may see ready=true but value=0
               // because CPU can reorder writes

// Thread B:
while (!ready) {}
System.out.println(value); // might print 0!
```

### volatile — Visibility + Ordering
```java
volatile boolean ready = false; // no CPU caching, memory barrier

// volatile write happens-before volatile read
// guarantees: if Thread A writes volatile, Thread B reads it,
// B sees A's write AND all previous writes by A

// Does NOT guarantee atomicity:
volatile int count = 0;
count++; // still a race condition — read-modify-write not atomic
```

### Happens-Before Rules
1. Program order: earlier statement happens-before later in same thread
2. Monitor: `unlock` happens-before `lock` of same monitor
3. Volatile: `write` happens-before subsequent `read` of same variable
4. Thread start: `Thread.start()` happens-before any code in started thread
5. Thread join: all code in thread happens-before `Thread.join()` returns

---

# PHASE 3 — Concurrency

---

## 16. Thread Fundamentals

### Thread Lifecycle
```
NEW → RUNNABLE → BLOCKED (waiting for monitor lock)
              → WAITING (Object.wait, Thread.join, LockSupport.park)
              → TIMED_WAITING (sleep, wait(timeout), join(timeout))
              → TERMINATED
```

```java
// Creating threads — 4 ways:
// 1. Extend Thread
class MyThread extends Thread {
    @Override public void run() { doWork(); }
}
new MyThread().start();

// 2. Implement Runnable (preferred — no inheritance used up)
Thread t = new Thread(() -> doWork());
t.start();

// 3. Callable + Future (returns result)
ExecutorService exec = Executors.newFixedThreadPool(4);
Future<String> future = exec.submit(() -> computeResult());
String result = future.get(); // blocks until done

// 4. Virtual Thread (Java 21)
Thread vt = Thread.ofVirtual().start(() -> doWork());
```

### Thread Interruption
```java
// Cooperative cancellation — threads must check for interruption
Thread worker = new Thread(() -> {
    while (!Thread.currentThread().isInterrupted()) {
        doWork();
    }
});

// If thread is blocked (sleep/wait): InterruptedException is thrown
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt(); // ALWAYS re-interrupt!
    return;
}
```

---

## 17. Synchronization

### synchronized
```java
class Counter {
    private int count = 0;

    // Instance method lock — locks on 'this'
    public synchronized void increment() { count++; }

    // Static method lock — locks on Counter.class
    public static synchronized void staticMethod() { }

    // Block lock — more granular
    public void method() {
        synchronized (this) { count++; }
        // non-synchronized work here
    }
}
// synchronized is reentrant: same thread can acquire same lock multiple times
```

### ReentrantLock
```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // critical section
} finally {
    lock.unlock(); // ALWAYS in finally
}

// tryLock — non-blocking
if (lock.tryLock(100, TimeUnit.MILLISECONDS)) {
    try { /* work */ } finally { lock.unlock(); }
}

// Condition — like wait/notify but per-condition
Condition notEmpty = lock.newCondition();
Condition notFull = lock.newCondition();

// Producer:
lock.lock();
try {
    while (queue.isFull()) notFull.await();
    queue.add(item);
    notEmpty.signal();
} finally { lock.unlock(); }
```

### synchronized vs ReentrantLock

| | synchronized | ReentrantLock |
|---|---|---|
| Syntax | keyword | explicit lock/unlock |
| Fairness | No | Optional (fair=true) |
| Try lock | No | Yes (tryLock) |
| Interruptible | No | Yes (lockInterruptibly) |
| Multiple conditions | No (one per object) | Yes (newCondition) |
| Auto-release | Yes | No — must unlock in finally |

### ReadWriteLock
```java
ReadWriteLock rwLock = new ReentrantReadWriteLock();
// Multiple threads can hold read lock simultaneously
// Only one thread can hold write lock (exclusive)

rwLock.readLock().lock();
try { return data; } finally { rwLock.readLock().unlock(); }

rwLock.writeLock().lock();
try { data = newData; } finally { rwLock.writeLock().unlock(); }
// Use when: many reads, infrequent writes
```

---

## 18. Atomic Classes

```java
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();           // atomic i++, returns new value
count.getAndIncrement();           // atomic i++, returns old value
count.compareAndSet(expected, update); // CAS — only updates if current == expected
count.addAndGet(5);

// AtomicReference for objects
AtomicReference<Node> head = new AtomicReference<>(null);
head.compareAndSet(null, newNode); // lock-free linked list trick

// LongAdder — better than AtomicLong under high contention
// Distributes across cells (one per CPU core), sums on read
LongAdder counter = new LongAdder();
counter.increment();
long total = counter.sum();
```

---

## 19. ExecutorService

```java
// ThreadPoolExecutor — the core
ThreadPoolExecutor pool = new ThreadPoolExecutor(
    4,                    // corePoolSize
    8,                    // maximumPoolSize
    60, TimeUnit.SECONDS, // keepAliveTime for idle threads above core
    new LinkedBlockingQueue<>(100), // work queue
    new ThreadFactory() { ... },    // custom thread naming
    new ThreadPoolExecutor.CallerRunsPolicy() // rejection policy
);

// Common factories:
ExecutorService fixed   = Executors.newFixedThreadPool(4);        // bounded threads, unbounded queue
ExecutorService cached  = Executors.newCachedThreadPool();         // unbounded threads — CAREFUL
ExecutorService single  = Executors.newSingleThreadExecutor();     // sequential
ScheduledExecutorService sched = Executors.newScheduledThreadPool(2);

// Scheduled tasks:
sched.scheduleAtFixedRate(task, 0, 5, TimeUnit.SECONDS);   // every 5s from start
sched.scheduleWithFixedDelay(task, 0, 5, TimeUnit.SECONDS); // 5s after last completion

// Shutdown:
pool.shutdown();          // no new tasks, wait for running tasks
pool.shutdownNow();       // interrupt running tasks, return pending
pool.awaitTermination(30, TimeUnit.SECONDS); // wait for clean shutdown
```

### Rejection Policies

| Policy | What happens when queue full + max threads reached |
|---|---|
| AbortPolicy (default) | Throws RejectedExecutionException |
| CallerRunsPolicy | Caller thread runs the task (backpressure) |
| DiscardPolicy | Silently drops the task |
| DiscardOldestPolicy | Drops oldest queued task, retries |

---

## 20. CompletableFuture

```java
// Basic async computation
CompletableFuture<User> future = CompletableFuture
    .supplyAsync(() -> userRepo.findById(id), executor) // always provide executor
    .thenApply(user -> enrich(user))                   // sync transform
    .thenApplyAsync(user -> callExternalAPI(user), executor) // async transform
    .thenCompose(user -> fetchOrders(user))            // flatMap
    .exceptionally(ex -> {                             // error recovery
        log.error("Failed", ex);
        return defaultUser;
    });

// Combining multiple futures
CompletableFuture<User> userFuture = fetchUser(id);
CompletableFuture<List<Order>> ordersFuture = fetchOrders(id);

CompletableFuture<UserProfile> profile = userFuture
    .thenCombine(ordersFuture, (user, orders) -> new UserProfile(user, orders));

// Wait for all
CompletableFuture.allOf(f1, f2, f3).thenRun(() -> processAll());

// First to complete
CompletableFuture.anyOf(f1, f2, f3).thenAccept(result -> process(result));

// Timeout (Java 9)
future.orTimeout(5, TimeUnit.SECONDS)
      .completeOnTimeout(defaultValue, 5, TimeUnit.SECONDS);

// get() vs join():
// get()  — checked exception (ExecutionException, InterruptedException)
// join() — unchecked (CompletionException) — prefer in streams/lambdas
```

---

## 21. Virtual Threads (Java 21 — Project Loom)

```java
// Problem: 1 platform thread = 1 OS thread = ~1MB stack
// Solution: virtual threads = ~few KB, managed by JVM, not OS

// Creating virtual threads:
Thread.ofVirtual().start(() -> handleRequest()); // one per request — millions OK

// Virtual thread executor — ideal for server request handling
ExecutorService exec = Executors.newVirtualThreadPerTaskExecutor();
exec.submit(() -> handleRequest()); // creates virtual thread per task

// When virtual thread blocks (I/O, sleep):
// JVM unmounts it from carrier thread
// Carrier thread picks up another virtual thread
// When I/O completes, virtual thread is remounted
// → OS thread is never idle waiting

// Pinning — when virtual thread CANNOT unmount:
synchronized(lock) {  // synchronized block PINS the virtual thread
    blockingCall();   // carrier thread blocks too — bad!
}
// Fix: replace synchronized with ReentrantLock

// When to use virtual threads:
// ✓ High-concurrency I/O bound tasks (HTTP servers, DB calls)
// ✗ CPU-bound tasks (no blocking = no benefit from unmounting)
// ✗ Heavy use of synchronized blocks (pinning defeats the purpose)
```

---

## 22. Common Concurrency Problems

### Deadlock
```java
// Thread A holds lock1, waits for lock2
// Thread B holds lock2, waits for lock1
// Neither can proceed

// Prevention: always acquire locks in the same order
// Detection: jstack — look for "waiting to lock" cycle

// Example fix: lock ordering by ID
void transfer(Account from, Account to, double amount) {
    Account first  = from.getId() < to.getId() ? from : to;
    Account second = from.getId() < to.getId() ? to : from;
    synchronized (first) {
        synchronized (second) {
            from.debit(amount);
            to.credit(amount);
        }
    }
}
```

### ThreadLocal Memory Leak
```java
// Thread pools reuse threads — ThreadLocal values persist between tasks!
private static final ThreadLocal<Connection> conn = new ThreadLocal<>();

// In request handler:
conn.set(getConnection());
try {
    handleRequest();
} finally {
    conn.remove(); // ALWAYS remove — otherwise leaks into next request on same thread
}
```

---

# PHASE 4 — Spring Boot

---

## 23. IoC and Dependency Injection

### Why IoC
```java
// Without IoC — tight coupling
class OrderService {
    private PaymentService payment = new StripePaymentService(); // hardcoded!
    // Can't test without real Stripe, can't swap to PayPal
}

// With IoC — loose coupling
class OrderService {
    private final PaymentService payment;
    OrderService(PaymentService payment) { // injected from outside
        this.payment = payment;
    }
    // Test: inject MockPaymentService
    // Prod: inject StripePaymentService
}
```

### Injection Types
```java
// 1. Constructor injection (PREFERRED)
@Service
public class OrderService {
    private final PaymentService payment; // final — immutable

    public OrderService(PaymentService payment) { // Spring injects
        this.payment = payment;
    }
}

// 2. Setter injection (optional dependencies)
@Autowired(required = false)
public void setNotificationService(NotificationService ns) {
    this.notificationService = ns;
}

// 3. Field injection (AVOID — hidden dependencies, can't be final, hard to test)
@Autowired
private PaymentService payment; // bad — can't inject in unit tests without Spring
```

### @Configuration and @Bean
```java
@Configuration // CGLIB-proxied so @Bean methods return singletons
public class AppConfig {

    @Bean
    public DataSource dataSource() {
        return new HikariDataSource(hikariConfig());
    }

    @Bean
    public HikariConfig hikariConfig() { // called directly from dataSource()
        // Because @Configuration is CGLIB-proxied, this returns the SAME
        // HikariConfig bean — not a new instance each time
        return new HikariConfig();
    }
}
```

---

## 24. Bean Lifecycle

```
Constructor → @Autowired injection → @PostConstruct → [ready] → @PreDestroy
```

```java
@Component
public class CacheWarmer {

    @Autowired
    private UserRepository repo;

    @PostConstruct
    public void warmUp() {
        // repo is injected — safe to use here
        // called once after all dependencies are set
        cache.putAll(repo.findAllActive());
    }

    @PreDestroy
    public void cleanup() {
        // called when Spring context is closing
        cache.clear();
    }
}
```

### Full Order (when you need it)
```
1.  Constructor
2.  @Autowired field/setter injection
3.  setBeanName()    (BeanNameAware)
4.  setBeanFactory() (BeanFactoryAware)
5.  setApplicationContext() (ApplicationContextAware)
6.  BeanPostProcessor.postProcessBeforeInitialization()
7.  @PostConstruct
8.  InitializingBean.afterPropertiesSet()
9.  @Bean(initMethod="...")
10. BeanPostProcessor.postProcessAfterInitialization()  ← AOP proxy created here
    [BEAN IS READY]
11. @PreDestroy
12. DisposableBean.destroy()
13. @Bean(destroyMethod="...")
```

---

## 25. Bean Scopes

```java
@Component                   // singleton (default) — one per Spring context
@Scope("prototype")          // new instance every injection/getBean()
@Scope("request")            // one per HTTP request
@Scope("session")            // one per HTTP session

// Prototype-in-singleton trap:
@Component // singleton
public class OrderService {
    @Autowired
    private CartHelper cartHelper; // prototype — but injected ONCE at startup!
    // cartHelper is NEVER refreshed
}

// Fix 1: @Lookup
@Component
public abstract class OrderService {
    @Lookup
    public abstract CartHelper getCartHelper(); // Spring creates new instance each call
}

// Fix 2: ObjectProvider
@Component
public class OrderService {
    @Autowired
    private ObjectProvider<CartHelper> cartHelperProvider;

    public void process() {
        CartHelper helper = cartHelperProvider.getObject(); // new instance each time
    }
}
```

---

## 26. Spring AOP

### How It Works
```
Your class → BeanPostProcessor (AbstractAdvisorAutoProxyCreator)
           → creates CGLIB/JDK proxy wrapping your class
           → @Autowired gets the PROXY, not the original object
           → proxy intercepts method calls → runs advice → calls real method
```

```java
// Custom annotation + AOP for timing
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Timed { }

@Aspect
@Component
public class TimingAspect {

    @Around("@annotation(Timed)")
    public Object time(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            return pjp.proceed(); // call actual method
        } finally {
            long elapsed = System.currentTimeMillis() - start;
            log.info("{} took {}ms", pjp.getSignature().getName(), elapsed);
        }
    }
}

// Usage:
@Timed
public User findUser(Long id) { return repo.findById(id); }
```

### Self-Invocation Trap
```java
@Service
public class InvoiceService {

    public void processAll(List<Invoice> invoices) {
        for (Invoice inv : invoices) {
            this.process(inv); // calls on 'this' — NOT the proxy!
            // @Transactional on process() has NO effect here
        }
    }

    @Transactional
    public void process(Invoice inv) { ... }
}

// Fix: inject self
@Autowired
private InvoiceService self; // Spring injects the proxy

self.process(inv); // goes through proxy → @Transactional works
```

### JDK Proxy vs CGLIB

| | JDK Proxy | CGLIB |
|---|---|---|
| Requires interface | Yes | No |
| Creates | Implements interface | Subclasses target class |
| Limitation | Target must implement interface | Cannot proxy final class/method |
| Spring Boot default | No | Yes (proxy-target-class=true) |

---

## 27. @Transactional Deep Dive

### Propagation

| Propagation | Behaviour |
|---|---|
| REQUIRED (default) | Join existing or create new |
| REQUIRES_NEW | Always new, suspend existing |
| NESTED | Savepoint inside existing tx |
| MANDATORY | Must have existing, else exception |
| NEVER | Must have no tx, else exception |
| SUPPORTS | Use tx if exists, else no tx |
| NOT_SUPPORTED | Suspend existing, run without tx |

```java
// REQUIRES_NEW example — audit log must always be saved even if outer tx rolls back
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void saveAuditLog(AuditEntry entry) {
    auditRepo.save(entry); // committed independently
}

@Transactional
public void placeOrder(Order order) {
    orderRepo.save(order);
    auditService.saveAuditLog(new AuditEntry("order.placed")); // separate tx
    if (invalid) throw new RuntimeException(); // rolls back order, NOT audit log
}
```

### Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|---|---|---|---|
| READ_UNCOMMITTED | ✓ | ✓ | ✓ |
| READ_COMMITTED | ✗ | ✓ | ✓ |
| REPEATABLE_READ | ✗ | ✗ | ✓ |
| SERIALIZABLE | ✗ | ✗ | ✗ |

```java
@Transactional(
    isolation = Isolation.READ_COMMITTED,
    propagation = Propagation.REQUIRED,
    rollbackFor = Exception.class,  // default: only RuntimeException
    readOnly = true,                // optimization: no dirty checking, flush=NEVER
    timeout = 30                    // seconds
)
public UserProfile getProfile(Long userId) { ... }
```

### Common @Transactional Pitfalls
```java
// 1. Private method — proxy can't intercept
@Transactional
private void save() { } // NO EFFECT

// 2. Catching exception — Spring can't see it to rollback
@Transactional
public void process() {
    try {
        repo.save(entity);
    } catch (Exception e) {
        log.error("failed"); // swallowed! no rollback
    }
    // Fix: don't catch, or rethrow, or call TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()
}

// 3. @Transactional on @Async method — transactions don't span threads
@Async
@Transactional // each async task needs its own transaction — this is fine, but be aware
public CompletableFuture<Void> asyncProcess() { ... }
```

---

## 28. Spring Data JPA

```java
// Entity
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String email;

    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    private List<Order> orders = new ArrayList<>();

    @Version  // optimistic locking — prevents lost updates
    private Long version;
}

// Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // Method name queries:
    List<User> findByEmailContainingIgnoreCase(String email);
    Optional<User> findByEmailAndActiveTrue(String email);
    List<User> findByAgeGreaterThanOrderByNameAsc(int age);

    // JPQL:
    @Query("SELECT u FROM User u JOIN FETCH u.orders WHERE u.id = :id")
    Optional<User> findByIdWithOrders(@Param("id") Long id);

    // Native SQL:
    @Query(value = "SELECT * FROM users WHERE email = ?1", nativeQuery = true)
    Optional<User> findByEmailNative(String email);

    // Modifying query:
    @Modifying
    @Transactional
    @Query("UPDATE User u SET u.active = false WHERE u.lastLogin < :date")
    int deactivateOldUsers(@Param("date") LocalDate date);
}
```

### N+1 Problem
```java
// N+1: loading 100 users → 1 query for users + 100 queries for each user's orders
List<User> users = userRepo.findAll(); // 1 query
for (User u : users) {
    u.getOrders().size(); // 1 query per user = 100 extra queries
}

// Fix 1: JOIN FETCH in JPQL
@Query("SELECT u FROM User u JOIN FETCH u.orders")
List<User> findAllWithOrders();

// Fix 2: @EntityGraph
@EntityGraph(attributePaths = {"orders", "address"})
List<User> findAll();

// Fix 3: @BatchSize — loads in batches of N instead of 1 by 1
@OneToMany
@BatchSize(size = 50)
private List<Order> orders;
```

### Projections
```java
// Interface projection — Spring creates proxy, only fetches needed columns
public interface UserSummary {
    Long getId();
    String getEmail();
}
List<UserSummary> summaries = userRepo.findAllProjectedBy();

// DTO projection — constructor expression
@Query("SELECT new com.app.dto.UserDto(u.id, u.email) FROM User u")
List<UserDto> findAllAsDto();
```

---

## 29. Spring Security

### Security Filter Chain
```
HTTP Request
    → DelegatingFilterProxy
        → SecurityFilterChain (20+ filters in order):
            1. DisableEncodeUrlFilter
            2. WebAsyncManagerIntegrationFilter
            3. SecurityContextPersistenceFilter  ← loads SecurityContext
            4. HeaderWriterFilter
            5. CsrfFilter
            6. LogoutFilter
            7. UsernamePasswordAuthenticationFilter ← form login
            8. BasicAuthenticationFilter
            9. BearerTokenAuthenticationFilter  ← JWT
            ...
            15. ExceptionTranslationFilter       ← 401/403 handler
            16. AuthorizationFilter              ← final access decision
    → Controller
```

### JWT Authentication Filter
```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest req,
            HttpServletResponse res, FilterChain chain)
            throws ServletException, IOException {

        String header = req.getHeader("Authorization");
        if (header == null || !header.startsWith("Bearer ")) {
            chain.doFilter(req, res);
            return;
        }

        String token = header.substring(7);
        String username = jwtUtil.extractUsername(token);

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            if (jwtUtil.validateToken(token, userDetails)) {
                UsernamePasswordAuthenticationToken auth =
                    new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                auth.setDetails(new WebAuthenticationDetailsSource().buildDetails(req));
                SecurityContextHolder.getContext().setAuthentication(auth);
            }
        }
        chain.doFilter(req, res);
    }
}
```

### Security Config
```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())  // disable for stateless REST
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated())
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12); // cost factor 12
    }
}

// Method security
@PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
public UserProfile getProfile(Long userId) { ... }

@PostAuthorize("returnObject.owner == authentication.principal.username")
public Document getDocument(Long docId) { ... }
```

---

## 30. Spring Boot Auto-Configuration

```java
// How @SpringBootApplication works:
@SpringBootApplication
// = @Configuration + @EnableAutoConfiguration + @ComponentScan

// Auto-config mechanism:
// 1. @EnableAutoConfiguration triggers AutoConfigurationImportSelector
// 2. Reads META-INF/spring/org.springframework.boot.autoconfigure.EnableAutoConfiguration.factories
// 3. Loads 100+ AutoConfiguration classes
// 4. Each filtered by @Conditional annotations

// Example: DataSourceAutoConfiguration
@ConditionalOnClass(DataSource.class)        // only if JDBC on classpath
@ConditionalOnMissingBean(DataSource.class)  // only if YOU haven't defined one
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration { ... }

// Override: just define your own @Bean
@Bean
public DataSource dataSource() {
    HikariDataSource ds = new HikariDataSource();
    ds.setJdbcUrl(url);
    return ds; // your bean takes precedence — auto-config backs off
}
```

### application.yml patterns
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: ${DB_USER}          # env var injection
    password: ${DB_PASS:default}  # with default value
    hikari:
      maximum-pool-size: 10
      connection-timeout: 30000

  jpa:
    hibernate:
      ddl-auto: validate          # never use create/create-drop in prod
    show-sql: false               # disable in prod
    properties:
      hibernate:
        format_sql: true
        default_batch_fetch_size: 50  # fix N+1 globally

  profiles:
    active: ${SPRING_PROFILES_ACTIVE:dev}

# Type-safe config binding
@ConfigurationProperties(prefix = "app.payment")
@Validated
public class PaymentConfig {
    @NotNull private String apiKey;
    @Min(1) @Max(3) private int retryCount = 3;
    private Duration timeout = Duration.ofSeconds(5);
}
```

---

## 31. Actuator + Observability

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,metrics,info,loggers,threaddump
  endpoint:
    health:
      show-details: when-authorized
      probes:
        enabled: true  # /health/liveness and /health/readiness for K8s
  metrics:
    export:
      prometheus:
        enabled: true  # /actuator/prometheus for Prometheus scraping
```

```java
// Custom health indicator
@Component
public class ExternalServiceHealth implements HealthIndicator {
    @Override
    public Health health() {
        try {
            externalService.ping();
            return Health.up().withDetail("service", "reachable").build();
        } catch (Exception e) {
            return Health.down().withDetail("error", e.getMessage()).build();
        }
    }
}

// Custom metrics with Micrometer
@Component
public class OrderMetrics {
    private final Counter orderCounter;
    private final Timer orderTimer;

    public OrderMetrics(MeterRegistry registry) {
        orderCounter = Counter.builder("orders.created")
            .tag("type", "retail")
            .register(registry);
        orderTimer = Timer.builder("orders.processing.time")
            .register(registry);
    }

    public void recordOrder() { orderCounter.increment(); }
    public void recordProcessingTime(Runnable task) {
        orderTimer.record(task);
    }
}
```

---

## 32. Spring Boot Testing

```java
// Slice tests — fast, load only relevant layer
@WebMvcTest(UserController.class) // only web layer
class UserControllerTest {
    @Autowired MockMvc mockMvc;
    @MockBean UserService userService; // mock service layer

    @Test
    void getUser_returnsUser() throws Exception {
        when(userService.findById(1L)).thenReturn(new User("Nilesh"));

        mockMvc.perform(get("/api/users/1")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("Nilesh"));
    }
}

@DataJpaTest // only JPA layer, H2 in-memory, @Transactional rollback
class UserRepositoryTest {
    @Autowired UserRepository repo;

    @Test
    void findByEmail_returnsUser() {
        repo.save(new User("nilesh@sap.com"));
        assertThat(repo.findByEmail("nilesh@sap.com")).isPresent();
    }
}

// Integration test with TestContainers
@SpringBootTest
@Testcontainers
class OrderServiceIntegrationTest {
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
}
```

---

# PHASE 5 — Production Patterns

---

## 33. Resilience Patterns (Resilience4j)

```java
// Circuit Breaker — stops calling failing service
@CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
public PaymentResult processPayment(Payment payment) {
    return paymentClient.process(payment);
}

public PaymentResult paymentFallback(Payment payment, Exception ex) {
    log.warn("Payment service down, using fallback", ex);
    return PaymentResult.queued(payment.getId());
}
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        slidingWindowSize: 10          # last 10 calls
        failureRateThreshold: 50       # open if >50% fail
        waitDurationInOpenState: 30s   # wait before HALF_OPEN
        permittedNumberOfCallsInHalfOpenState: 3
  retry:
    instances:
      paymentService:
        maxAttempts: 3
        waitDuration: 500ms
        exponentialBackoffMultiplier: 2  # 500ms, 1s, 2s
        retryExceptions:
          - java.net.ConnectException
```

---

## 34. Caching

```java
@Configuration
@EnableCaching
public class CacheConfig {
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))
            .disableCachingNullValues()
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()));
        return RedisCacheManager.builder(factory)
            .cacheDefaults(config).build();
    }
}

@Service
public class UserService {

    @Cacheable(value = "users", key = "#id", unless = "#result == null")
    public User findById(Long id) {
        return repo.findById(id).orElse(null); // cached after first call
    }

    @CacheEvict(value = "users", key = "#user.id")
    public User update(User user) {
        return repo.save(user); // evicts stale cache entry
    }

    @CachePut(value = "users", key = "#result.id")
    public User create(User user) {
        return repo.save(user); // updates cache with new entry
    }
}
```

---

## 35. Async Processing

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean
    public Executor asyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(4);
        executor.setMaxPoolSize(8);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}

@Service
public class NotificationService {

    @Async("asyncExecutor")
    public CompletableFuture<Void> sendEmail(String to, String body) {
        emailClient.send(to, body); // runs in thread pool, not caller's thread
        return CompletableFuture.completedFuture(null);
    }
}
// Note: @Async has same self-invocation problem as @Transactional
// Must call through Spring proxy — not this.sendEmail()
```

---

## 36. Design Patterns in Java/Spring

### Singleton (Thread-Safe)
```java
// Best: Enum singleton — JVM guarantees single instance
public enum DatabasePool {
    INSTANCE;
    private final DataSource ds = createDataSource();
    public Connection getConnection() throws SQLException { return ds.getConnection(); }
}

// Alternative: Initialization-on-demand holder
public class Singleton {
    private Singleton() {}
    private static class Holder {
        static final Singleton INSTANCE = new Singleton();
    }
    public static Singleton getInstance() { return Holder.INSTANCE; }
}
```

### Builder
```java
// Java 14+ records for simple immutable objects
record User(Long id, String name, String email) {}

// Builder for complex objects (Lombok @Builder or manual)
public class HttpRequest {
    private final String url;
    private final String method;
    private final Map<String, String> headers;
    private final Duration timeout;

    private HttpRequest(Builder b) {
        this.url = b.url; this.method = b.method;
        this.headers = b.headers; this.timeout = b.timeout;
    }

    public static Builder builder(String url) { return new Builder(url); }

    public static class Builder {
        private final String url;
        private String method = "GET";
        private Map<String, String> headers = new HashMap<>();
        private Duration timeout = Duration.ofSeconds(30);

        private Builder(String url) { this.url = url; }
        public Builder method(String m) { this.method = m; return this; }
        public Builder header(String k, String v) { headers.put(k, v); return this; }
        public Builder timeout(Duration t) { this.timeout = t; return this; }
        public HttpRequest build() { return new HttpRequest(this); }
    }
}
// Usage: HttpRequest.builder("https://api.example.com").method("POST").build()
```

### Strategy
```java
// Open/Closed Principle — add new strategy without modifying existing code
interface PaymentStrategy {
    PaymentResult pay(double amount);
}

@Component("upi") class UpiPayment implements PaymentStrategy { ... }
@Component("card") class CardPayment implements PaymentStrategy { ... }
@Component("wallet") class WalletPayment implements PaymentStrategy { ... }

@Service
public class CheckoutService {
    @Autowired
    private Map<String, PaymentStrategy> strategies; // Spring injects all impls

    public PaymentResult checkout(String method, double amount) {
        return strategies.get(method).pay(amount);
    }
}
```

### Observer (Spring Events)
```java
// Event
public record UserCreatedEvent(User user) {}

// Publisher
@Service
public class UserService {
    @Autowired ApplicationEventPublisher eventPublisher;

    public User create(User user) {
        User saved = repo.save(user);
        eventPublisher.publishEvent(new UserCreatedEvent(saved));
        return saved;
    }
}

// Listener
@Component
public class WelcomeEmailListener {
    @EventListener
    @Async // handle in different thread
    public void onUserCreated(UserCreatedEvent event) {
        emailService.sendWelcome(event.user().getEmail());
    }
}
```

### Decorator (Spring AOP does this automatically)
```java
// Manual decorator — wraps functionality
interface DataService { List<Item> getData(); }

class CachedDataService implements DataService {
    private final DataService delegate;
    private List<Item> cache;

    CachedDataService(DataService delegate) { this.delegate = delegate; }

    @Override
    public List<Item> getData() {
        if (cache == null) cache = delegate.getData(); // fetch once, cache
        return cache;
    }
}
```

---

## 37. Java Records, Sealed Classes (Java 17)

```java
// Record — immutable data carrier, auto-generates: constructor, getters, equals, hashCode, toString
record Point(double x, double y) {
    // Compact constructor for validation
    Point {
        if (x < 0 || y < 0) throw new IllegalArgumentException("Negative coordinates");
    }
    // Can add methods
    double distanceTo(Point other) {
        return Math.sqrt(Math.pow(x - other.x, 2) + Math.pow(y - other.y, 2));
    }
}

// Sealed classes — restrict class hierarchy
sealed interface Shape permits Circle, Rectangle, Triangle {}
record Circle(double radius) implements Shape {}
record Rectangle(double width, double height) implements Shape {}
final class Triangle implements Shape { ... }

// Pattern matching switch (Java 21)
double area = switch (shape) {
    case Circle c    -> Math.PI * c.radius() * c.radius();
    case Rectangle r -> r.width() * r.height();
    case Triangle t  -> t.base() * t.height() / 2;
};
// Compiler ensures exhaustive matching — no default needed!
```

---

## 38. Quick Reference — Numbers Every Dev Should Know

| Metric | Value |
|---|---|
| L1 cache | 0.5 ns |
| L2 cache | 7 ns |
| RAM read | 100 ns |
| SSD random read | 150 µs |
| HDD seek | 10 ms |
| Network roundtrip (same DC) | 0.5 ms |
| Network CA → NL | 150 ms |
| 1M requests/day | ~12 QPS |
| 1B requests/day | ~12,000 QPS |
| 1 char in Java 8 String | 2 bytes |
| Default HashMap capacity | 16 |
| HashMap load factor | 0.75 |
| ArrayList initial capacity | 10 |
| ArrayList growth factor | ×1.5 |
| G1GC default pause target | 200ms |
| BCrypt recommended cost | 12 |
| JWT recommended expiry | 15 min (access), 7 days (refresh) |
| HikariCP default pool size | 10 |
| Thread stack size default | 512KB–1MB |
| Virtual thread stack | ~few KB |

---

*End of Java + Spring Boot Master Reference*
*Fresher: read top to bottom. SDE-3: use as lookup. Architect: challenge every "why" in here.*
