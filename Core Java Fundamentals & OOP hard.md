## Why is String immutable in Java? What are the benefits of this immutability?
Immutability means an object's internal state cannot be modified in place once it is created. In Java, String is immutable for four main reasons. First, it enables the String Constant Pool for memory optimization; multiple references can point to the same object in the pool, and if strings were mutable, modifying one reference would unintentionally affect all other references pointing to that same object. Second, it guarantees security because strings are used for sensitive parameters like file paths, network URLs, and database credentials; if a string were mutable, it could be altered after passing security checks, creating severe vulnerabilities. Third, strings are widely used as HashMap keys, and because they are immutable, their hash code can be cached safely without risking map corruption or lost entries. Lastly, immutability makes strings inherently thread-safe across multiple threads without needing any synchronization.

## Explain the String Constant Pool (SCP). Where does it reside in memory? What is string interning?
String Constant Pool is a mechanism used by Java to optimize memory usage while creating string objects, and it resides inside the heap. Whenever we create string literals with the same value, instead of creating separate objects, Java stores a single object in the String Constant Pool and points both references to it. String interning is the process of storing or fetching a string reference from the String Constant Pool. String literals are interned automatically by Java. When we create a string object using the new keyword, it is created in the regular heap; we can call .intern() on it so that if the string does not exist in the pool, it gets added and its reference is returned, and if it already exists, it simply returns the reference from the pool.

## String vs StringBuilder vs StringBuffer — when to use which? Performance & thread-safety differences.
A String cannot be modified in place, so whenever we perform concatenation operations in a loop, a new object gets created each time, hurting performance. That is why we use StringBuilder or StringBuffer. Under the hood, both use a dynamically resizable character array; whenever we append characters, instead of creating new objects, they simply increase the size of the array and append the characters in place. The main difference between them is thread safety: StringBuilder is not thread-safe, so it is recommended for single-threaded environments because it is faster. In StringBuffer, methods are synchronized, making it safe for multi-threaded environments, but it comes with a bit of performance overhead due to thread locking.

## How do you create a truly Immutable class in Java? (What if the class contains a mutable object like java.util.Date?)
To create a truly immutable class, we first make the class final so no other class can extend it and override methods to break immutability. Next, we make all fields private and final, provide only getters without any setters, and initialize all fields through the constructor so proprties canenot be modified after object creation. If the class contains a mutable field like java.util.Date, we must use defensive copying. In the constructor, we never store the incoming reference directly; we create a new copy and store that instead so the outside caller cannot modify our internal state. In the getter, we never return the direct reference either; we return a newly created copy using that same defensive copying mechanism so the internal object stays completely protected.

## Explain the contract between equals() and hashCode(). What happens if you override equals() but not hashCode()?
the contract between equals() and hashCode() is that if two objects are logically equal according to equals(), they must produce the same hash code. However, if two objects have the same hash code, they do not necessarily need to be equal due to hash collisions. If we override equals() but do not override hashCode(), the class uses Object's default hashCode() method, which generates a hash code based on the object's memory address. When we add two logically equal objects to a HashMap or HashSet, their different hash codes make them land in different buckets. Because of this hash code mismatch, duplicate logically equal objects get stored in the map or set, breaking their core behavior.

## How does the clone() method work? Shallow Copy vs Deep Copy in Java — how to implement Deep Copy?
The clone() method belongs to the Object class and, by default, performs a shallow copy. The main difference between a shallow copy and a deep copy is how they handle nested objects. A shallow copy duplicates all primitive fields into the new object, but for nested object references, it only copies the memory address, meaning both the old and new objects end up pointing to the exact same child object in the heap. In contrast, a deep copy creates completely new instances for those nested references so that each object points to independent child objects, ensuring changes in one do not affect the other. We can implement a deep copy by overriding the clone() method to manually clone every nested object, by using a copy constructor to instantiate fresh objects for all nested fields, or by using serialization with object streams to reconstruct the entire object graph.

The `clone()` method belongs to the `Object` class, and by default, it creates a **shallow copy** of an object. To use it, the class must implement the `Cloneable` marker interface, otherwise calling `super.clone()` throws `CloneNotSupportedException`.

---
---

**How to Implement Deep Copy**

There are three common ways:

