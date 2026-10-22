# Time Complexity: O(1) and O(n)

Time complexity describes how the amount of work an operation does grows as
the input grows. Here, **n** is the number of items in a collection.

- **O(1) — constant time:** the work stays roughly the same as `n` grows.
- **O(n) — linear time:** the work grows in proportion to `n`. An operation
  may need to inspect or move every item.

Big O describes growth, not an exact number of seconds. It also describes an
upper bound: an O(n) search might finish on its first check, but in the worst
case it checks all `n` items.

## Common operations

| Data structure | Access by index | Search by value | Add/remove at end | Add/remove at front or middle |
| --- | --- | --- | --- | --- |
| Array (`T[]`) | O(1) | O(n) | Fixed size; no append or removal | O(n) when items must be shifted |
| `List<T>` (dynamic array) | O(1) | O(n) | Append: amortized O(1); remove last: O(1) | O(n) when items must be shifted |
| Stack | Top/peek: O(1) | O(n) if searching | Push/pop: O(1)* | Not a normal stack operation |
| Queue | Front/peek: O(1) | O(n) if searching | Enqueue/dequeue: O(1)* | Not a normal queue operation |
| Singly linked list | O(n) | O(n) | Add at head: O(1); add at tail: O(1)** | Add/remove at head: O(1); elsewhere: O(n) to find the position |
| Doubly linked list | O(n) | O(n) | Add at head or tail: O(1)** | Add/remove at either end: O(1); elsewhere: O(n) to find the position |

`*` Standard stack and queue operations are O(1) when implemented with a
dynamic array (amortized for resizing) or a linked list. A queue implemented
by shifting an array on every dequeue would instead take O(n).

`**` Tail operations are O(1) when the implementation keeps a tail reference.
Without one, finding the tail of a linked list takes O(n).

## Why the costs differ

### Arrays and `List<T>`

An array stores items next to one another in memory. Its index gives the
location directly, so reading `items[i]` is O(1). Searching by value is
usually O(n), because the value may be anywhere.

Inserting into or removing from the beginning or middle of an array-backed
collection can require shifting the following items, so that work is O(n).
Appending to a `List<T>` is **amortized O(1)**: most appends are constant
time, but an occasional resize copies the existing items and takes O(n).

```csharp
int[] numbers = { 10, 20, 30 };
int value = numbers[1];             // O(1): index access

var items = new List<int> { 10, 20, 30 };
items.Add(40);                      // Amortized O(1): append
items.Insert(0, 5);                 // O(n): shift existing items
bool found = items.Contains(30);    // O(n): linear search
```

### Stacks and queues

A **stack** is last-in, first-out (LIFO): add and remove at the top. A **queue**
is first-in, first-out (FIFO): add at the back and remove from the front.
Their defining operations do not need to search the collection, so they are
O(1) under the usual implementations.

```csharp
var stack = new Stack<string>();
stack.Push("first");                // O(1) amortized
string top = stack.Peek();          // O(1)
string popped = stack.Pop();        // O(1)

var queue = new Queue<string>();
queue.Enqueue("first");             // O(1) amortized
string front = queue.Peek();        // O(1)
string dequeued = queue.Dequeue();  // O(1)
```

The amortized note accounts for the occasional internal resize; across many
operations, the average cost per operation remains constant.

### Singly and doubly linked lists

A linked list stores each item in a node that points to another node. It does
not provide direct index access: reaching the item at position `i` requires
traversing nodes, which is O(n). Searching for a value is also O(n).

Adding or removing a node is O(1) **once the relevant node or position is
already known**. Finding a position by walking from the head still takes O(n).
A doubly linked list also keeps a link to the previous node, which makes
traversal and node removal more convenient in both directions; it does not
make index access O(1).

```csharp
var list = new LinkedList<int>();
list.AddFirst(10);                  // O(1)
list.AddLast(20);                   // O(1)

LinkedListNode<int>? node = list.Find(20); // O(n): locate the node
if (node is not null)
{
    list.AddAfter(node, 25);        // O(1): position is already known
    list.Remove(node);              // O(1): remove the known node
}
```

## Choosing a structure

- Choose an array or `List<T>` when you need fast index access.
- Choose a stack when you need to process the most recently added item first.
- Choose a queue when you need to process items in arrival order.
- Choose a linked list when you have a node/position and need frequent
  insertions or removals there; it is not automatically faster than a list
  when you first have to search for that position.

The actual performance can also depend on memory allocation and cache
behavior, but these complexity classes are useful for comparing how operations
scale as collections grow.
