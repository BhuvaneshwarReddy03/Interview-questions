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