1. **Manual Cloning / Overriding `clone()`:** Call `super.clone()` for the outer object, then explicitly clone each inner mutable object.
```java
class Address implements Cloneable {
    String city;
    public Address(String city) { this.city = city; }
    @Override
    public Address clone() throws CloneNotSupportedException {
        return (Address) super.clone();
    }
}

class Person implements Cloneable {
    String name;
    Address address;

    @Override
    public Person clone() throws CloneNotSupportedException {
        Person cloned = (Person) super.clone(); // shallow copy
        cloned.address = this.address.clone();  // deep copy inner object
        return cloned;
    }
}

```


2. **Copy Constructor (Most Preferred in Modern Java):** Pass the existing object into a constructor and manually instantiate fresh copies of all nested objects:
```java
public Person(Person other) {
    this.name = other.name;
    this.address = new Address(other.address.city);
}

```


3. **Serialization:** Serialize the object graph into a byte stream and deserialize it back. This automatically produces a completely independent deep copy (though it has performance overhead).

---


## So basically, does object class contain this clone method already which is implemented? And why do we have to override the clone? I don't understand the part here. Because if we want to clone it to perform type casting, we have to clone it to perform type casting, is it?
We override the clone method basically when we want to change the visibility. If we want to allow our object to be cloned, we must expose the clone method. By default, object class clone method is protected, it cannot be accessed. So basically it cannot be accessed if that belongs to a different package, if the class doesn't belong to subclass, it cannot be accessed. So we override the clone method in our class and make it public. So that the classes which are using our objects to clone, clone method of our objects, they can use, they can access. And next one is when we want to implement deep copying, we will absolutely have to override the clone method. We have to absolutely override the clone method.

## So, you implemented deep copy for the address. Let's say if address has some, you know, it still has some nested objects. Let's say deep nested objects. So, what how does it work there? Will the deep learn code you wrote, will it work fine?

No, that code will **not** do a full deep copy if `Address` has nested objects.

Because inside `Address.clone()`, we called `super.clone()`. That native `super.clone()` only performs a **shallow copy** of `Address` itself. If `Address` contains its own mutable object—say, a `City` or `Street` object—only the reference to that `City` gets copied.

---

**What Happens With Nested Objects:**

If `Person` has an `Address`, and `Address` has a `City`:

```java
// Person.clone() clones Address -> Good so far
cloned.address = this.address.clone(); 

// BUT if Address.clone() only calls super.clone():
return (Address) super.clone(); 
// -> Address gets a new instance, but BOTH addresses still point to the EXACT SAME City object!

```

---

**How to Make It Work:**

Every single mutable class down the entire object tree must recursively clone its children:

1. `Person.clone()` must call `address.clone()`.
2. `Address.clone()` must call `city.clone()`.
3. `City.clone()` must clone its own mutable fields, and so on.

This cascading dependency is messy, fragile, and prone to breaking whenever a new field is added.

---

**The Clean Alternative for Deeply Nested Objects:**

When you have multiple layers of nested objects, developers avoid `clone()` and use **Serialization** or a library like Jackson (JSON serialization/deserialization):

```java
// Serializing to byte stream and reading back reconstructs the ENTIRE object graph deeply
ByteArrayOutputStream baos = new ByteArrayOutputStream();
ObjectOutputStream oos = new ObjectOutputStream(baos);
oos.writeObject(originalPerson);

ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
ObjectInputStream ois = new ObjectInputStream(bais);
Person deepCopiedPerson = (Person) ois.readObject();

```

---

## What is the Object class? What are its methods?
The Object class is the root class of the entire class hierarchy in Java, and every class inherits from it to provide a common set of baseline behaviors. It contains 11 methods:

toString(): Returns a textual representation of the object, commonly overridden for readability and debugging.

equals(Object obj): Checks logical equality between two objects, defaulting to reference equality (==).

hashCode(): Returns an integer hash value used by hash-based collections like HashMap and HashSet to compute bucket indices.

getClass(): Returns the runtime Class instance representing the object's actual class, widely used in reflection.

clone(): Creates and returns a shallow copy of the object, requiring the class to implement the Cloneable interface.

wait(), wait(long timeoutMillis), wait(long timeoutMillis, int nanos): Three overloaded methods that cause the executing thread to release the object's monitor lock and pause until notified or until the specified timeout expires.

notify(): Wakes up a single thread waiting on the object's monitor lock.

