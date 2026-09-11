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
