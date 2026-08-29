# Java Revision Notes — Mock Interview Session

Each topic: full explanation (the "why," so you actually understand it) → **Say this** (the tightened spoken answer for the interview) → common follow-up.

---

## 1. HashMap vs ConcurrentHashMap
**Explanation:** `HashMap` isn't thread-safe — concurrent writes from multiple threads can corrupt its internal structure. `ConcurrentHashMap` fixes this by locking at the **bucket level** instead of locking the whole map — so multiple threads can write to *different* buckets at the same time without blocking each other; only two threads writing to the *same* bucket actually contend. Reads are largely lock-free (volatile reads), which is why it scales better than wrapping a plain HashMap in `Collections.synchronizedMap()`, which locks the entire map for every single operation — reads included.

**Say this:**
> "HashMap isn't thread-safe — concurrent writes can corrupt it. ConcurrentHashMap solves that by locking at the bucket level instead of the whole map, so multiple threads can write to different buckets simultaneously, and reads are mostly lock-free. I'd use it any time a map is shared across threads — for example, an in-memory cache accessed by multiple request-handling threads in a Spring Boot service."

**Follow-up:** "Why not synchronizedMap?" → It locks the entire map for every operation, including reads — much worse throughput.

---

## 2. wait() vs sleep()
**Explanation:** Both pause a thread, but they differ in what happens to the lock. `sleep()` belongs to `Thread` — it pauses the current thread for a fixed time and **keeps holding** any lock it has; it doesn't need to be inside a `synchronized` block. `wait()` belongs to `Object` — it pauses the thread **and releases the lock** on that object, so other threads can get in; it must be called inside a `synchronized` block or you get `IllegalMonitorStateException`. It stays paused until another thread calls `notify()`/`notifyAll()` on the same object, or an optional timeout passes. This is the mechanism behind producer-consumer coordination: a consumer calls `wait()` when there's nothing to consume, releasing the lock so a producer can get in, produce something, and `notify()` the consumer to wake up.

**Say this:**
> "Both put a thread into a waiting state, but they differ in lock handling. sleep() keeps any lock it holds until the thread finishes sleeping. wait() releases the lock immediately, and resumes either when another thread calls notify() on that object, or when an optional timeout passes. wait() is for thread coordination — like a producer-consumer setup — sleep() is just a plain delay, like a retry backoff before re-polling Kafka."

**Follow-up code cue:** wait()/notify() must be inside `synchronized`; use `while` not `if` to re-check the condition after waking (guards against spurious wakeups).

---

## 3. volatile vs synchronized
**Explanation:** `volatile` guarantees **visibility** — every thread reads the latest value from main memory instead of a cached/local copy — but it does *not* guarantee **atomicity**. `synchronized` gives you both visibility and atomicity (mutual exclusion). The classic trap: `volatile` does NOT make `i++` thread-safe, because that's actually three steps (read, increment, write) — volatile only guarantees each individual read/write sees the latest value, not that the whole sequence happens without interruption. Two threads can both read the same value before either writes back, losing an increment. That's why `AtomicInteger` (using CAS — compare-and-swap) exists, to make the whole read-modify-write atomic without a full lock.

**Say this:**
> "volatile gives you visibility — every thread sees the latest write — but not atomicity. So it's fine for a flag like `running = false` that one thread sets and others just read. But for something like a counter with i++, volatile alone isn't enough because the operation isn't atomic — you'd need synchronized or AtomicInteger instead."

---

## 4. String Immutability
**Explanation:** Three real reasons. First, the **string pool** — multiple references can point to the same pooled string object; if strings were mutable, modifying through one reference would corrupt every other reference pointing to the same object. Second, **security** — strings are used everywhere for file paths, credentials, network addresses; mutability would open vulnerabilities if something changed a string after it was validated. Third, **hashmap key integrity** — a String's `hashCode()` is computed and cached at creation, and Strings are commonly used as map keys. If a String were mutable and its contents changed after being used as a key, the map would still be looking for it in the *old* hash bucket, effectively losing the entry.

