# Java + Spring Boot: Complete Interview Prep
### Story-style · End-to-End · Tricky Interview Questions + InterviewBit Depth
### Covers: Core Java → OOP → Collections → Strings → Sorting → Spring Boot → Beans → Context

---

## TL;DR

> This doc is written as a story — the way an interviewer takes you from one topic to another. Every answer is explained like you're explaining it to the interviewer out loud, not just reciting a definition. Topics covered: Java internals, String immutability proof, hashCode/equals contract, bean types, ApplicationContext, sorting deep dives, and Spring Boot internals.

---

## The Interview Story — How Questions Connect

```
Interviewer starts safe → Types of beans (scope warm-up)
  ↓ goes deeper → Can you PROVE String is immutable?
    ↓ pivots → hashCode vs equals — give me a real scenario
      ↓ moves to Spring → What does ApplicationContext hold?
        ↓ algorithm check → QuickSort — why pick last element as pivot?
          ↓ follows up → What's the worst case? Can you fix it?
```

This is the exact flow. Every question is a trap door into a deeper concept. Let's go through each one.

---

---

# PART 1 — Java Core

---

## 1.1 Types of Beans in Spring (The Warm-Up Trap)

### What the interviewer actually wants to know
Not just the names. They want to know **when each scope is dangerous** and whether you've been burned by it in production.

### The 5 Bean Scopes

| Scope | One instance per... | Typical use |
|---|---|---|
| `singleton` | Spring container (entire app) | Services, Repositories, Controllers |
| `prototype` | Each `getBean()` / injection | Stateful objects, per-use helpers |
| `request` | HTTP request | Request-scoped data holders |
| `session` | HTTP session | User session state |
| `application` | ServletContext lifetime | App-wide shared config |

### The Singleton Trap (What they really test)

```java
@Component  // singleton by default
public class UserService {
    private List<String> cache = new ArrayList<>(); // DANGER

    public void addToCache(String item) {
        cache.add(item);  // shared across ALL threads
    }
}
```

**Problem:** `cache` is an instance field on a singleton bean — shared across every request. Thread A and Thread B both write to the same list. Race condition.

**Fix:**
```java
// Option 1: Use ThreadLocal
private ThreadLocal<List<String>> cache = ThreadLocal.withInitial(ArrayList::new);

// Option 2: Make it prototype scope
@Scope("prototype")
@Component
public class UserService { ... }

// Option 3: Use stateless design (best)
// Pass data through method parameters, don't store in fields
```

### The Prototype-in-Singleton Trap (Interview favourite)

```java
@Component  // singleton
public class OrderService {

    @Autowired
    private CartHelper cartHelper;  // prototype bean — but injected ONCE at startup
    // cartHelper will NEVER be refreshed. You get the same instance every time.
}
```

**Fix — use `ApplicationContext.getBean()` manually, or `@Lookup`:**
```java
@Component
public abstract class OrderService {

    @Lookup
    public abstract CartHelper getCartHelper(); // Spring overrides this method each call

    public void processOrder() {
        CartHelper helper = getCartHelper(); // new instance each time
    }
}
```

### InterviewBit-style follow-ups

**Q: What is the default scope of a Spring bean?**
> `singleton`. One instance per Spring IoC container. NOT one instance per JVM — if you have two Spring contexts in the same JVM, each has its own singleton.

**Q: Can a singleton bean have state?**
> It can, but it's dangerous in multi-threaded environments. Stateless singletons are best practice. If you need state, use `prototype`, `request`, or `ThreadLocal`.

**Q: What happens when a prototype bean is injected into a singleton?**
> The prototype bean is instantiated once at injection time and never refreshed. Effectively becomes a singleton. Use `@Lookup` or `ObjectFactory<T>` to get a new instance each time.

---

## 1.2 Can You PROVE String is Immutable?

### What "immutable" actually means
An object is immutable if its state **cannot change after construction**. For String this means: once `new String("hello")` is created, the characters inside can never be altered by any method.

### The 4 Pillars of String Immutability

#### Pillar 1: `final` class — no subclassing
```java
public final class String { ... }
```
If String weren't `final`, you could do this:
```java
public class HackedString extends String {
    @Override
    public String concat(String s) {
        // mutate internal state here
    }
}
```
`final` prevents this. Nobody can override String's behaviour.

#### Pillar 2: `private final char[]` (or `byte[]` in Java 9+)
```java
public final class String {
    private final char[] value;  // Java 8 and below
    // private final byte[] value; // Java 9+ (compact strings)
}
```
`private` — no outside class can access `value` directly.
`final` — the reference `value` cannot point to a different array after construction.

**But wait — `final` on an array only makes the reference final, not the array contents!**

```java
final char[] arr = {'h', 'e', 'l', 'l', 'o'};
arr[0] = 'X';  // LEGAL — final on the reference, not the contents
```

So why is String still immutable? Because:

#### Pillar 3: No method exposes the internal array
```java
// String NEVER does this:
public char[] getValue() { return value; } // would allow mutation

// String does this instead:
public char[] toCharArray() {
    return Arrays.copyOf(value, value.length); // defensive copy
}
```
Every method that "returns" the data returns a **copy**, never the original array.