notifyAll(): Wakes up all threads waiting on the object's monitor lock.

finalize(): Historically invoked by the garbage collector before reclaiming an unreachable object; deprecated in Java 9 and removed due to unpredictability and resource leak risks.

## What is a Memory Leak in Java if Java has a Garbage Collector?
A memory leak in Java occurs when an object that is no longer needed by the application cannot be reclaimed by the garbage collector due to unintentional active references still pointing to it. The garbage collector can only free objects that are completely unreachable from GC roots, so these referenced but obsolete objects stay trapped in memory. Over time, these uncollected objects accumulate on the heap, eventually triggering a java.lang.OutOfMemoryError. Common causes include unbounded static collections, unclosed system resources, unregistered listeners, and unremoved ThreadLocal variables in thread pools.

## What are Enums in Java? Can an Enum implement an interface? Can it be instantiated using new?
An enum in Java is a special data type used to represent a fixed set of constants with compile-time type safety. Under the hood, the compiler generates a final class extending java.lang.Enum, where each constant is created as a public static final instance of that enum type.
Yes. It can implement one or more interfaces, allowing shared or constant-specific method implementations (though it cannot extend another class since it already extends java.lang.Enum).
No. Its constructors are implicitly or explicitly private. They are invoked strictly by the JVM during class loading to initialize the predefined constant instances, and the JVM explicitly prevents instantiation via new or reflection.

## I'm not understanding this enum concept properly.I'm not able to understand the exact use case, like how it is different from normal constants file. We can solve the problem using normal constants file, but why do we need separate enum object? 

A constants file uses regular types like `String` or `int`. An `enum` creates a completely **distinct data type**.

To see why a constants file fails, consider what happens in real code when using both approaches.

---

**1. The Problem with a Constants File (No Type Boundary)**

Imagine a constants file for order statuses and payment modes:

```java
public class AppConstants {
    // Order statuses
    public static final String ORDER_PENDING = "PENDING";
    public static final String ORDER_COMPLETED = "COMPLETED";

    // Payment modes
    public static final String PAYMENT_UPI = "UPI";
    public static final String PAYMENT_CARD = "CARD";
}

```

Now look at the service method:

```java
public void updateOrderStatus(String status) {
    // updates database with status...
}

```

Because the parameter is just a generic `String`, the compiler cannot enforce *which* string belongs there. All of these compile without any errors:

```java
// 1. Valid intended call:
updateOrderStatus(AppConstants.ORDER_COMPLETED);

// 2. Passing a completely unrelated constant from the same file:
updateOrderStatus(AppConstants.PAYMENT_UPI); // Compiles! Stored as status "UPI"!

// 3. Typo / invalid arbitrary string:
updateOrderStatus("COMPLETD");              // Compiles! Corrupts business logic!
updateOrderStatus("RANDOM_TEXT");           // Compiles!

```

Because `String` is an open set with infinite possibilities, you are forced to write manual runtime validation checks (`if (!status.equals(...))`) in every single method that receives a status.

---

**2. How an Enum Solves This**

An enum defines a **closed, bounded set** of allowed instances.

```java
public enum OrderStatus {
    PENDING, COMPLETED, CANCELLED
}

public enum PaymentMode {
    UPI, CARD, NET_BANKING
}

```

Now update the method signature to accept only that specific enum:

```java
public void updateOrderStatus(OrderStatus status) {
    // status is 100% guaranteed to be only PENDING, COMPLETED, or CANCELLED
}

```

Now test the same problematic calls:

```java
updateOrderStatus(OrderStatus.COMPLETED);      // Compiles cleanly.

updateOrderStatus(PaymentMode.UPI);            // COMPILE ERROR: PaymentMode cannot be converted to OrderStatus
updateOrderStatus("COMPLETED");                // COMPILE ERROR: String cannot be converted to OrderStatus
updateOrderStatus("RANDOM_GARBAGE");           // COMPILE ERROR: Won't even compile!

```

---

**3. Key Differences at a Glance**

