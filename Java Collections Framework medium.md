## ArrayList vs LinkedList? When would you choose one over the other?
Both ArrayList and LinkedList implement the List interface, but they differ in internal structure and performance:
**Internal Structure**:ArrayList is backed by a dynamic array. It starts with an initial capacity and resizes automatically as elements are added.LinkedList is backed by a doubly-linked list. It stores elements in nodes, where each node links to the previous and next nodes.
**Read Operations**:ArrayList provides direct index access with O(1) time complexity.LinkedList requires traversing node-by-node to reach an index, resulting in O(n) time complexity.
**Insertion & Deletion**:ArrayList takes O(n) when inserting or deleting in the middle because remaining elements must be shifted.LinkedList takes O(1) for insertion or deletion once you reach the node (especially fast at the head or tail).
**When to use**:Use ArrayList when your application is read-heavy.Use LinkedList when you frequently add or remove elements at the ends or perform frequent modifications without index lookups.

## How does ArrayList grow dynamically? What is the default capacity and growth factor?
Initially, ArrayList creates an internal array with a default capacity of 10.
When the array becomes 100% full, it resizes by creating a new array with 50% more capacity (1.5x). It then copies the elements from the old array into the new one and discards the old array.
This same resizing process repeats each time the array reaches full capacity

## What is the difference between HashSet, LinkedHashSet, and TreeSet?
The main differences between HashSet, LinkedHashSet, and TreeSet are their underlying data structures, ordering guarantees, and time complexity:
**HashSet**:
Backed by a HashMap.
Does not guarantee any insertion order.
Operations (add, remove, contains) run in O(1) time.
**LinkedHashSet**:
Backed by a LinkedHashMap (hash table + doubly-linked list).
Preserves insertion order.
Also offers O(1) time complexity with minor overhead for node references.
**TreeSet**:
Backed by a TreeMap (Red-Black tree).
Stores elements in sorted order (natural or via a Comparator).
Operations run in O(log n) time, and it does not allow null values.

## What is the difference between HashMap, LinkedHashMap, and TreeMap?
The main difference between HashMap, LinkedHashMap, and TreeMap is the underlying data structure they use, the time complexity, and the ordering of elements:
**HashMap** uses a hash table under the hood, doesn't preserve any insertion order, and has an average time complexity of $O(1)$. It allows one null key.
**LinkedHashMap** uses a hash table and a doubly-linked list under the hood. It preserves the insertion order, has an average time complexity of $O(1)$, and allows one null key.
**TreeMap** uses a Red-Black tree under the hood. It stores elements in sorted order with a time complexity of $O(\log n)$, and it doesn't allow null keys.
TreeMap doesn't allow null elements (since it compares keys), whereas HashMap and LinkedHashMap permit one null

## HashSet vs TreeSet vs LinkedHashSet? How does HashSet work internally?
The main difference between HashSet, TreeSet, and vs LinkedHashSet is the underlying data structure they use and the insertion order they maintain, I mean the order of the elements they maintain, and the time complexity of operations. Basically, HashSet is backed by HashMap. It doesn't preserve any insertion order and time complexity of operations is O(1). The next thing is TreeSet. It's backed by a TreeMap. It sorts the element in ascending order and the time complexity of operations is O(log n). Coming to LinkedHashSet, it is backed by LinkedHashMap. It preserves the insertion order and the time complexity, operation time complexity is O(1). And coming to how HashSet works internally is basically it just under the hood uses HashMap object. It stores whatever we are adding to set, it stores them as keys and for values, it just uses some dummy object to store that as value. That's how it works under the hood.
TreeSet doesn't allow null elements (since it compares keys), whereas HashSet and LinkedHashSet permit one null



## Explain Comparable vs Comparator interfaces. Which one modifies the original class?

> "Both are interfaces used to provide comparison logic for sorting custom objects:
> * **`Comparable` modifies the original class.** Our custom class implements it and overrides `compareTo()`. It defines a single, fixed natural sorting order.
> * **`Comparator` does not modify the original class.** It is implemented externally—typically via lambda expressions or anonymous classes—and passed directly into the `sort()` method. This allows us to define multiple, flexible sorting strategies (like sorting by name, age, or price) without touching the original class code."
> 
> 

## How do you sort a collection using a custom comparator?