#### Pillar 4: All "modification" methods return NEW String objects
```java
String s = "hello";
String upper = s.toUpperCase();  // returns NEW String "HELLO"
// s is still "hello" — unchanged
System.out.println(s);     // hello
System.out.println(upper); // HELLO
```

### The Live Proof — Using Reflection
```java
String s = "hello";
System.out.println(s); // hello

// Access private field via reflection
Field field = String.class.getDeclaredField("value");
field.setAccessible(true);
char[] value = (char[]) field.get(s);
value[0] = 'X'; // mutate the underlying array

System.out.println(s); // Xello — the string changed!
// But this is a hack — normal Java code cannot do this
```
This PROVES immutability is enforced at the API level (no public way to mutate), but not at the JVM memory level.

### Why Does Immutability Matter?

**1. String Pool (interning) works because of immutability:**
```java
String a = "hello";
String b = "hello";
System.out.println(a == b); // true — same object from pool

// If Strings were mutable:
a.setChar(0, 'X'); // would change b too — disaster
```

**2. Thread safety — Strings can be shared across threads without synchronization**

**3. HashMap keys are safe:**
```java
String key = "user123";
map.put(key, user);
// Even if key reference changes elsewhere, the hashCode in the map never changes
```

### InterviewBit follow-ups

**Q: What is String interning?**
> `String.intern()` puts the string into the String Pool and returns the pooled reference. All string literals are automatically interned. `new String("hello")` creates a new heap object — NOT from the pool.

```java
String a = "hello";
String b = new String("hello");
String c = b.intern();

System.out.println(a == b); // false — b is on heap
System.out.println(a == c); // true — c is the pooled reference
```

**Q: Why is String used as HashMap key?**
> Because its `hashCode()` is cached after first computation (stored in a private `hash` field), and it never changes since the string is immutable. Fast and safe.

**Q: StringBuilder vs StringBuffer vs String**

| | Thread-safe | Mutable | Use when |
|---|---|---|---|
| `String` | Yes (immutable) | No | Default, small concatenation |
| `StringBuilder` | No | Yes | Single-threaded, heavy concatenation |
| `StringBuffer` | Yes (synchronized) | Yes | Multi-threaded string building |

---

## 1.3 hashCode vs equals — The Contract

### What the interviewer really asks
"Give me a scenario where breaking this contract causes a real bug."

### The Contract (must memorize word for word)

1. If `a.equals(b)` is `true` → `a.hashCode() == b.hashCode()` **must** be true
2. If `a.hashCode() == b.hashCode()` → `a.equals(b)` **may or may not** be true (hash collision is ok)
3. If `a.equals(b)` is `false` → hashCodes **may or may not** be equal

In short: **equal objects must have equal hashCodes. The reverse is not required.**

### The Bug — Override equals without hashCode

```java
public class User {
    private String email;

    public User(String email) { this.email = email; }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof User)) return false;
        return this.email.equals(((User) o).email);
    }
    // hashCode NOT overridden — uses Object's default (memory address)
}
```

```java
User u1 = new User("nilesh@sap.com");
User u2 = new User("nilesh@sap.com");

System.out.println(u1.equals(u2)); // true — same email
System.out.println(u1.hashCode() == u2.hashCode()); // FALSE — different memory address

// Now put in HashMap:
Map<User, String> map = new HashMap<>();
map.put(u1, "engineer");
System.out.println(map.get(u2)); // NULL — even though u1.equals(u2) is true!
```

**Why?** HashMap first checks `hashCode()` to find the bucket. `u1` and `u2` have different hashCodes → go to different buckets → `get(u2)` looks in the wrong bucket → returns null.

### The Fix

```java
@Override
public int hashCode() {
    return Objects.hash(email); // same email → same hashCode
}
```

### How HashMap Uses hashCode + equals Internally

```
HashMap.put(key, value):
  1. Compute key.hashCode()
  2. Apply bit mixing: (h = key.hashCode()) ^ (h >>> 16)
  3. bucket index = hash & (capacity - 1)
  4. Go to that bucket
  5. If bucket empty → insert node
  6. If bucket not empty → walk the chain, call equals() on each node
     → if equals() true → update value
     → if equals() false for all → append new node
     → if chain length > 8 → convert to TreeMap (Red-Black tree) [Java 8+]
```

### The equals() Contract (reflexive, symmetric, transitive, consistent, null-safe)

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;           // reflexive
    if (!(o instanceof User)) return false; // null-safe + type check
    User other = (User) o;
    return Objects.equals(this.email, other.email); // symmetric
}
```

**Never use `==` to compare objects** (compares references). Always use `.equals()`.

### InterviewBit follow-ups

**Q: What is the default hashCode implementation?**
> `Object.hashCode()` returns the memory address (converted to int). Two different objects always get different hashCodes even if they're logically equal.

**Q: Can two different objects have the same hashCode?**
> Yes — this is a hash collision. int has 2^32 possible values but infinite possible objects. HashMap handles collisions by chaining (linked list → tree in Java 8+).

**Q: What happens if hashCode always returns the same value (e.g., `return 1`)?**
> Technically valid — never breaks the contract. But all objects land in the same bucket → HashMap degrades to O(n) linked list traversal. Never do this in production.

**Q: Why does String cache its hashCode?**
```java
public final class String {
    private int hash; // cached hashCode, 0 means not computed yet