| Feature | Constants Class (`static final String/int`) | Enum Class (`public enum`) |
| --- | --- | --- |
| **Type Boundary** | Open: Any arbitrary `String` or `int` is accepted | Closed: Only the declared constants are valid |
| **Error Detection** | Runtime (bugs, invalid database entries) | Compile-time (compiler blocks invalid values) |
| **Accidental Mixing** | Easy to pass a `PAYMENT` constant to an `ORDER` parameter | Impossible: different types cannot cross over |
| **Method Attachment** | Cannot attach methods or logic directly to constants | Can have custom methods, fields, and constructors |
| **`switch` Statements** | Prone to typos; no compiler check for missing cases | Compiler alerts or warns if cases are missing |

---

**Core Takeaway**

A constants file just groups loose values together under a common type (`String` or `int`). An `enum` invents a brand-new restricted type so the compiler prevents anyone from passing invalid or unrelated values into your methods.

## Hold on, I don't understand how can a Method accepting object of type order status can accept order status dot pending, order status dot completed or order status dot cancelled. Basically, we are not passing order status object, right?

Yes, you **are** passing an `OrderStatus` object.

`OrderStatus.PENDING`, `OrderStatus.COMPLETED`, and `OrderStatus.CANCELLED` are not strings, numbers, or labels—they are **actual, live instances (objects) of the `OrderStatus` class**.

---

**What the Java Compiler Actually Generates**

When you write this:

```java
public enum OrderStatus {
    PENDING,
    COMPLETED,
    CANCELLED
}

```

The Java compiler converts that code into a regular Java class that looks like this:

```java
public final class OrderStatus extends java.lang.Enum<OrderStatus> {

    // These ARE objects of type OrderStatus!
    public static final OrderStatus PENDING = new OrderStatus("PENDING", 0);
    public static final OrderStatus COMPLETED = new OrderStatus("COMPLETED", 1);
    public static final OrderStatus CANCELLED = new OrderStatus("CANCELLED", 2);

    private OrderStatus(String name, int ordinal) {
        super(name, ordinal);
    }
}

```

---

**Why the Method Accepts Them**

Because `OrderStatus.COMPLETED` is literally a `public static final OrderStatus` object:

```java
public void updateStatus(OrderStatus status) {
    // ...
}

```

When you write:

```java
updateStatus(OrderStatus.COMPLETED);

```

You are passing the pre-created, static **object** named `COMPLETED` of type `OrderStatus`.

Every single constant listed inside an enum is a real object instantiated on the heap when the class loads.

## If constructor is private, how can JVM create the object?
When you write an enum, the compiler translates it into a final class extending java.lang.Enum with a private constructor.
It generates a static initialization block (<clinit>) directly inside that class. Because that code lives within the class body, it has full access to call the private constructor, creating the public static final instances for each declared constant when the JVM loads the class.

```java
public final class OrderStatus extends java.lang.Enum<OrderStatus> {

    // These ARE objects of type OrderStatus!
    public static final OrderStatus PENDING = new OrderStatus("PENDING", 0);
    public static final OrderStatus COMPLETED = new OrderStatus("COMPLETED", 1);
    public static final OrderStatus CANCELLED = new OrderStatus("CANCELLED", 2);

    private OrderStatus(String name, int ordinal) {
        super(name, ordinal);
    }
}

```

## Can an abstract class have constructor? Can you instantiate an abstract class?
Yes, an abstract class can have a constructor, but we cannot instantiate an abstract class using the new keyword. Its construction exists solely to initialize base class fields when called through child class constructor using super.

## How does Java Reflection work? When should you avoid it? What is the relationship between reflection and encapsulation?
Java Reflection is an API used to inspect and access classes, methods, fields, and constructors dynamically at runtime without hardcoding names at compile time.
It directly breaks encapsulation because calling setAccessible(true) bypasses private access modifiers to read or modify internal state.
We should avoid it in business logic because it introduces performance overhead, loses compile-time type safety by failing at runtime, and makes code fragile during refactoring.
It is primarily designed for frameworks and libraries like Spring Boot, Hibernate, and JUnit.

## What are Annotations? How do you create a custom Annotation? What is annotation retention and target?
An annotation provides metadata about classes, methods, or fields, allowing either the compiler or the JVM to process it.
We create a custom annotation using the @interface keyword, configured primarily with two meta-annotations:
@Retention: Defines the lifespan of the annotation:
SOURCE: Stays only till compilation and is discarded before bytecode is generated.
CLASS: Recorded in the .class bytecode, but not loaded into memory by the JVM at runtime.
RUNTIME: Retained in JVM memory at runtime, allowing it to be read via Reflection.
@Target: Restricts where the annotation can be applied using ElementType—such as methods, classes, fields, or parameters.

