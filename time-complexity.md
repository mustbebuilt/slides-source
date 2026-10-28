# Time Complexity: O(1), O(n), and O(n log n)

Time complexity describes how the amount of work an operation does grows as
the input grows. Here, **n** is the number of items in a collection.

- **O(1) — constant time:** the work stays roughly the same as `n` grows.
- **O(n) — linear time:** the work grows in direct proportion to `n`. An operation
  may need to inspect or move every item.
- **O(n log n) — linearithmic (or quasilinear) time:** the work grows faster than
  linear but vastly slower than quadratic ($O(n^2)$). It is the characteristic
  benchmark for efficient divide-and-conquer algorithms, notably sorting.

Big O describes growth, not an exact number of seconds. It also describes an
upper bound: an O(n) search might finish on its first check, but in the worst
case it checks all `n` items.

### Growth comparison across complexity classes

| Collection size ($n$) | O(1) | O(n) | O(n log₂ n) | O(n²) |
| --- | --- | --- | --- | --- |
| 10 | 1 | 10 | ~33 | 100 |
| 1,000 | 1 | 1,000 | ~9,966 | 1,000,000 |
| 100,000 | 1 | 100,000 | ~1,660,964 | 10,000,000,000 |
| 1,000,000 | 1 | 1,000,000 | ~19,931,569 | 1,000,000,000,000 |

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

### Sorting operations

| Algorithm / Method | Typical average time | Worst-case time | Auxiliary space | Strategy |
| --- | --- | --- | --- | --- |
| `Array.Sort()` / `List<T>.Sort()` | O(n log n) | O(n log n) | O(log n) | Hybrid Introsort (Quick Sort + Heap Sort fallback) |
| Merge Sort | O(n log n) | O(n log n) | O(n) | Divide and Conquer (halve & merge) |
| Quick Sort | O(n log n) | O(n²) | O(log n) | Divide and Conquer (partition around pivot) |
| Binary Insertion Sort | O(n²) | O(n²) | O(1) | Binary search for slot (O(n log n) comparisons) + array shifts |
| Naive Sorts (Bubble, Selection, Insertion) | O(n²) | O(n²) | O(1) | Nested loops comparing every element |

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

### O(n log n) and divide-and-conquer (sorting)

An **O(n log n)** runtime—often called **linearithmic** or **quasilinear** time—emerges
when an algorithm breaks a problem down into halves (logarithmic depth) and performs
linear work across each level of division.

#### The intuition behind n × log n

To see why $O(n \log n)$ is the standard benchmark for efficient sorting and
divide-and-conquer algorithms, break it into its two constituent factors:

1. **The Logarithmic factor ($\log_2 n$):**
   When an algorithm repeatedly divides a collection of size $n$ in half, the maximum
   number of times it can split before reaching single-element base cases is
   $\log_2 n$.
   - For an array of 8 items: $8 \rightarrow 4 \rightarrow 2 \rightarrow 1$ requires $3$ divisions ($\log_2 8 = 3$).
   - For an array of 1,024 items: only $10$ divisions ($\log_2 1024 = 10$).
   - For an array of 1,000,000 items: only $\approx 20$ divisions ($\log_2 10^6 \approx 19.93$).

2. **The Linear factor ($n$):**
   At each level of the recursion tree, the algorithm must still process all $n$ items:
   - In **Merge Sort**, it compares and merges pairs of sorted subarrays at each level ($O(n)$ work per level).
   - In **Quick Sort**, it scans and partitions elements around a pivot across each level ($O(n)$ work per level on average).

3. **The Multiplier ($n \times \log_2 n$):**
   Multiplying $n$ units of work across $\log_2 n$ tree levels yields **$O(n \log n)$** total operations.

```
Recursion Tree (e.g., Merge Sort with n = 8):

Level 0:  [ . . . . . . . . ]               8 items processed       = 8 operations (n)
                 /        \
Level 1:  [ . . . . ]  [ . . . . ]          4 + 4 items merged      = 8 operations (n)
           /      \      /      \
Level 2:  [ .. ] [ .. ] [ .. ] [ .. ]       2 + 2 + 2 + 2 merged    = 8 operations (n)
          / \    / \    / \    / \
Level 3:  [.] [.] [.] [.] [.] [.] [.] [.]   Base cases (size 1)
          ----------------------------------------------------------------------------
          Height = log₂(8) = 3 levels       Total Work = 3 × 8      = 24 ≈ n log₂(n)
```

#### Why O(n log n) is the theoretical gold standard

Mathematically, any comparison-based sorting algorithm requires at least
$\Omega(n \log n)$ comparisons in the worst case (proven by the decision tree
model, where distinguishing between $n!$ possible permutations requires a tree
depth of $\lceil \log_2(n!) \rceil \approx n \log_2 n - 1.44n$). You cannot sort
faster than $O(n \log n)$ using comparisons alone.

Contrast this with naive sorting algorithms like Bubble Sort, Selection Sort,
or standard Insertion Sort, which compare every element against every other
element ($O(n^2)$):

- Sorting **100,000 items** with an $O(n^2)$ algorithm takes roughly **10 billion** operations.
- Sorting **100,000 items** with an $O(n \log n)$ algorithm takes roughly **1.7 million** operations.
- That is a **~5,800× speedup**.

#### C# sorting in practice

In .NET, `Array.Sort()` and `List<T>.Sort()` use **Introsort** (an adaptive hybrid
algorithm combining Quick Sort, Heap Sort when recursion depth gets too deep,
and Insertion Sort for small partitions). This guarantees $O(n \log n)$ performance
without the worst-case $O(n^2)$ pitfall of basic Quick Sort:

```csharp
var numbers = new List<int> { 45, 12, 89, 3, 27, 64 };

// O(n log n): In-place Introsort (Quick Sort + Heap Sort fallback)
numbers.Sort();

// O(n log n): LINQ OrderBy buffers elements and sorts upon iteration
var sortedAscending = numbers.OrderBy(n => n).ToList();

// Binary search on an already sorted list is O(log n)
int index = numbers.BinarySearch(27); // O(log n)
```

## Choosing a structure

- Choose an array or `List<T>` when you need fast index access.
- Choose a stack when you need to process the most recently added item first.
- Choose a queue when you need to process items in arrival order.
- Choose a linked list when you have a node/position and need frequent
  insertions or removals there; it is not automatically faster than a list
  when you first have to search for that position.
- Expect **O(n log n)** whenever you sort a collection using standard comparison
  methods (`List<T>.Sort()`, `Array.Sort()`, or LINQ `.OrderBy()`). If your application
  requires frequent lookups without paying the $O(n \log n)$ sort cost up front,
  consider a `Dictionary<TKey, TValue>` or `HashSet<T>` for O(1) average lookups.

The actual performance can also depend on memory allocation and cache
behavior, but these complexity classes are useful for comparing how operations
scale as collections grow.