**Say this:**
> "String is immutable in Java for three main reasons. First, the string pool — multiple references can safely point to the same string object without one modification affecting the others. Second, security — strings are used for things like file paths and credentials, so mutability would open up vulnerabilities. Third, hashmap integrity — a string's hashcode is cached when it's created and used as a key. If the string were mutable, changing it after insertion would corrupt the map, since the key would no longer hash to the bucket it's actually stored in."

---

## 5. == vs .equals()
**Explanation:** `==` compares **references** — do two variables point to the same object in memory. `.equals()` compares **content/value**. With string literals, the JVM pools them, so two literals with the same value can actually share the same reference, making `==` return `true`. But `new String("abc")` forces a brand-new heap object outside the pool, so `==` returns `false` even though the content is identical — only `.equals()` correctly returns `true` here. Practical risk: code like `if (userInput == "admin")` can silently evaluate to `false` even when the value is right, because `userInput` likely didn't come from a literal — a real bug source, not just trivia.

**Say this:**
> "== compares object references — do two variables point to the same memory location. .equals() compares actual content. With string literals, the JVM pools them, so two literals with the same value can share a reference and == returns true. But with new String(...), you force a separate object even with identical content, so == returns false while .equals() correctly returns true. That's why you always use .equals() for strings — relying on == can silently break if the string didn't come from a literal."

---

## 6. Checked vs Unchecked Exceptions
**Explanation:** The real distinguishing factor is the **class hierarchy and compiler enforcement**, not "runtime vs compile-time occurrence" (both kinds actually happen at runtime). **Checked** exceptions extend `Exception` but not `RuntimeException` — e.g. `IOException`, `SQLException`. The compiler forces you to either `catch` them or declare `throws` — they represent recoverable conditions the caller should be prepared to handle, like a failed network call. **Unchecked** exceptions extend `RuntimeException` — e.g. `NullPointerException`, `IllegalArgumentException`. The compiler has no way to predict these, so it doesn't force any handling — they usually represent programming bugs rather than expected failure conditions.

**Say this:**
> "Checked exceptions extend Exception but not RuntimeException — the compiler forces you to either catch them or declare throws. They represent recoverable conditions, like an IOException from a network call or an SQLException from a database call. Unchecked exceptions extend RuntimeException — the compiler has no way to predict these, so it doesn't enforce handling. They're usually programming or logic errors, like a NullPointerException or IllegalArgumentException."

**Custom exception example (have ready):**
> "For a custom checked exception, I'd use something like InsufficientInventoryException in my order platform — the caller needs to catch it and trigger a compensating transaction. For a custom unchecked one, something like InvalidOrderRequestException for bad input — that should bubble up to a global @ControllerAdvice handler and return a 400."

---

## 7. Abstract Class vs Interface
**Explanation:** Both achieve abstraction, but they differ in state and intent. An abstract class can hold **instance-level state** (real fields with values per object); an interface can't — only constants (`public static final`) and, since Java 8, `default`/`static` methods with implementation, but never instance fields. The deciding factor for which to use isn't just "forcing method implementation" (both can do that) — it's whether there's a genuine **"is-a" relationship with shared state/implementation** (→ abstract class) versus defining a **contract/capability** with no shared state, especially across unrelated classes (→ interface). Java also allows implementing multiple interfaces but only extending one class — so if a class needs to satisfy multiple unrelated contracts, interfaces are the only option.

**Say this:**
> "Both achieve abstraction. The main difference: an abstract class can hold instance-level state, an interface can't — only constants and, since Java 8, default and static methods. We use an abstract class when there's a genuine is-a relationship with shared state or shared implementation — for example, an abstract PaymentProcessor with a shared logTransaction() method, extended by StripeProcessor and PaypalProcessor. We use an interface to define a contract or capability, especially across unrelated classes — like a RateLimiter interface with tryAcquire(), implemented by TokenBucketRateLimiter."