    public int hashCode() {
        if (hash == 0 && value.length > 0) {
            hash = computeHash(); // computed once, stored
        }
        return hash;
    }
}
```
> Because Strings are immutable the hashCode never changes, so computing it once and caching it is safe and fast.

---

## 1.4 Java Memory Model — Stack vs Heap

### What lives where

```
Stack (per thread):
  - Primitive variables (int, boolean, char, etc.)
  - Object references (the pointer, not the object itself)
  - Method call frames

Heap (shared across all threads):
  - All objects created with `new`
  - String pool (part of heap since Java 7+)
  - Static variables (in Metaspace since Java 8+)

Metaspace (was PermGen before Java 8):
  - Class definitions
  - Method bytecode
  - Static fields
```

```java
void method() {
    int x = 5;              // stack: primitive
    String s = "hello";     // stack: reference 's', heap: String object
    User u = new User();    // stack: reference 'u', heap: User object
}
// When method() exits: x, s, u popped from stack
// The actual objects on heap remain until GC collects them (if no more references)
```

### Why this matters for interviews

**Q: Where is `static` stored?**
> Static fields are stored in Metaspace (class metadata area). One copy shared across all instances.

**Q: Is String pool on stack or heap?**
> Heap. Since Java 7, String pool moved from PermGen to heap so it's subject to garbage collection.

---

## 1.5 `final`, `finally`, `finalize` — The Classic Trio

| Keyword | What it does |
|---|---|
| `final` on variable | Reference cannot change (primitives: value cannot change) |
| `final` on method | Cannot be overridden in subclasses |
| `final` on class | Cannot be subclassed (e.g., `String`, `Integer`) |
| `finally` | Block always executes after try/catch — for cleanup |
| `finalize()` | Called by GC before object is collected — **deprecated in Java 9, removed in Java 18** |

```java
// final variable
final int MAX = 100;
MAX = 200; // compile error

// final reference — reference is fixed, object is not
final List<String> list = new ArrayList<>();
list.add("hello"); // OK — modifying the object, not the reference
list = new ArrayList<>(); // compile error — changing the reference

// finally
try {
    return result;
} finally {
    connection.close(); // ALWAYS runs, even if return or exception happens
}
```

---

## 1.6 Generics + Type Erasure

### What type erasure means
Java generics exist only at compile time. At runtime, the JVM sees raw types — all `<T>` are replaced with `Object` (or the upper bound).

```java
List<String> strings = new ArrayList<>();
List<Integer> ints = new ArrayList<>();

// At runtime:
strings.getClass() == ints.getClass(); // true — both are just ArrayList.class
```

### Why this causes problems

```java
// You cannot do this:
public <T> void method(List<T> list) {
    T t = new T(); // ERROR — type unknown at runtime, can't instantiate
    if (list instanceof List<String>) {} // ERROR — can't check generic type at runtime
}

// You cannot create generic arrays:
T[] arr = new T[10]; // ERROR
```

### Wildcards

```java
// ? extends T — read-only (covariant, upper bounded)
List<? extends Number> nums = new ArrayList<Integer>();
Number n = nums.get(0); // OK
nums.add(1); // ERROR — don't know exact type

