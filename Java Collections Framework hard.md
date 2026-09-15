## Explain the internal working of HashMap. (Hash collision, Buckets, Node array, load factor)?

> "`HashMap` is backed by an array of buckets with an initial capacity of **16** and a load factor of **0.75**.
> 1. When adding a key-value pair, it computes the key's hash and determines the bucket index.
> 2. If the bucket is empty, it inserts a new node. If occupied, it iterates through the bucket chain, comparing the incoming key against existing keys using both **hash code and `equals()**`:
> * If the key matches, it **replaces the old value** with the new value.
> * If no key matches, it appends a new node to the end of the chain.
> 
> 
> 3.  If a bucket's chain reaches **8** nodes and table capacity is at least **64**, the chain converts into a **Red-Black tree**, reducing search complexity from $O(n)$ to **$O(\log n)$**.
> 4. Once total entries exceed the threshold ($\text{capacity} \times \text{load factor}$, or $16 \times 0.75 = 12$), the array size **doubles** and elements are re-indexed."
> 
> 

---

## What changes were made to HashMap in Java 8? (Treeify threshold, O(log n) lookup)
Prior to Java 8, HashMap used to store elements in a bucket backed by a linked list data structure. In the worst case, if all elements have the same hash, which will lead to storing all the elements in the same bucket in the linked list data structure. So, time complexity for reading or writing operations will be O(n). So, to solve this, in Java 8, when the items in the linked list exceeds 8 and elements in all the Table exceeds 64, HashMap would convert the linked list data structure to a red-black tree. The worst-case time complexity after converting it to red-black tree is O(log n). 

## Why is it recommended to use a power of 2 for HashMap capacity?
> "There are two main reasons:
> 1. Finding a bucket index normally requires `hash % capacity`, but the modulo/division operator takes multiple CPU clock cycles. When capacity $n$ is a power of 2, `hash % n` is mathematically identical to the bitwise operation **`hash & (n - 1)`**, which executes in a single CPU clock cycle.
> 2. When $n$ is a power of 2, $n - 1$ is always all binary ones (for example, $16 - 1 = 15$, or `1111` in binary). This acts as an open mask where every bit of the hash can contribute to the index. If $n$ were not a power of 2, the mask would contain zeroes, causing certain bucket indices to never be used and drastically increasing collisions."
> 
> 

---

## What is a ConcurrentHashMap? How does it differ from Hashtable and Collections.synchronizedMap()?
> "All three are thread-safe maps, but they differ sharply in locking granularity and performance:
> 1. **`Hashtable` and `Collections.synchronizedMap**` use **coarse-grained locking**. They lock the entire map for every operation—meaning writes block writes, and reads block other reads, leading to heavy contention.
> 2. **`ConcurrentHashMap`** is optimized for high concurrency:
> * **Lock-free reads:** Read operations never block and execute concurrently at full speed.
> * **Bucket-level write locking:** It only locks the specific bucket being modified (using CAS for empty buckets and synchronizing on the head node for collisions), allowing multiple threads to write simultaneously to different buckets."
> 
> 
> 
> 

---

You were cutting right to the finish line, and every single concept you articulated is dead accurate. You clearly understand the shift from Java 7 Segment locking to Java 8's dynamic bucket locking.

Here is how you cleanly finish those final two thoughts:

> "...and empty buckets use **CAS** (Compare-And-Swap) for lock-free insertion, while buckets with collisions greater than **8 elements** convert into a **Red-Black Tree** to keep lookup at $O(\log n)$. Most importantly, **reads never block** because node values and next pointers are declared `volatile`."

---

## Explain the internal working of ConcurrentHashMap (Java 7 Segment locking vs Java 8 CAS & Node locking).

> "In **Java 7**, `ConcurrentHashMap` used **Segment Locking**:
> * The map was split into 16 `Segment` objects, each wrapping its own bucket array with an independent `ReentrantLock`.
> * It required two rounds of hashing: first to find the segment, then to find the bucket within it.
> * The limitation was a strict ceiling: only 16 concurrent writes, and two threads writing to different buckets in the *same* segment blocked each other.
> 
> 
> In **Java 8**, segments were removed in favor of **fine-grained bucket-level concurrency**:
> 1. **Lock-Free Inserts:** Empty buckets use atomic **CAS** (Compare-And-Swap) with zero locking overhead.
> 2. **Node-Level Locking:** When collisions occur, it synchronizes **strictly on the head node** of that specific bucket (`synchronized(headNode)`), leaving every other bucket open for simultaneous writes.
> 3. **Treeification:** Buckets with $\ge 8$ colliding nodes convert to a **Red-Black Tree** ($O(\log n)$).
> 4. **Lock-Free Reads:** Reads never acquire locks—even on buckets actively being modified—because the `val` and `next` references in `Node` are marked **`volatile`**."
> 
> 