## What is Serialization and Deserialization? What is serialVersionUID? Why is Serializable considered weakly designed?
Java serialization converts an in-memory object into a byte stream for storage or network transfer, and deserialization reconstructs that stream back into an active object.
serialVersionUID is a version identifier declared in a Serializable class. During deserialization, the JVM verifies that the byte stream's version matches the loaded class's ID to prevent loading incompatible data, throwing an InvalidClassException if they differ.
It is considered weakly designed primarily because:
Bypasses Constructors: It instantiates objects without calling any constructor, bypassing critical validation and class invariants.
Security Vulnerabilities: Arbitrary deserialization exposes systems to gadget chain attacks, which can lead to Remote Code Execution (RCE).
Tight Coupling: It locks internal private implementation details to the serialized wire format, complicating refactoring.

## Difference between ClassNotFoundException and NoClassDefFoundError?
ClassNotFoundException is a checked exception that occurs during explicit dynamic loading—such as using Class.forName() or ClassLoader.loadClass()—when the specified class name string cannot be found on the classpath. It is recoverable and must be caught or declared.
NoClassDefFoundError is a fatal LinkageError that occurs when a class was available at compile time, but the JVM cannot locate or link it during runtime execution (when attempting to instantiate an object, call a static method, or access a static member). This typically happens due to a missing JAR in deployment or because the class failed during its static initialization (<clinit>).

## What is Type Erasure in Java Generics?
Type erasure is the compile-time process where the Java compiler enforces strict type safety during compilation, but then strips away all generic type information from the resulting bytecode to ensure backward compatibility with older JVM versions.
Under the hood, a List<String> is compiled into a raw List of Object. Whenever an element is retrieved from the list and assigned to a String reference, the compiler automatically inserts an explicit cast to (String) in the bytecode.

## Explain upper bounds (? extends T) and lower bounds (? super T) in Generics (PECS principle).
PECS stands for Producer extends, Consumer super:
Upper Bound (? extends T — Producer):
Used when we primarily want to read elements from a collection.
The collection can hold T or any subtype of T, so we can safely read elements as type T.
Writing is blocked because the compiler cannot guarantee the exact runtime subtype of the list (e.g., trying to add an Integer to what is actually a List<Double>).
Lower Bound (? super T — Consumer):
Used when we want to write or add elements into a collection.
The collection is backed by T or an ancestor of T, meaning we can safely add T or any subclass of T into it.
When reading, we only get Object back because the exact supertype cannot be guaranteed.

### Why Do We Even Need Wildcards?

In Java, generic types are **invariant**. Even though `Integer` is a subtype of `Number`, a `List<Integer>` is **not** a subtype of `List<Number>`.

```java
List<Number> numbers = new ArrayList<Integer>(); // COMPILE ERROR!

```

If the compiler allowed this, you could write `numbers.add(3.14)` (adding a `Double` into what is actually an `ArrayList<Integer>`), destroying type safety.

Wildcards (`?`) solve this by providing flexible bounds.

---

### 1. Upper Bound: `? extends T` (Producer Extends)

Use `? extends T` when your collection acts as a **Producer**—meaning your code is primarily **reading data from** the collection.

* **Syntax:** `List<? extends Number>`
* **Meaning:** "A list containing instances of `Number` or any subtype of `Number` (e.g., `Integer`, `Double`)."

#### Reading is Allowed (Safe):

When you pull an item out, the compiler knows that whatever is in the list is guaranteed to be at least a `Number`.

```java
public void printNumbers(List<? extends Number> list) {
    for (Number n : list) { // SAFE: everything is at least a Number
        System.out.println(n.doubleValue());
    }
}

```

#### Writing is Forbidden:

You **cannot** add anything to a `? extends T` list (except `null`).

```java
public void addNumber(List<? extends Number> list) {
    list.add(10); // COMPILE ERROR!
}

```

*Why?* The compiler doesn't know whether the actual list passed at runtime is a `List<Integer>`, a `List<Double>`, or a `List<Float>`. Adding an `Integer` to a `List<Double>` would break type safety.

---

### 2. Lower Bound: `? super T` (Consumer Super)