// ? super T — write-only (contravariant, lower bounded)
List<? super Integer> nums2 = new ArrayList<Number>();
nums2.add(1); // OK
Number n2 = nums2.get(0); // ERROR — returns Object, not Number
```

**Mnemonic: PECS — Producer Extends, Consumer Super**
- If you're reading from the list (producing values) → `extends`
- If you're writing to the list (consuming values) → `super`

---

---

# PART 2 — Collections Deep Dive

---

## 2.1 ArrayList vs LinkedList vs Vector

| | ArrayList | LinkedList | Vector |
|---|---|---|---|
| Backed by | Dynamic array | Doubly linked list | Dynamic array |
| Random access | O(1) | O(n) | O(1) |
| Insert at end | O(1) amortized | O(1) | O(1) amortized |
| Insert at middle | O(n) | O(1) if you have node | O(n) |
| Thread safe | No | No | Yes (synchronized) |
| Use when | Most cases | Frequent insert/delete at ends | Legacy — don't use |

### ArrayList resize internals
```
Initial capacity: 10
When full: new capacity = old * 1.5 (Java 8: oldCapacity + (oldCapacity >> 1))
Creates new array, copies all elements — O(n) operation
This is why amortized O(1) for add, but occasionally O(n)
```

---

## 2.2 HashMap vs LinkedHashMap vs TreeMap vs ConcurrentHashMap

| | Ordering | Thread-safe | Null key | Performance |
|---|---|---|---|---|
| `HashMap` | None | No | 1 null key | O(1) avg |
| `LinkedHashMap` | Insertion order | No | 1 null key | O(1) avg |
| `TreeMap` | Sorted (natural or Comparator) | No | No | O(log n) |
| `ConcurrentHashMap` | None | Yes | No | O(1) avg |
| `Hashtable` | None | Yes (fully) | No | O(1) — legacy |

### ConcurrentHashMap internals
- Java 7: Segment-based locking (16 segments by default)
- Java 8+: CAS (Compare-And-Swap) + synchronized on individual bucket head
- Reads: lock-free always
- Writes: only lock the affected bucket, not the entire map

```java
// Safe multi-threaded counter
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.merge("key", 1, Integer::sum); // atomic increment
```

---

## 2.3 HashSet Internals

```java
// HashSet IS a HashMap internally
public class HashSet<E> {
    private transient HashMap<E, Object> map;
    private static final Object PRESENT = new Object(); // dummy value

    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }
}
```

Every element is a key in the internal HashMap. The value is always the same dummy `PRESENT` object. All HashMap rules apply — including the hashCode + equals contract.

---

---

# PART 3 — Sorting Deep Dive

---

## 3.1 QuickSort — Why Last Element as Pivot?

### The question she asked you
> "In QuickSort, why do you choose the last element as pivot? You said 'maximum switch can be done' — explain that."

### The real answer

Choosing the **last element as pivot** is the **simplest textbook implementation** — not the optimal one. Here's what's actually true:

**Reason it's taught this way:**
- Simplest code to write and teach
- Partition logic is clean — you scan from left, swap anything smaller than pivot to the left partition, pivot lands in its final sorted position

```java
int partition(int[] arr, int low, int high) {
    int pivot = arr[high]; // last element
    int i = low - 1;       // index of smaller element

    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            // swap arr[i] and arr[j]
            int temp = arr[i]; arr[i] = arr[j]; arr[j] = temp;
        }
    }
    // place pivot in correct position
    int temp = arr[i+1]; arr[i+1] = arr[high]; arr[high] = temp;
    return i + 1; // pivot index
}
```

**Why the "maximum swaps" answer is partially right:**
The partition puts all elements smaller than pivot to the left and all larger to the right — every swap brings one element closer to its final sorted position. This is why it's efficient on average.

### QuickSort Complexity

| Case | When | Time |
|---|---|---|
| Best | Pivot always lands in the middle | O(n log n) |
| Average | Random input | O(n log n) |
| Worst | Already sorted array + last element pivot | O(n²) |

### Why Worst Case Happens with Last Element Pivot

```
Array: [1, 2, 3, 4, 5]  ← already sorted
Pivot = 5 (last)
Partition: [1, 2, 3, 4] | [5]
Next pivot = 4 (last of left partition)
Partition: [1, 2, 3] | [4]
...
```

Every partition creates one partition of size n-1 and one of size 0. Recursion depth = n. O(n²).

### How to Fix It — 3 Strategies

**Strategy 1: Random pivot**
```java
int randomPivot = low + (int)(Math.random() * (high - low + 1));
// swap arr[randomPivot] with arr[high], then proceed as normal
```
Eliminates worst case for sorted arrays. Expected O(n log n).

**Strategy 2: Median-of-three**
```java
int mid = low + (high - low) / 2;
// pick median of arr[low], arr[mid], arr[high] as pivot
// swap that median to arr[high], proceed as normal
```
More consistent pivot selection. Used by many production implementations.

**Strategy 3: Use Java's `Arrays.sort()` — Dual-Pivot QuickSort (Java 7+)**
```java
// Java's Arrays.sort() uses:
// - Dual-pivot QuickSort for primitives (Yaroslavskiy algorithm)
// - TimSort for objects (merge sort + insertion sort hybrid)

Arrays.sort(arr); // primitives → dual-pivot quicksort
Arrays.sort(objArr); // objects → TimSort (stable)
```

Dual-pivot uses TWO pivots, creating 3 partitions instead of 2. Faster in practice.

### InterviewBit follow-ups

**Q: Is QuickSort stable?**
> No. Elements with equal values may swap their relative order. If stability matters, use MergeSort or TimSort.

**Q: QuickSort vs MergeSort — when to use which?**

| | QuickSort | MergeSort |
|---|---|---|
| Stability | Unstable | Stable |
| Space | O(log n) stack | O(n) extra array |
| Cache | Cache-friendly (in-place) | Cache-unfriendly |
| Worst case | O(n²) | O(n log n) guaranteed |
| Use for | Primitives, in-memory | Objects, linked lists, external sort |

**Q: What is TimSort?**
> Hybrid of MergeSort + InsertionSort. Exploits naturally occurring sorted runs in real data. O(n) best case (already sorted), O(n log n) worst. Used by Java (objects), Python, Android.

**Q: What sorting algorithm for nearly sorted data?**
> InsertionSort — O(n) for nearly sorted. Or TimSort which detects runs.

---

## 3.2 Other Sorting Algorithms Cheat Sheet

| Algorithm | Best | Average | Worst | Space | Stable |
|---|---|---|---|---|---|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes |
| TimSort | O(n) | O(n log n) | O(n log n) | O(n) | Yes |

---

---

# PART 4 — Spring Boot Internals

---

## 4.1 ApplicationContext — What Does It Actually Hold?

### What the interviewer wants
Not "it's the Spring container". They want to know **what's inside** and how it works.

### ApplicationContext = BeanFactory + Much More

```
BeanFactory (basic container):
  - Bean definitions
  - Bean instantiation
  - Dependency injection
  - Lazy loading by default

ApplicationContext (full container) adds:
  - Eager singleton instantiation at startup
  - Event publishing (ApplicationEventPublisher)
  - i18n / MessageSource
  - Resource loading (classpath, file, URL)
  - Environment and PropertySource abstraction
  - AOP integration
  - @Component scanning
```

### What ApplicationContext physically holds

```java
ApplicationContext ctx = SpringApplication.run(App.class, args);

// 1. All bean definitions
String[] beans = ctx.getBeanDefinitionNames();