To sort a collection using a custom comparator, we implement the compare method—typically using a lambda expression or Comparator.comparing(). We then pass that comparator into either list.sort(comparator) or Collections.sort(list, comparator) to apply our custom sorting order.
Your explanation covers the exact conceptual mechanism: overriding the `compare` method via a lambda or anonymous class and passing that logic into the sort method.

---

### The Two Practical Ways to Sort

Suppose you have a `List<Employee> employees` and want to sort by `salary`:

#### 1. Using `List.sort()` (Most common in modern Java)

You call `.sort()` directly on the list and pass a lambda or method reference:

```java
// Lambda:
employees.sort((e1, e2) -> Integer.compare(e1.getSalary(), e2.getSalary()));

// Or using Comparator.comparing (cleanest):
employees.sort(Comparator.comparingInt(Employee::getSalary));

```

#### 2. Using `Collections.sort()` (Classic utility method)

```java
Collections.sort(employees, (e1, e2) -> Integer.compare(e1.getSalary(), e2.getSalary()));

```

*(Note: If you are working with streams instead of modifying the list in-place, you can also use `employees.stream().sorted(comparator)...`)*

---
## Explain the difference between Iterator and ListIterator.
they are the objects Java uses behind the scenes when you run a `for-each` loop to traverse collections.
> "Both `Iterator` and `ListIterator` are used to traverse collections, but the key differences are:
> * **`Iterator`** works across all collection types (`List`, `Set`, `Queue`). It moves only in **one direction** (forward) using `hasNext()` and `next()`, and supports only reading and `remove()`.
> * **`ListIterator`** is specific **only to `List` implementations** (like `ArrayList` and `LinkedList`). It supports **bidirectional traversal** (forward and backward using `previous()`), tracks element indexes, and allows you to **add, modify (`set`), or remove** elements during iteration."
> 
> 

---

### Key Differences

| Feature | `Iterator` | `ListIterator` |
| --- | --- | --- |
| **Applicability** | Works on **any `Collection**` (`List`, `Set`, `Queue`). | Works **only on `List**` implementations (`ArrayList`, `LinkedList`). |
| **Direction** | **One-way only** (forward). | **Bi-directional** (forward and backward). |
| **Traversal Methods** | `hasNext()`, `next()` | `hasNext()`, `next()`, `hasPrevious()`, `previous()` |
| **Index Access** | Cannot tell you the current index. | Has `nextIndex()` and `previousIndex()`. |
| **Modification Operations** | Can only **read** and **remove** (`remove()`). | Can **read**, **remove**, **add** (`add()`), and **replace/set** (`set()`). |

---

---

## for each on ArrayList or LinkedList uses ListIterator by default or do we have to explicitly mention the ListIterator?
A `for-each` loop **always uses standard `Iterator**`, even on `ArrayList` and `LinkedList`. It **never** uses `ListIterator` by default.

---

### How the Compiler Handles `for-each`

Under the hood, Java’s enhanced `for-each` loop is syntax sugar for any object that implements `Iterable<T>`.

When you write:

```java
List<String> list = new ArrayList<>();

for (String s : list) {
    System.out.println(s);
}

```

The Java compiler rewrites it into bytecode using `iterator()`:

```java
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String s = it.next();
    System.out.println(s);
}

```

Because `Iterable` defines only the method `iterator()`, the `for-each` loop can only ever invoke standard `Iterator`.

---

### You Must Explicitly Request `ListIterator`

If you want the extra power of `ListIterator` (going backwards, modifying via `set()`, or checking indexes), you must call `list.listIterator()` manually in code:

```java
ListIterator<String> listIt = list.listIterator();

// Moving backward from the end:
ListIterator<String> reverseIt = list.listIterator(list.size());
while (reverseIt.hasPrevious()) {
    System.out.println(reverseIt.previous());
}

```

---

### The Quick Interview Answer

> "A `for-each` loop always uses standard `Iterator` under the hood because it relies on the `Iterable` interface. To use `ListIterator`—for two-way traversal, index tracking, or element modification—you must obtain it explicitly by calling `list.listIterator()`."

