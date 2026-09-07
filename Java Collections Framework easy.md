## What is the Java Collections Framework?
The Java Collections Framework provides built-in data structures and algorithms to store and manipulate groups of objects. It saves developers from writing data structures from scratch by offering ready-to-use, efficient implementations like ArrayList, HashSet, and HashMap.

## Explain the Collection hierarchy in Java.
In the collection hierarchy, the Collection interface extends the Iterable interface.
The Collection interface is extended by three interfaces: List, Set, and Queue.
Each has its own implementations:
List has ArrayList and LinkedList
Set has HashSet and TreeSet
Queue has ArrayDeque and PriorityQueue
This collection hierarchy is specifically for storing single elements, not key-value pairs.

## Collection vs Collections?
The main difference between Collection and Collections is basically Collection is an interface and Collections is a class. And the thing is Collection is basically used to represent the root interface. Root interface used to represent single element data structures like list, set, and queue. And Collections is a class that provides Utility static methods to perform the operations on collections like Collections.sort() and Collections.reverse().

## Difference between List, Set, and Map
Basically, Set and List extend Collection hierarchy. Map is an independent, and List and Set are used to store elements of type single, and Map is used to store key-value pairs. List allows duplicate elements. Set doesn't allow duplicate elements. Map doesn't allow duplicate keys, and it can allow duplicate values. List maintains the insertion order, and we can fetch the elements by index. Set doesn't preserve any insertion order(though LinkedHashSet maintains insertion order and TreeSet maintains sorted order). and has no index access.

## What is the difference between Arrays and Collections?
The main difference between array and collections are array size is fixed once created, collections can grow or shrink dynamically. Arrays can store either primitive or object data types. Collections can only store object data types. Arrays are like low-level data structures. Collections provide rich high-level data structures like list, set, map.

## What is the Collections utility class used for?
The Collections utility class contains static methods used to perform operations on collections or return modified collection wrappers.
Its primary uses are:
Data manipulation algorithms: Common operations like sorting (Collections.sort()) or reversing (Collections.reverse()).
Thread-safe wrappers: Making collections thread-safe using methods like Collections.synchronizedList().
Read-only wrappers: Creating unmodifiable collections using methods like Collections.unmodifiableList().

## What is the Deque interface?
Basically, Deque stands for Double-Ended Queue. It is an interface that extends Queue and allows elements to be added and removed from both ends.
Because of this, it can act as both a Queue (FIFO) and a Stack (LIFO). Its most common implementations are ArrayDeque and LinkedList.

##  linked list will implement both the interfaces, list and queue.?
LinkedList implements both the List and Deque interfaces. This means it can be used as a standard indexed list, or as a double-ended queue / stack.

## What do you mean by standard indexed list? We never use index to fetch the item from linked list.
LinkedList implements List, so Java gives it methods like get(index), but under the hood it has no array indices. To fetch an element at an index, it traverses node by node, which makes random access slow (O(n)).