**Follow-up:** Java allows multiple interface implementation, only single class inheritance.

---

## 8. equals() / hashCode() Contract
**Explanation:** The rule: if two objects are equal according to `equals()`, they **must** have the same `hashCode()`. Why this matters mechanically: a `HashMap`/`HashSet` doesn't scan every entry — it first computes `hashCode()` to jump straight to a **bucket**, then uses `equals()` to compare only the entries **within** that bucket. If you override `equals()` (say, to mean "same orderId") without also overriding `hashCode()` to match, the object's hash code stays identity-based (memory address) — so two objects that `equals()` would call "the same" can land in **completely different buckets** and never even get compared. The map treats them as unrelated, and you silently get a duplicate instead of a detected match. The fix: always override both together, based on the same fields — or let the IDE, Lombok's `@EqualsAndHashCode`, or a `record` generate them consistently.

**Say this:**
> "If two objects are equal according to equals(), they must have the same hashCode(). This matters because hash-based collections like HashMap and HashSet use hashCode() to decide which bucket to look in, and only use equals() to compare objects within that bucket. If you override equals() without overriding hashCode() to match, two logically-equal objects can get different hash codes, land in different buckets, and never get compared — so the collection treats them as different even though equals() says they're the same. That's why you always override both together, based on the same fields."

**Concrete example (have ready):**
> "This is real in my order platform's ProcessedEvent deduplication — if I override equals() on eventId but forget hashCode(), a redelivered Kafka event with the same eventId but a new object instance won't be detected as a duplicate, because contains() looks in the wrong bucket entirely. That breaks the idempotency guarantee."

---

## 9. Generics
**Explanation:** Before generics, collections held plain `Object` — you could add anything to a list with no compiler warning, and bugs only surfaced later as a `ClassCastException` at runtime when you cast wrong, often far from where the actual mistake happened. Generics let you declare `List<String>`, so the compiler catches type mismatches **at compile time** instead, and you no longer need manual casts on read. `List<Object>` vs `List<?>` is a subtler distinction: `List<Object>` is a concrete list that specifically holds `Object` — you can add anything to it once you have one, but a `List<String>` variable is *not* assignable to a `List<Object>` variable, even though String is an Object. `List<?>` means "list of some unknown type" — it *can* accept a `List<String>`, `List<Integer>`, anything, as a parameter, but you can't add anything to it (except null) since the compiler doesn't know the actual type.

**Say this:**
> "Generics catch type mismatches at compile time instead of runtime. Without them, you could add any type to a list and only find out about a bad cast when it crashes in production. List<Object> is a concrete list of Objects you can add to, but a List<String> isn't assignable to it. List<?> means 'list of some unknown type' — flexible to accept as a parameter, but you can't add to it since the actual type is unknown."

---

## 10. Streams — Intermediate vs Terminal
**Explanation:** A `Stream` processes a sequence of elements through a pipeline of operations instead of manual loops. **Intermediate operations** (`filter`, `map`, `sorted`) transform or filter the stream and return *another stream* — they're **lazy**, meaning nothing actually executes when those lines run; they just build up the pipeline definition. **Terminal operations** (`collect`, `forEach`, `count`, `reduce`) actually *trigger* the pipeline to run, processing each element through the entire chain in one pass, and produce a real result — after which the stream is "used up" and can't be reused.

**Say this:**
> "A Stream processes a sequence of elements through a chain of operations. Intermediate operations — like filter, map, sorted — are lazy, they don't execute right away. They only run once a terminal operation — like collect, count, reduce — is called, which triggers the whole pipeline and produces the actual result."