---

## What is a TreeMap? When would you use it over a HashMap?

> "`TreeMap` implements `NavigableMap` and is backed by a self-balancing **Red-Black Tree**, whereas `HashMap` is backed by an array of buckets.
> * **Ordering:** `TreeMap` keeps its keys continuously sorted based on natural ordering (`Comparable`) or a custom `Comparator`.
> * **Performance:** Operations like `get()`, `put()`, and `remove()` run in **$O(\log n)$** time, compared to $O(1)$ average time in `HashMap`.
> * **Null Keys:** `TreeMap` **rejects `null` keys** with a `NullPointerException` because it must compare keys to place them in the tree.
> * **When to use:** Choose `TreeMap` when you require **sorted iteration**, **range queries** (such as `subMap()`), or **boundary searches** like `floorKey()` and `ceilingKey()`."
> 
> 

---

## Can we use null as a key in HashMap? What about ConcurrentHashMap? Why?

> "Yes, `HashMap` allows exactly **one `null` key** (and multiple `null` values). It avoids a `NullPointerException` by hardcoding the hash of a `null` key to **`0`**, storing it directly in bucket index `0`.
> In contrast, `ConcurrentHashMap` **strictly bans both `null` keys and `null` values**:
> * **The Ambiguity Problem:** If `map.get(key)` returns `null`, it could mean the key is missing **or** the value is explicitly `null`.
> * **The Concurrency Race Condition:** In a non-concurrent map, you can disambiguate with `containsKey(key)`. But in a concurrent system, state can change between the `get()` and `containsKey()` calls—another thread could insert, delete, or modify that entry in between.
> * To keep lookups deterministic and support atomic operations like `putIfAbsent()` or `computeIfAbsent()`, `ConcurrentHashMap` permanently reserves `null` to mean only one thing: **the entry does not exist**."
> 
> 

---
## What happens if two threads try to modify a HashMap simultaneously?
So basically, if two threads are trying to modify the same hash map, that would lead to corruption of hash map. The size variable will get corrupted, and the tail or head, these things will get corrupted, or treeification will also get corrupted.

## Why are HashMap keys typically immutable?

> "A key's `hashCode()` is mathematically calculated from its internal fields (for example, via `Objects.hash(fieldA, fieldB)`).
> * **At Insertion:** `HashMap` computes the hash code from the key's current field values to determine its initial bucket index and stores the entry there.
> * **After Mutation:** If the key is mutable and its fields are modified, calling `hashCode()` on that same object now yields a **completely different number**.
> * **The Retrieval Failure:** When calling `map.get(key)`, the map calculates the new hash code and searches the corresponding **new bucket index**. Because the entry was placed in the old bucket based on its original hash, the map looks in the wrong bucket and returns **`null`**.
> * **Memory Leak:** The original entry remains stuck in the old bucket forever—it can neither be found nor removed.
> 
> 
> That is why keys in a `HashMap` must be **immutable** (like `String` or `Integer`), ensuring their hash code and bucket destination remain constant for their entire lifecycle."

---

## What is rehashing? When does it happen?

> "**Rehashing** is the process where `HashMap` doubles its internal bucket array and redistributes existing entries across the new buckets:
> * **Trigger:** It occurs when the map size exceeds the threshold ($\text{capacity} \times \text{load factor}$). With default values ($16 \times 0.75 = 12$), rehashing triggers on the 13th element.
> * **Mechanism:** A new array with double the capacity is allocated, and the index for each existing node is recomputed against the new capacity.
> * **Purpose:** It reduces bucket collision chain lengths and restores average **$O(1)$** performance."
> 
> 

---
## What is a hash collision and how is it resolved?

> "A **hash collision** occurs when two distinct keys produce the same hash code or map to the same bucket index via `hash & (n - 1)`.
> * Java's `HashMap` resolves this using **Separate Chaining**.
> * Colliding entries are initially stored in a **singly linked list** at that bucket. When traversing the chain, it calls `.equals()` to check if the key already exists (updating the value) or appends a new node.
> * In Java 8+, if collisions in a bucket reach **8 elements** (with total table capacity $\ge 64$), the chain converts into a **Red-Black Tree** to keep search time bounded at **$O(\log n)$**."
> 
> 