Use `? super T` when your collection acts as a **Consumer**—meaning your code is primarily **writing data into** the collection.

* **Syntax:** `List<? super Integer>`
* **Meaning:** "A list containing `Integer` or any supertype of `Integer` (e.g., `Number`, `Object`)."

#### Writing is Allowed (Safe):

You can safely add `Integer` objects (or any subtype of `Integer`) into this list.

```java
public void addIntegers(List<? super Integer> list) {
    list.add(1);  // SAFE
    list.add(2);  // SAFE
}

```

*Why?* Whether the list passed at runtime is a `List<Integer>`, `List<Number>`, or `List<Object>`, an `Integer` can legally exist in all of them.

#### Reading is Limited:

When you read an element, the compiler only guarantees it is an `Object`.

```java
public void readFromConsumer(List<? super Integer> list) {
    Integer i = list.get(0); // COMPILE ERROR!
    Object obj = list.get(0); // SAFE, but you lose specific type information
}

```

---

### The Classic Example: `Collections.copy`

The JDK's `Collections.copy()` method is the textbook demonstration of PECS:

```java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    for (int i = 0; i < src.size(); i++) {
        dest.set(i, src.get(i));
    }
}

```

* `src` **produces** data to be read $\rightarrow$ **`? extends T`**
* `dest` **consumes** data to be stored $\rightarrow$ **`? super T`**

---

### Quick Summary for an Interview

| Bound | Keyword | PECS Role | Safe Operation | Unsafe / Blocked Operation |
| --- | --- | --- | --- | --- |
| **Upper Bound** | `? extends T` | **Producer** | **Reading** items as type `T` | **Writing** anything (except `null`) |
| **Lower Bound** | `? super T` | **Consumer** | **Writing** items of type `T` | **Reading** items as specific type `T` (only yields `Object`) |

* **Rule of thumb:** If you only pull values out of a collection, use `extends`. If you only push values into a collection, use `super`. If you need to do both, don't use wildcards—use an exact type parameter (`List<T>`).

## Explain Classloaders in Java (Bootstrap, Extension, Application). Parent delegation model?
A ClassLoader is a JVM component that dynamically loads .class bytecode into memory on demand.
Java provides three built-in loaders in a hierarchy:
Bootstrap Class Loader: Loads core JDK classes (java.lang.*, java.util.*).
Platform / Extension Class Loader: Loads standard platform extension modules.
Application Class Loader: Loads our application code and third-party dependencies from the classpath.
They follow the Parent Delegation Model:
When a class needs to be loaded, the request goes to the Application Loader first, but it delegates the request upward to its parent all the way to the Bootstrap Loader.
Only if the parent loaders cannot find the class does the child loader attempt to load it from the classpath. This ensures security—so a custom java.lang.String in our classpath cannot override the trusted JDK String—and uniqueness, preventing identical classes from being loaded multiple times.

## What is the immutable class design pattern?
To create a truly immutable class, we first make the class final so no other class can extend it and override methods to break immutability. Next, we make all fields private and final, provide only getters without any setters, and initialize all fields through the constructor so properties cannot be modified after object creation. If the class contains a mutable field like java.util.Date, we must use defensive copying. In the constructor, we never store the incoming reference directly; we create a new copy and store that instead so the outside caller cannot modify our internal state. In the getter, we never return the direct reference either; we return a newly created copy using that same defensive copying mechanism so the internal object stays completely protected.

## Why do we need to make fields final?
Without final, the CPU or JIT compiler can reorder instructions during object creation, potentially publishing the object reference to another thread before its constructor finishes assigning field values.
The Java Memory Model provides a safe publication guarantee for final fields: it inserts a memory barrier at the end of the constructor, ensuring no thread can ever observe a partially initialized object. All final fields are guaranteed to be fully visible to all threads as soon as the constructor finishes.

## What is the difference between core language features and library features in Java?
Core language features define the fundamental grammar, reserved keywords, and control structures enforced directly by the Java compiler. These include syntax rules, constructs like class and interface, loops like for and while, and exception handling. You cannot change or invent these without modifying the compiler itself.
Library features, on the other hand, are reusable classes, data structures, and APIs that are built using those core language features and bundled with the JDK—like List, HashMap, and Math. Instead of reinventing the wheel from scratch, we can simply import and reuse these pre-built components to build our applications faster.