**Three code patterns to have ready cold:**
```java
// filter + sort + collect
orders.stream()
    .filter(o -> o.getStatus() == OrderStatus.PENDING)
    .sorted(Comparator.comparing(Order::getAmount))
    .collect(Collectors.toList());

// filter + aggregate
double total = orders.stream()
    .filter(o -> o.getStatus() == OrderStatus.COMPLETED)
    .mapToDouble(Order::getAmount)
    .sum();

// unique-key map — toMap, not groupingBy
Map<Long, Order> byId = orders.stream()
    .collect(Collectors.toMap(Order::getOrderId, order -> order));
```
**Say when asked toMap vs groupingBy:**
> "groupingBy is for when a key can repeat — it produces a Map of key to List. toMap is for when the key is guaranteed unique — one key maps to one value directly, and it throws if it finds a duplicate key."

---

## 11. Optional
**Explanation:** The problem `Optional` solves isn't technical, it's about **visibility of a possible absence**. With plain null checks, nothing forces you to check — if a method returns `Order` and might return `null`, the method signature gives zero warning; you find out via a `NullPointerException` somewhere downstream, disconnected from the actual cause. `Optional<Order>` as a return type bakes the "might be missing" signal into the type system itself, so the caller is required to handle it — via `orElseThrow()`, `ifPresent()`, `orElse()`/`orElseGet()`. It moves the responsibility from "developer must remember" to "compiler/type system makes it visible" — the same underlying idea as checked exceptions and generics. It's meant for **return types**, not fields, parameters, or collections — wrapping every variable in Optional is an anti-pattern.

**Say this:**
> "Optional forces the developer to handle the case where a value might be missing. With a plain null check, the developer has to remember to check — if they forget, you get a NullPointerException somewhere downstream, disconnected from where the actual issue was. Optional makes that a compile-time requirement instead, using methods like orElseThrow() or ifPresent() to explicitly handle the absent case. It's meant for return types especially — not for fields or parameters."

---

## 12. Functional Interfaces
**Explanation:** All are "single abstract method" interfaces, differing in whether they take input, return output, and whether they can throw checked exceptions.
| Interface | Input | Output | Checked exceptions? | Use case |
|---|---|---|---|---|
| `Runnable` | No | No | No | Fire-and-forget task |
| `Callable<V>` | No | Yes | Yes | Executor task needing a result |
| `Supplier<T>` | No | Yes | No | Lazy value production (e.g. `orElseGet`) |
| `Function<T,R>` | Yes | Yes | No | Transformation (e.g. `.map()`) |
`Runnable`'s `run()` returns nothing and can't declare a checked exception (must catch internally). `Callable<V>`'s `call()` returns a value and can throw — `ExecutorService.submit(Callable<V>)` returns `Future<V>`, and `.get()` blocks until the result is ready. `Supplier` looks similar to `Callable` (no input, returns a value) but it's general-purpose — used throughout Streams/Optional, not tied to threading, and can't throw checked exceptions. `Function<T,R>` takes input and transforms it — this is literally what powers `.map()` in Streams.

**Say this:**
> "Runnable and Callable are both functional interfaces used to submit tasks to something like an ExecutorService. The difference: Callable's call() method returns a value and can throw a checked exception, while Runnable's run() returns nothing and can't throw a checked exception. Supplier also takes no input and returns a value, but it's general-purpose — used for lazy value production, like Optional's orElseGet(). Function takes an input and returns an output — that's what powers map() in Streams."

**Decision rule to say out loud:** "If it's being submitted to an executor as a standalone task, it's Runnable or Callable. If it's transforming a value I already have, it's Function."

---

## 13. final / finally / finalize
**Explanation:** Three unrelated concepts that just sound alike. `final` has three uses: a `final` variable can't be reassigned, a `final` method can't be overridden, a `final` class can't be extended (e.g. `String` itself). `finally` is a block that runs **no matter what** after try/catch — whether the try succeeded, an exception was caught, or even if a `return` happens inside `try` — the JVM guarantees it runs before the method actually exits. The only things that skip it are `System.exit()` or a JVM crash. This is why it's used for guaranteed cleanup: code placed *after* a try-catch block only runs if control flow naturally falls through to it, but a `return` inside `try` or an uncaught exception both skip that — `finally` is specifically designed to run regardless. `finalize()` is a completely different, older mechanism — a method the garbage collector *used to* call on an object right before reclaiming its memory, meant as a last chance to clean up. It's deprecated since Java 9 and unreliable (you don't control if/when GC runs), so the modern replacement is try-with-resources.