## What is the difference between Fail-Fast and Fail-Safe iterators? Give examples.
> "The main difference between fail-fast and fail-safe iterators is how they handle modifications during iteration:
> * **Fail-Fast iterators** operate directly on the original collection. If the collection is modified while iterating (other than via the iterator's own `remove()` method), it immediately throws a **`ConcurrentModificationException`** using an internal `modCount` counter.
> * *Examples:* `ArrayList`, `HashSet`, `HashMap`.
> 
> 
> * **Fail-Safe iterators** operate on a snapshot or a copy of the collection, not the direct data. They do not throw an exception if modifications occur during iteration.
> * *Examples:* `CopyOnWriteArrayList`, `ConcurrentHashMap`."
> 
> 
> 
> 
---
### Core Differences

| Feature | Fail-Fast Iterator | Fail-Safe (Weakly Consistent) Iterator |
| --- | --- | --- |
| **Behavior on Modification** | Throws **`ConcurrentModificationException`** immediately if modified during iteration. | **Does not throw** an exception; continues iterating safely. |
| **How It Operates** | Operates directly on the **actual collection data**. | Operates on a **clone / snapshot** or a weakly consistent view of the data. |
| **Modification Tracking** | Checks an internal counter called **`modCount`** on every step. | Does not rely on strict `modCount` matching. |
| **Memory & Overhead** | Fast, uses no extra memory copy. | Incurs memory/performance overhead (especially if copying underlying arrays). |
| **Examples** | `ArrayList`, `LinkedList`, `HashSet`, `HashMap` (standard `java.util` collections). | `CopyOnWriteArrayList`, `ConcurrentHashMap` (`java.util.concurrent` collections). |

---

### How Fail-Fast Works (The `modCount` Mechanism)

Standard collections maintain an internal variable:

```java
protected transient int modCount = 0;

```

1. Every time you call `add()` or `remove()` directly on the collection, `modCount` increments by `1`.
2. When you create an `Iterator`, it saves a local copy: `expectedModCount = modCount`.
3. On every `next()` call, it checks:
```java
if (modCount != expectedModCount)
    throw new ConcurrentModificationException();

```

If you alter the list from outside the iterator while looping, the numbers mismatch and it fails immediately.

---
### How Fail-Safe Works

Instead of looking at the live structure under strict lock:

* **`CopyOnWriteArrayList`**: Creates an exact copy (snapshot) of the array whenever a write occurs. The iterator keeps reading the old snapshot, so it never sees the mid-loop modification or crashes.
* **`ConcurrentHashMap`**: Uses a *weakly consistent* iterator that reflects the state of the map at or since creation, without locking or throwing exceptions.
---

## How does PriorityQueue work internally? (Min-heap / Max-heap)
> "`PriorityQueue` serves elements based on their priority rather than FIFO order.
> * **Internally**, it is implemented as a **binary heap** backed by a **dynamic array**.
> * By default, Java uses a **Min-Heap**, meaning the smallest element stays at index `0`. We can make it a **Max-Heap** by passing `Collections.reverseOrder()`.
> * **Performance:** Peeking at the highest-priority element is **$O(1)$**, while inserting (`add`) and removing (`poll`) take **$O(\log n)$** because Java must restore the heap order."
> 
> 


## What is CopyOnWriteArrayList? When should it be used?

> "`CopyOnWriteArrayList` is a thread-safe implementation of `List`.
> * **How it works:** Any write operation (`add`, `remove`, `set`) creates a brand-new cloned copy of the entire underlying array under a lock, while read operations and iterators access the snapshot without any locking.
> * **Why no exception:** Iterators loop over the fixed snapshot, making them fail-safe and immune to `ConcurrentModificationException`.
> * **When to use:** In **read-heavy, write-rare** multi-threaded scenarios—like storing event listeners or notification subscribers. We avoid it when writes are frequent because copying the array on every write is expensive ($O(n)$)."
> 
> 

## What is a BlockingQueue? How is it used in Producer-Consumer problems?

`BlockingQueue` acts as a thread-safe buffer that naturally synchronizes the speed differences between producers and consumers:

1. **When Consumers Are Faster Than Producers:**
* Consumers consume everything; the queue empties.
* Consumers calling `take()` are automatically suspended.
* As soon as a producer calls `put()`, the queue signals and wakes up a waiting consumer.

2. **When Producers Are Faster Than Consumers (Backpressure):**
* If using a bounded queue (e.g., size 100), producers fill it up.
* The next producer calling `put()` is suspended, stopping the application from blowing up with an `OutOfMemoryError`.
* As soon as a consumer calls `take()`, space opens up, signaling a waiting producer to wake up.

> "With a standard queue, we have to write explicit, error-prone code using `wait()`, `notify()`, and locks to coordinate communication between producer and consumer threads.
> A **`BlockingQueue`** handles all this thread synchronization automatically:
> * When the queue is **empty**, a consumer calling **`take()`** blocks automatically until an element arrives.
> * When the queue is **full**, a producer calling **`put()`** blocks automatically until space opens up, giving us built-in backpressure.
> 
> 
> It eliminates manual locking, avoids busy-waiting (0% CPU waste while waiting), and provides thread-safe coordination right out of the box."

---

## Difference between ArrayBlockingQueue and LinkedBlockingQueue?

> "The main differences come down to locking, capacity, and memory:
> 1. **Locking & Throughput:** `ArrayBlockingQueue` uses a **single shared lock** for both reads and writes, meaning a producer and consumer cannot act at the exact same instant. `LinkedBlockingQueue` uses **two separate locks** (`putLock` and `takeLock`), allowing producers and consumers to operate in parallel with higher throughput.
> 2. **Capacity:** `ArrayBlockingQueue` is **always bounded** with a fixed capacity set at creation. `LinkedBlockingQueue` is **optionally bounded**—if left unbounded, it defaults to `Integer.MAX_VALUE`, which can lead to `OutOfMemoryError`.
> 3. **Memory & Garbage Collection:** `ArrayBlockingQueue` is backed by a circular array and allocates **zero node objects** during inserts. `LinkedBlockingQueue` creates a new `Node` object for every single item added, leading to higher GC overhead."
> 
> 

---
## How do you sort a HashMap by values?

> "Java does not have a direct `map.sort()` method because `HashMap` has no concept of order.
> To sort a map by values:
> 1. We extract the entries using **`map.entrySet()`**.
> 2. We sort them using a **`Comparator`** that targets the value (`Map.Entry.comparingByValue()`).
> 3. We collect the sorted entries into a **`LinkedHashMap`**, which is necessary to preserve the sorted insertion order."


There is **no built-in method like `map.sort()` or `MapUtils.sort()**` in standard Java.

---

### The Two Steps to Sort a Map by Values

1. **Extract and sort:** Pull the key-value pairs (`Map.Entry`) out into a list or stream, and sort them using a `Comparator` on `entry.getValue()`.
2. **Collect into a `LinkedHashMap`:** A standard `HashMap` does not maintain order. You must insert the sorted entries into a **`LinkedHashMap`** to preserve that sorted sequence.

---

### Method 1: Using Java 8 Streams (Modern & Interview-Preferred)

Java provides `Map.Entry.comparingByValue()` out of the box:

```java
Map<String, Integer> map = Map.of("A", 30, "B", 10, "C", 20);

Map<String, Integer> sortedMap = map.entrySet()
    .stream()
    .sorted(Map.Entry.comparingByValue()) // Uses a Comparator on values
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        Map.Entry::getValue,
        (e1, e2) -> e1,
        LinkedHashMap::new // Crucial: preserves the sorted order!
    ));

```

---

### Method 2: Classic Approach (List + `Collections.sort`)

If an interviewer asks how to do it without the Stream API:

```java
// 1. Convert entrySet to a List
List<Map.Entry<String, Integer>> list = new ArrayList<>(map.entrySet());

// 2. Sort the list with a custom Comparator on values
list.sort((e1, e2) -> e1.getValue().compareTo(e2.getValue()));

// 3. Put sorted entries into a LinkedHashMap
Map<String, Integer> sortedMap = new LinkedHashMap<>();
for (Map.Entry<String, Integer> entry : list) {
    sortedMap.put(entry.getKey(), entry.getValue());
}

```
---
## What is the difference between Optional and null? When should Optional be used?

> "`Optional` was introduced to eliminate silent `NullPointerException`s and nested null checks by providing an **explicit API contract**.
> * When a method returns `Optional<T>`, it clearly communicates to the developer that the value might be absent, forcing them to handle that case gracefully using functional methods like `.map()`, `.filter()`, or `.orElse()`.
> * **Best Practice:** It should **only be used as a return type** for methods that might not have a value. We should never use it as class fields (it adds heap overhead and isn't `Serializable`), as method parameters, or wrapped around collections (empty collections should just return `Collections.emptyList()`)."
> 
> 

---