// 2. The Environment (all properties)
Environment env = ctx.getEnvironment();
String port = env.getProperty("server.port");

// 3. Resource loader
Resource res = ctx.getResource("classpath:config.json");

// 4. Event publisher
ctx.publishEvent(new UserCreatedEvent(this, user));

// 5. Parent context (if exists — e.g., parent = root context, child = web context)
ApplicationContext parent = ctx.getParent();
```

### Spring Boot Startup Sequence (full story)

```
1. main() → SpringApplication.run()
2. Create ApplicationContext (AnnotationConfigServletWebServerApplicationContext for web)
3. Load and parse all @Configuration classes
4. Perform @ComponentScan — find all @Component, @Service, @Repository, @Controller
5. Process @EnableAutoConfiguration → load spring.factories (META-INF/spring/org.springframework.boot.autoconfigure.EnableAutoConfiguration.factories)
6. Build BeanDefinition registry — register all bean definitions (not instances yet)
7. BeanFactoryPostProcessor runs — can modify bean definitions (e.g., PropertyPlaceholderConfigurer)
8. Instantiate all singleton beans (eagerly) — resolve @Autowired dependencies
9. BeanPostProcessor runs — wraps beans with proxies (AOP, @Transactional, @Async)
10. ApplicationContext refreshed — ready
11. CommandLineRunner / ApplicationRunner beans execute
12. Application is live
```

### The Three Types of ApplicationContext

| Type | Use case |
|---|---|
| `AnnotationConfigApplicationContext` | Pure Java, no web (tests, CLI) |
| `AnnotationConfigServletWebServerApplicationContext` | Spring Boot web app (default) |
| `AnnotationConfigReactiveWebServerApplicationContext` | Spring WebFlux reactive app |

### InterviewBit follow-ups

**Q: What is the difference between BeanFactory and ApplicationContext?**
> BeanFactory is the base container — lazy loads beans, minimal features. ApplicationContext extends it — eager loads singletons at startup, adds events, i18n, resource loading. In production, always use ApplicationContext.

**Q: What is a BeanDefinition?**
> The metadata Spring stores about a bean before creating it: class name, scope, constructor args, property values, init/destroy methods, lazy flag. Like a blueprint — the actual bean instance is created later.

**Q: What is BeanFactoryPostProcessor vs BeanPostProcessor?**

```
BeanFactoryPostProcessor:
  - Runs BEFORE beans are instantiated
  - Can modify BeanDefinitions
  - Example: PropertySourcesPlaceholderConfigurer (resolves ${property.name})

BeanPostProcessor:
  - Runs AFTER each bean is instantiated
  - Can wrap beans with proxies
  - Example: AutowiredAnnotationBeanPostProcessor (processes @Autowired)
  - Example: AbstractAdvisorAutoProxyCreator (creates AOP proxies for @Transactional)
```

**Q: What is `@DependsOn`?**
> Forces Spring to initialize one bean before another, even if no direct `@Autowired` dependency exists. Useful when beans have implicit ordering requirements.

---

## 4.2 Bean Lifecycle — The Full Journey

```
1. Instantiation — constructor called
2. Populate properties — @Autowired dependencies injected
3. BeanNameAware.setBeanName() — if implemented
4. BeanFactoryAware.setBeanFactory() — if implemented
5. ApplicationContextAware.setApplicationContext() — if implemented
6. BeanPostProcessor.postProcessBeforeInitialization()
7. @PostConstruct method — if present
8. InitializingBean.afterPropertiesSet() — if implemented
9. Custom init-method (init-method="...") — if configured
10. BeanPostProcessor.postProcessAfterInitialization()
    → This is where AOP proxies are created (wraps the bean)
    → What @Autowired gets is the PROXY, not the original bean

--- BEAN IS READY AND IN USE ---

11. @PreDestroy — called when context is closing
12. DisposableBean.destroy()
13. Custom destroy-method
```

### The @PostConstruct Use Case

```java
@Component
public class CacheLoader {

    @Autowired
    private UserRepository repo;

    @PostConstruct
    public void loadCache() {
        // repo is already injected here
        // called once after construction — perfect for warming cache
        cache = repo.findAllActive();
    }
}
```

---

## 4.3 @Transactional Deep Dive

### How it works — AOP proxy

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder(Order order) {
        // Spring wraps this class with a CGLIB proxy
        // The proxy:
        //   1. Opens transaction before method
        //   2. Calls the real method
        //   3. Commits on success
        //   4. Rolls back on RuntimeException
    }
}
```

### The Self-Invocation Trap (very common interview question)

```java
@Service
public class OrderService {

    public void processOrders(List<Order> orders) {
        for (Order o : orders) {
            this.placeOrder(o); // calls method on SAME object — bypasses proxy!
        }
    }

    @Transactional
    public void placeOrder(Order order) {
        // @Transactional has NO effect here when called from processOrders()
        // because the call goes directly to the real object, not the proxy
    }
}
```

**Fix:**
```java
// Option 1: Inject self
@Autowired
private OrderService self; // Spring injects the proxy

self.placeOrder(o); // goes through proxy → @Transactional works

// Option 2: Use ApplicationContext.getBean()
// Option 3: Refactor into a separate bean
```

### Propagation Types