**Say this:**
> "final prevents reassignment on a variable, overriding on a method, or extension on a class. finally is a block that always runs after try/catch — even if there's a return inside try, or an uncaught exception — used for guaranteed cleanup like closing a connection. finalize() was a garbage-collector hook meant for cleanup before an object is destroyed, but it's deprecated since Java 9 and unreliable, since you don't control when or if it runs. The modern replacement is try-with-resources."

**If asked "why not just put cleanup code after try-catch":**
> "Because a return inside try, or an uncaught exception, skips any code written after the block — finally is guaranteed to run before the method actually exits, regardless."

---

## 14. try-with-resources
**Explanation:** Any resource that needs explicit closing (a `Connection`, `FileInputStream`, Kafka consumer) traditionally required a manual `finally` block with a null-check before calling `close()` — verbose and easy to get wrong. Try-with-resources fixes this: declare the resource inside the `try`'s parentheses, and the JVM automatically calls `.close()` on it once the block finishes, whether it succeeded or threw — no `finally` needed. This works for any class implementing `AutoCloseable` (a single-method interface: `void close()`). Multiple resources can be declared together and close automatically in reverse order of declaration.

**Say this:**
> "Try-with-resources automatically closes any resource that implements AutoCloseable, without needing a manual finally block — you declare it in the try's parentheses, and Java guarantees close() gets called whether the block succeeds or throws."
```java
try (Connection conn = openConnection()) {
    // use conn
} catch (Exception e) {
    log.error("failed", e);
}
```

---

## 15. Heap vs Stack
**Explanation:** Heap is **shared across all threads** and stores actual **objects**. Stack is **per-thread** and stores local variables — primitives (actual values stored directly) and references (pointers to heap objects) — plus method call frames. `Order order = new Order();` splits into two things: the actual `Order` instance lives on the heap, while `order` (the reference/variable) lives on the stack, just pointing at that heap location. Because each thread gets its own stack, local variables are automatically isolated per-thread with zero risk of one thread stomping another's locals — no synchronization needed. The heap, being shared, is exactly where multi-threading problems come from — which connects directly back to why `ConcurrentHashMap` exists.

**Say this:**
> "Heap is shared across all threads and stores actual objects. Stack is per-thread and stores local variables, primitives, and references pointing to heap objects, plus method call frames. Because each thread has its own stack, local variables are automatically thread-safe — heap objects are where concurrency issues actually happen, which is why we need things like ConcurrentHashMap."

---

## 16. Instance vs Reference
**Explanation:** An **instance** is the actual object sitting in memory (on the heap), created by `new` — the real thing with actual field values. A **reference** is a variable that *points to* an instance — like an address written on paper, not the house itself. Multiple references can point to the same instance: `Order order2 = order1;` creates a new reference (`order2`) pointing to the *same* object as `order1` — not a new order. Mutating through one reference is visible through the other, since they're really the same object. This is also exactly why `==` compares references (do two variables point to the same instance), not content.

**Say this:**
> "An instance is the actual object in memory, created with new. A reference is a variable that points to that instance — like an address on paper versus the actual house. Multiple references can point to the same instance — if I do order2 = order1, both variables point to the same object, and mutating through one is visible through the other. That's also why == compares references, not content."

---

## 17. JVM Generational GC
**Explanation:** The root reason the heap is split at all is the **generational hypothesis**: most objects die young — the vast majority of objects (local variables, temp calculation objects) are used briefly and discarded almost immediately; very few live a long time (like a cache or a singleton bean). If the JVM treated the whole heap as one region, every GC cycle would have to scan everything, including long-lived objects that are obviously still in use — slow and wasteful. So the heap is split by age: **Young Generation** is where all new objects are created, and it's cleaned frequently via **Minor GC** — fast, since most objects there are already garbage by the time it runs. Objects that survive several Minor GC rounds (still referenced, still in use) get **promoted to Old Generation**, which is cleaned much less often via **Major/Full GC** — expensive, because it has to scan a much larger region, but rare because objects there have already proven they're long-lived.

