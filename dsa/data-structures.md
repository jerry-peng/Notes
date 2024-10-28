<!-- markdownlint-disable MD013 -->

# Data Structures

Data Structures Review by Rob Hess

- [Complexity Analysis](https://docs.google.com/document/d/1WQV7KBKU-7zHu8yb9GpBpM_U4hcFOWeKDsfWI35IJ4o/edit)
- [Dynamic Array](https://docs.google.com/document/d/1mit4eCq_JDTMkKHbh1Gw2FojFN4Ys4XTu58MLyzHmPo/edit)
- [Linked List](https://docs.google.com/document/d/1Vvbw2nOP0gPv_MgoS_mVHgZf9xHqb_raYuAD6d5icoE/edit)
- [Stack, Queue, Deque](https://docs.google.com/document/d/1xDMnd8184MtkgEl15f_VWERIY0rulIbWEZh_8TyNfgg/edit)
- [Iterator](https://docs.google.com/document/d/118fcC6eiszQBE-PcyyRJ25mA4SZqOd7G4-R6oDJ24i4/edit)
- [Ordered Array and Binary Search](https://docs.google.com/document/d/1e6mOMTUdXTz6fGbmXmNwRqRsPUsKXVtlZ0bZDVIhi6M/edit)
- [Binary Search Tree](https://docs.google.com/document/d/1r_yRycWr4UnT8rGvCs7p_xGCCEelU64NOcphKu49O7Y/edit)
- [Binary Tree Traversal](https://docs.google.com/document/d/13OrIFv9CUOWCU7DYDqtoOvVRqKnz8_hCVjKqIpAYymA/edit)
- [AVL Tree](https://docs.google.com/document/d/1m-6XSKWGsU7ofqE2xupWYXThE-SFjV9xWhUU0V2qKQ8/edit)
- [Map and Hash Table](https://docs.google.com/document/d/1bsVlTCeeSDht7KBj2aRZrL0r0MZTwY8Whxwwia8qyyw/edit)
- [Graphs](https://docs.google.com/document/d/1Qf44u-JzfUOGScRSh7Z0J8_cgvMzgR9GKOXLAoX_0Tg/edit)
- [Priority Queues and Heap](https://docs.google.com/document/d/1j_H0v6kWLx81Jz-Y73_zfqycv2cbAxFvA4BhpuuQlr0/edit)

## Complexities

| Data Structure           | Access            | Search            | Insertion              | Deletion          |
| ------------------------ | ----------------- | ----------------- | ---------------------- | ----------------- |
| Dynamic Array            | O(1)              | O(n)              | O(1) Amortized -> O(n) | O(n)              |
| Linked List, Stack/Queue | O(n)              | O(n)              | O(1)                   | O(1)              |
| Binary Search Tree       | O(log(n)) -> O(n) | O(log(n)) -> O(n) | O(log(n)) -> O(n)      | O(log(n)) -> O(n) |
| AVL Tree                 | O(log(n))         | O(log(n))         | O(log(n))              | O(log(n))         |
| Hash Table               | N/A               | O(1) - O(n)       | O(1) - O(n)            | O(1) - O(n)       |
| Priority Queue           | N/A               | N/A               | O(log(n))              | O(log(n))         |

## Dynamic Array

Dynamic array expands once capacity is reached.
Insertion complexity is amortized O(1).
Search and deletion requires traversal, complexity is O(n).

### Linked List

Linked List: nodes are linked using pointers. Types: singly-linked list, doubly-linked list, circular linked list.
Search and access requires traversal, complexity O(n).
Insertion and deletion to head/tail requires constant time operation, complexity O(1).

### Stack

Stack is FILO (first in last out). Usually implemented using linked list.

### Queue

Queue is FIFO (first in first out). Usually implemented using linked list.

### Ordered Array

Ordered array is an array whose elements are in sorted order, which allows efficient implementations of other data types such as set.
Set union complexity O(n).
Set intersection complexity O(n).

Due to nature of ordered array, insertion/deletion/search complexity is O(log(n)) with help of binary search. Access is still O(1)

### Binary Search Tree (BST)

Search is O(log(n)). Go to left subtree if val < node.val, else go to right subtree
Insertion is O(log(n)). Similar to search, append a new node in the end
Deletion is O(log(n)). Similar to search, remove node when found. Attach child if node has one child, otherwise, find inorder successor.
Traversal: preorder, inorder, postorder.
Inorder traversal return the sorted node values.
BFS uses queue, while DFS uses stack.

### AVL Tree

AVL Tree is a self-balancing binary search tree. Each node keeps track of height.
Search is O(log(n)), same as BST.
Insertion/Deletion are also similar to BST, however, tree needs to be balanced after operations using rotations if imbalance is found.
Insertion/Deletion complexity is O(log(n))

#### Imbalance Scenarios

left-left: rotate right on node
left-right: rotate left on node.left, then rotate right on node
right-left: rotate right on node.right, then rotate left on node
right-right: rotate left on node

### Trie

Trie consists of multiple branches, each represents a possible character of keys. Each node contains ALPHABET_SIZE amount of pointers to other nodes, as well as a boolean to determine leaf nodes.

Search/Insertion time: complexity is O(k), space complexity is O(ALPHABET*SIZE \_k* N), where k is key length, N is number of keys in trie.
Deletion time complexity O(k)
Deletion operations

- Key does not exist in trie, trie not modified
- Key is unique, delete all nodes
- Key is prefix of another key, unmark leaf node
- Key includes at least another key as prefix. Delete nodes until first node with over two children are found.

### Hash Table

hash = hash_function(key)
index = hash % array_size

Average complexity O(1). Worst-case complexity O(n) when resizing.

**Collision resolution with chaining**, which involves storing a collection of elements at each index in the hash table array, called bucket or chain.
load factor lambda = n/m. The bigger the load factor, the slower the hash table.
To maintain performance, increase bucket counts once load factor reach a certain limit, re-compute hash function for each element in hash table.
Complexity for all operation is O(lambda), if bucket counts is adjusted, complexity becomes O(1).

**Collision resolution with open addressing**, which involves probing for empty spot in the hash table if a collision occurs.
Probing method:

- Linear Probing (i = i + 1)
- Quadratic Probing (i = i + j^2)
- Double Hashing (i + j \* h2(key)) where h2 is a second hashing function

When reaching end of array, simply wrap around the array.
When removing element, add a tombstone value, which can be replaced when adding a new entry, but it doesn’t halt search.
Problem: clustering, quadratic probing and double hashing can help to reduce clustering.
The load factor cannot exceed 1.

Complexity analysis:
probability of first, second, third probe success: p = (m - n)/m, p = (m - n)/(m - 1), p = (m - n)/(m - 2)
… (geometric distribution)
So the number of probe required to success = 1/p = 1/((m - n)/m) = 1/(1 - n/m) = 1/(1-lambda).
Assuming we limit the load factor to a constant, complexity becomes O(1).

### Graph

Two ways to represent a graph:

- Adjacent list, in which each vertex stores a list of adjacent vertices
- Adjacent matrix, which is a 2d array where rows/columns represent vertices

Space complexity:

- Adjacent list: O(|V| + |E|)
- Adjacent matrix: O(|V|^2)

The adjacent list is more space efficient when the graph is sparse (when it has relatively few edges).

BFS uses queue, and DFS uses stack.

DFS is **backtracking** search, but can become lost down an infinite graph.
BFS is complete and optimal, but it takes a long time if solution is keep in graph.
Both algorithms have O(V) space complexity in the worst case, but BFS may take more in practice.
If graph has high branching factor, BFS can take a lot of memory.

**Dijkstra’s algorithm**

```
visited = map(key=v, val=minDist)
initialize priority queue with starting vertice
while queue is not empty
- v, d = queue.pop
- if v not in visited
    - add v,d to visited
    - for each successor of v
        - add to queue with added distance
```

Complexity O(|E| log|E|)
The log|E| comes from the priority queue operation

### Priority Queue

A data structure that associates a priority value with each element, resembles complete tree.
min/max heap: every node’s value is less/greater than its children.

Insertion operation insert node to next open spot, then percolate to parents. Complexity: O(log(n))
Deletion operation removes the min/max value, the root, and replacing it with the last item in heap, and then percolate to children. Complexity: O(log(n))

Building a heap from an arbitrary array: Iterate through the array, if not heap structure, move percolate node down until heap structure is maintained.
Time complexity: O(nlog(n))
Space complexity: O(1)

#### heapsort

```
- build heap from array
- repeat until all elements are swapped to the end
    - swap the last element with first element, reduce the range of heap on array by 1.
    - percolate the first element in heap
```

Time complexity: O(nlog(n))
Space complexity: O(1)