| Propagation | Behaviour |
|---|---|
| `REQUIRED` (default) | Join existing tx or create new |
| `REQUIRES_NEW` | Always create new tx, suspend existing |
| `NESTED` | Nested tx inside existing (savepoint) |
| `MANDATORY` | Must have existing tx, else exception |
| `NEVER` | Must have no tx, else exception |
| `SUPPORTS` | Use tx if exists, else no tx |
| `NOT_SUPPORTED` | Suspend existing tx, run without tx |

### Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|---|---|---|---|
| `READ_UNCOMMITTED` | Possible | Possible | Possible |
| `READ_COMMITTED` (default PG) | Prevented | Possible | Possible |
| `REPEATABLE_READ` (default MySQL) | Prevented | Prevented | Possible |
| `SERIALIZABLE` | Prevented | Prevented | Prevented |

---

## 4.4 Spring AOP — JDK Proxy vs CGLIB

```
JDK Dynamic Proxy:
  - Works only if bean implements an interface
  - Creates a proxy that implements the same interface
  - Uses java.lang.reflect.Proxy

CGLIB Proxy:
  - Works on classes (no interface needed)
  - Creates a subclass of the target class at runtime
  - Cannot proxy final classes or final methods
  - Default in Spring Boot (spring.aop.proxy-target-class=true)
```

```java
// JDK proxy situation:
public interface UserService { void createUser(); }
public class UserServiceImpl implements UserService { ... }
// Spring injects JDK proxy (implements UserService)

// CGLIB proxy situation:
@Service
public class UserService { // no interface
    @Transactional
    public void createUser() { ... }
}
// Spring creates CGLIB subclass proxy
```

---

## 4.5 @SpringBootApplication — What It Really Is

```java
@SpringBootApplication
// is exactly equivalent to:
@Configuration         // This class is a source of bean definitions
@EnableAutoConfiguration // Enable auto-config based on classpath
@ComponentScan         // Scan current package and subpackages for @Component
public class App { ... }
```

### Auto-configuration Mechanism

```
@EnableAutoConfiguration triggers:
  1. Spring reads META-INF/spring/org.springframework.boot.autoconfigure.EnableAutoConfiguration.factories
  2. This file lists 100+ auto-configuration classes (DataSourceAutoConfiguration, JpaAutoConfiguration, etc.)
  3. Each class is annotated with @ConditionalOnClass, @ConditionalOnMissingBean, etc.
  4. Only configs whose conditions pass are applied

Example:
  DataSourceAutoConfiguration:
    @ConditionalOnClass(DataSource.class)  // only if JDBC jar is on classpath
    @ConditionalOnMissingBean(DataSource.class)  // only if you haven't defined one yourself
```

---

---

# PART 5 — OOP + Design Patterns (Interview Style)

---

## 5.1 The 4 Pillars — With Real Java Examples

### Encapsulation
Hide internal state. Expose only what's needed.
```java
public class BankAccount {
    private double balance; // hidden

    public void deposit(double amount) {
        if (amount > 0) balance += amount; // controlled access
    }
    public double getBalance() { return balance; } // read-only
}
```

### Inheritance
```java
public class Animal { public void eat() { System.out.println("eating"); } }
public class Dog extends Animal {
    @Override
    public void eat() { System.out.println("eating dog food"); }
    public void bark() { System.out.println("woof"); }
}
```

### Polymorphism
Same method, different behaviour based on runtime type.
```java
Animal a = new Dog(); // runtime type is Dog
a.eat(); // calls Dog.eat() — runtime polymorphism (dynamic dispatch)
```

### Abstraction
Hide complexity. Expose essential interface.
```java
List<String> list = new ArrayList<>(); // using List interface
// Don't care about ArrayList internals — just use the List contract
```

---

## 5.2 Abstract Class vs Interface

| | Abstract Class | Interface |
|---|---|---|
| Instantiate | No | No |
| Constructor | Yes | No |
| Fields | Any (state allowed) | `public static final` only |
| Methods | Any (abstract + concrete) | Default + abstract (Java 8+) |
| Extends/implements | Single | Multiple |
| Use when | Shared base with state | Contract definition |

```java
// Abstract class — IS-A relationship with shared state
abstract class Shape {
    protected String color; // shared state
    abstract double area();
    public String getColor() { return color; } // concrete method
}

// Interface — CAN-DO relationship
interface Serializable { void serialize(); }
interface Drawable { void draw(); }

class Circle extends Shape implements Serializable, Drawable { ... }
```

---

## 5.3 SOLID in One Page

**S — Single Responsibility**
```java
// BAD: UserService saves users AND sends emails
class UserService {
    void saveUser(User u) { ... }
    void sendWelcomeEmail(User u) { ... } // separate concern
}

// GOOD: split
class UserService { void saveUser(User u) { ... } }
class EmailService { void sendWelcomeEmail(User u) { ... } }
```

**O — Open/Closed**
```java
// Open for extension, closed for modification
// BAD: add new payment → modify existing class
class PaymentProcessor {
    void process(String type) {
        if (type.equals("UPI")) { ... }
        else if (type.equals("CARD")) { ... } // add new type = modify this
    }
}

// GOOD: add new payment → add new class, don't modify existing
interface PaymentStrategy { void pay(double amount); }
class UpiPayment implements PaymentStrategy { ... }
class CardPayment implements PaymentStrategy { ... }
```