**Say this:**
> "The JVM splits the heap based on the pattern that most objects die young. Young Generation holds new objects and gets scanned frequently with Minor GC — cheap, since most things there are already garbage. Objects that survive multiple Minor GC cycles get promoted to Old Generation, which is scanned rarely with Major GC, since objects there have proven they're long-lived."

---

## 18. Overloading vs Overriding
**Explanation:** Overloading = same method name, different parameter list (can also differ in return type, as long as parameters differ) — resolved at **compile time** ("static binding"), purely based on the argument types in the call; the compiler decides which method body to call before the program even runs. Overriding = same signature redefined in a subclass — resolved at **runtime** ("dynamic binding"), based on the **actual object type**, not the declared/reference type. E.g. `PaymentProcessor p = new StripeProcessor(); p.process();` calls `StripeProcessor`'s version, even though `p` is declared as `PaymentProcessor` — because the JVM checks what the object actually *is* at runtime. Why this has to be runtime: the compiler only knows a variable's declared type — it has no way of knowing what actual object will be assigned, since that can depend on runtime conditions, user input, or dependency injection decisions that don't exist yet at compile time. This mechanism is what makes polymorphism work — a `List<PaymentProcessor>` holding a mix of subclasses correctly calls each one's own `.process()`.

**Say this:**
> "Overloading is resolved at compile time based on the arguments you pass — the compiler picks the method purely from parameter types. Overriding is resolved at runtime based on the actual object's type, not the reference type — that's what makes polymorphism work. The compiler only knows a variable's declared type, not what actual object it'll hold — that's often decided dynamically, through conditions or dependency injection — so it can't pick the overridden method body ahead of time. The JVM has to check the real object type once the program is executing."

---

## 19. Composition vs Inheritance
**Explanation:** Both promote code reuse, but the relationship differs — inheritance is "is-a" (`StripeProcessor extends PaymentProcessor` gets all its parent's behavior automatically), composition is "has-a" (`OrderService` holds a `PaymentProcessor` as a field and delegates to it, rather than extending it). Composition is favored for three real reasons: (1) **fragile base class problem** — a subclass is tightly coupled to its parent's internal implementation, not just its public contract, so if the parent's internals change, subclasses can silently break even without being touched; (2) **Java only allows single inheritance** — `extends` is capped at one class, while composition has no such limit; (3) **runtime flexibility + testability** — a composed object can be swapped at runtime (e.g. `orderService.setPaymentProcessor(new PaypalProcessor())`), which is impossible with inheritance since the relationship is fixed at compile time. This is exactly the pattern behind Spring's dependency injection — `@Autowired` composes a `PaymentProcessor` implementation into `OrderService` without any inheritance, and lets you swap in a mock for testing.

**Say this:**
> "Both are ways to promote code reusability. Inheritance is an is-a relationship, composition is a has-a relationship. Composition is preferred for a few reasons: with inheritance, if the parent changes its internal implementation, the child class can break even without being touched — that's the fragile base class problem. Java also only allows single inheritance, so you're boxed in if you need behavior from multiple sources. Composition avoids tight coupling through dependency injection — you can swap implementations at runtime, which is also what makes code easier to unit test. This is literally what Spring's dependency injection is built on."

---

## Delivery reminders
- Slow down — most stumbles tonight were pacing/repetition, not knowledge gaps.
- Land answers on a clear closing sentence — don't trail off into a question or repeat the last phrase.
- Say "thread" deliberately (not "threat"), "intermediate" (not "internet"), watch enum case (OrderStatus.COMPLETED, all-caps).
- Structure every answer: definition → why it matters / what breaks without it → concrete example. That pattern worked well all night.