**L — Liskov Substitution**
> A subclass must be substitutable for its parent without breaking behaviour.
```java
// VIOLATION: Square extends Rectangle, but setWidth breaks invariant
Rectangle r = new Square();
r.setWidth(5);
r.setHeight(10);
// Expected area = 50, but Square forces width=height → area = 100
```

**I — Interface Segregation**
> Don't force classes to implement methods they don't need.
```java
// BAD: fat interface
interface Worker { void work(); void eat(); }
class Robot implements Worker {
    void eat() { throw new UnsupportedOperationException(); } // robot can't eat

// GOOD: split
interface Workable { void work(); }
interface Eatable { void eat(); }
```

**D — Dependency Inversion**
> Depend on abstractions, not concrete classes.
```java
// BAD
class OrderService {
    private MySQLOrderRepository repo = new MySQLOrderRepository(); // concrete
}

// GOOD
class OrderService {
    private OrderRepository repo; // interface
    public OrderService(OrderRepository repo) { this.repo = repo; } // injected
}
```

---

---

# PART 6 — Multithreading

---

## 6.1 volatile vs synchronized vs AtomicInteger

```java
// volatile — visibility guarantee, no atomicity
private volatile boolean running = true;
// Reads/writes go directly to main memory, not CPU cache
// Good for flags, NOT for compound operations like i++

// synchronized — mutual exclusion + visibility
synchronized void increment() {
    count++; // only one thread at a time
}

// AtomicInteger — lock-free using CAS (Compare-And-Swap)
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet(); // atomic, no lock, faster than synchronized
```

### The i++ Problem
```java
// i++ is NOT atomic — it's 3 operations:
// 1. Read i
// 2. Add 1
// 3. Write i back
// Thread A and Thread B can interleave between these steps → lost update
```

## 6.2 ThreadLocal

```java
// Each thread gets its OWN copy of the variable
ThreadLocal<User> currentUser = new ThreadLocal<>();

// In filter/interceptor:
currentUser.set(authenticatedUser);

// Anywhere in the same thread:
User user = currentUser.get();

// IMPORTANT: Always clean up after request
currentUser.remove(); // prevent memory leaks in thread pools
```

## 6.3 CompletableFuture

```java
CompletableFuture<User> future = CompletableFuture
    .supplyAsync(() -> userRepo.findById(id))  // runs on ForkJoinPool
    .thenApply(user -> enrich(user))           // transform result
    .thenCompose(user -> fetchOrders(user))    // flatMap — returns another future
    .exceptionally(ex -> defaultUser());       // handle errors

// Combine two futures
CompletableFuture<String> combined = CompletableFuture
    .allOf(future1, future2)
    .thenApply(v -> future1.join() + future2.join());
```

---

---

# PART 7 — Exception Handling

---

## 7.1 Checked vs Unchecked

```
Checked (extends Exception):
  - Must be declared or caught at compile time
  - Examples: IOException, SQLException, ClassNotFoundException
  - Use for: recoverable conditions (file not found, network timeout)

Unchecked (extends RuntimeException):
  - Not required to catch or declare
  - Examples: NullPointerException, ArrayIndexOutOfBoundsException, IllegalArgumentException
  - Use for: programming errors, invalid state
```

## 7.2 Custom Exception Best Practice

```java
// Custom exception with error code (production pattern)
public class PaymentException extends RuntimeException {
    private final String errorCode;

    public PaymentException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }

    public PaymentException(String errorCode, String message, Throwable cause) {
        super(message, cause); // always chain the original cause
        this.errorCode = errorCode;
    }
}

// Spring global exception handler
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(PaymentException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ProblemDetail handle(PaymentException ex) {
        ProblemDetail detail = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        detail.setTitle("Payment Failed");
        detail.setDetail(ex.getMessage());
        detail.setProperty("errorCode", ex.getErrorCode());
        return detail; // RFC 7807 format
    }
}
```

---

---

# PART 8 — Quick-Fire Interview Questions

---

## Java Quick-Fire

**Q: What is the difference between `==` and `.equals()`?**
> `==` compares references (memory address). `.equals()` compares content. For `String`, `Integer` (cached -128 to 127), always use `.equals()`.

**Q: What is autoboxing? What's the trap?**
```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b); // true — cached

Integer c = 128;
Integer d = 128;
System.out.println(c == d); // false — new objects, beyond cache range

// Always use: c.equals(d) // true
```

**Q: What is a marker interface?**
> Interface with no methods — used to mark a class for special JVM or framework treatment. Examples: `Serializable`, `Cloneable`, `RandomAccess`.

**Q: Difference between `throw` and `throws`?**
> `throw` actually throws an exception instance. `throws` in method signature declares that the method might throw that exception type.

**Q: What is `transient`?**
> Field marked `transient` is excluded from Java serialization. Use for sensitive data (passwords), derived fields, or non-serializable fields.

**Q: What is `instanceof` and pattern matching (Java 16+)?**
```java
// Old way
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// Java 16+ pattern matching
if (obj instanceof String s) {
    System.out.println(s.length()); // s is already cast
}
```

---

## Spring Quick-Fire

**Q: What is `@Qualifier`?**
> When multiple beans of the same type exist, `@Qualifier("beanName")` tells Spring which one to inject.

**Q: What is `@Primary`?**
> Marks a bean as the default injection candidate when multiple candidates exist. `@Qualifier` overrides `@Primary`.

**Q: What is `@Lazy`?**
> Delays bean instantiation until first use. `@Lazy` on `@Autowired` means the proxy is injected immediately but the actual bean is only created on first method call.

**Q: What is `@ConditionalOnProperty`?**
```java
@Bean
@ConditionalOnProperty(name = "feature.cache.enabled", havingValue = "true", matchIfMissing = false)
public CacheManager cacheManager() { ... }
// Only creates this bean if feature.cache.enabled=true in properties
```

**Q: What is Spring Boot Actuator?**
> Production-ready endpoints: `/actuator/health`, `/actuator/metrics`, `/actuator/env`, `/actuator/beans`. Add `spring-boot-starter-actuator` dependency. Secure with Spring Security in prod.

**Q: What is the difference between `@Controller` and `@RestController`?**
> `@RestController = @Controller + @ResponseBody`. `@Controller` returns view names (for Thymeleaf/JSP). `@RestController` returns JSON/XML directly.

**Q: What happens if two `@Configuration` classes define the same bean?**
> In Spring Boot, the last one wins by default (based on component scan order). You can use `@Primary` or `@Conditional` annotations to control which one is used. Best practice: never define the same bean twice.

---

---

# PART 9 — Java 8–17 Features Cheat Sheet

---

## Java 8
```java
// Lambda
Comparator<String> c = (a, b) -> a.compareTo(b);

// Stream API
List<Integer> evens = list.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

// Optional
Optional<User> user = Optional.ofNullable(findUser(id));
user.ifPresent(u -> process(u));
String name = user.map(User::getName).orElse("Anonymous");

// Default methods in interfaces
interface Validator {
    boolean validate(String s);
    default boolean isNotEmpty(String s) { return s != null && !s.isEmpty(); }
}

// Method references
list.forEach(System.out::println);
List<String> names = users.stream().map(User::getName).collect(Collectors.toList());
```

## Java 11
```java
// var (local type inference)
var list = new ArrayList<String>(); // inferred as ArrayList<String>

// String methods
"  hello  ".strip();        // Unicode-aware trim
"hello".repeat(3);          // "hellohellohello"
"".isBlank();               // true

// Files.readString / writeString
String content = Files.readString(Path.of("file.txt"));
```

## Java 14–17
```java
// Records (Java 16)
record Point(int x, int y) {} // auto-generates constructor, getters, equals, hashCode, toString
Point p = new Point(1, 2);
p.x(); // getter

// Sealed classes (Java 17)
sealed class Shape permits Circle, Rectangle {}
final class Circle extends Shape {}
final class Rectangle extends Shape {}
// Only Circle and Rectangle can extend Shape

// Pattern matching for switch (Java 17 preview)
String result = switch (obj) {
    case Integer i -> "int: " + i;
    case String s -> "string: " + s;
    default -> "other";
};

// Text blocks (Java 15)
String json = """
    {
        "name": "Nilesh",
        "role": "engineer"
    }
    """;
```

---

---

# PART 10 — The Tricky One-Liners (Interview Traps)

---

```java
// 1. What prints?
String s1 = "hello";
String s2 = "hello";
String s3 = new String("hello");
System.out.println(s1 == s2);    // true — same pool object
System.out.println(s1 == s3);    // false — s3 is on heap
System.out.println(s1.equals(s3)); // true — same content

// 2. What prints?
Integer a = 100, b = 100;
System.out.println(a == b); // true — cached (-128 to 127)
Integer c = 200, d = 200;
System.out.println(c == d); // false — beyond cache range

// 3. What is the output?
try {
    return 1;
} finally {
    return 2; // finally OVERRIDES the return in try
}
// Output: 2

// 4. Can you catch an Error?
try {
    throw new OutOfMemoryError();
} catch (Error e) {
    System.out.println("caught"); // YES — but you shouldn't in production
}

// 5. What prints?
List<String> list = new ArrayList<>(Arrays.asList("a", "b", "c"));
for (String s : list) {
    if (s.equals("b")) list.remove(s); // ConcurrentModificationException!
}
// Fix: use Iterator.remove() or list.removeIf(s -> s.equals("b"))

// 6. Static initializer order
class A {
    static int x = 10;
    static { x = 20; } // runs after field init
}
// A.x == 20

// 7. Abstract class with constructor — can it be called?
abstract class Base {
    Base() { System.out.println("Base constructor"); }
}
class Child extends Base {
    Child() { super(); } // YES — called by subclass constructor
}
```

---

---

## Study Order for Interview Prep

```
Day 1 (2 hrs): Part 1 — Core Java (String, hashCode, beans)
Day 2 (2 hrs): Part 4 — Spring Boot (@Transactional, ApplicationContext, lifecycle)
Day 3 (2 hrs): Part 3 — Sorting + Collections
Day 4 (1 hr):  Part 8 — Quick-fire + tricky one-liners
Day 5 (1 hr):  Mock — answer out loud, no notes
```

**The one rule:** For every concept, ask yourself **"what breaks if this wasn't true?"**
- String not immutable? String pool breaks, HashMap keys break, threads break.
- hashCode without equals? HashMap lookups silently fail.
- @Transactional proxy? Self-invocation silently skips transactions.

That's the depth level interviewers are probing for.
