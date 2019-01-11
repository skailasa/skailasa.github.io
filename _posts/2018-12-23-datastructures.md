---
layout: post
title: "Data Structures for Humans"
date: 2018-12-23
---

Just as there are atomic [algorithms](2018/12/05/sorting-algorithms.html)
there are also atomic data strutures. These essentially amount to different
objects to store data in, each object following a conveniently designed
protocol. In this post I'm going to go through the most common ones that
every coder should just sort of know. Namely: **linked lists**, **hash tables**,
**stacks**, **queues** and **r rooted trees**.

This may sound like a lot for a single post, but as we'll see, a lot of 
the protocols kind of boil do to a single use case - storing data, and 
accessing it conveniently in order to *implement* different algorithms. 
Implementation of algorithms is key here, the design of data structures 
is intrinsically interesting from a maths perspective, but their design is
heavily influenced by the problems you can solve by using them.

Similar to the previous post on [algorithms](2018/12/05/sorting-algorithms.html)
I'm going to go through the above data structures one by one, and write a
simple Python implementation of the protocol. Furthermore, as a Python
programmer I'm also going to write a little bit about how Python implements 
these data structures as language *primitives* (i.e. for free!) as it 
doesn't necessarily strictly adhere to the above protocols as described 
in CS [textbooks](https://en.wikipedia.org/wiki/Introduction_to_Algorithms)
but they do offer some convenient extras that are special to the language.
My aim here is simply to make visible all the moving parts.


## Stacks and Queues

### Stacks

Stacks and queues are well named, and would behave as you would expect
for such objects. Stacks operate under 'Last In First Out' (LIFO), 
like a stack of plates - you would get to the bottom plate by first 
removing the plates on top of it. Whereas queues operate under 'First In
First Out' (FIFO), much like standing in a (well functioning) queue.

Below I've implemented a stack object using the Python list. Stacks
implement two methods to add (push) and delete (pop) data. The pop method
usually doens't accept any arguments as Stacks by their nature follow LIFO.

```python
class Stack:
    def __init__(self):
        """Implement using a list"""
        self._stack = []
 
    def __repr__(self):
        return str(self._stack)
        
    def empty(self):
        """Check whether stack is empty"""
        return not bool(self._stack)
    
    def push(self, v):
        """Add an item to top of stack"""
        self._stack += [v]
    
    def pop(self):
        """Remove item from top of stack"""
        self._stack = self._stack[:-1]
```

Python doesn't offer stack primitives, but they do encourage the use of
Python lists as stacks, as you can perform operations in a LIFO manner:

```python
l = list([1, 2, 3, 4])

# add to top of stack
l.append(5)

# pop from top of stack
l.pop()

# lists are truthy, so can check if empty:
[]  # False
not []  # True

```

### Queues

Again, Python doesn't offer a primitive queue object, so I've implemented
the basic protocol below using a list. Additionally, the delete and insert
operations are called dequeue and enqueue by convention for queues.

```python
class Queue:
    def __init__(self):
        """Implement using a list"""
        self._queue = []
        
    def empty(self):
        """Check if queue is empty"""
        return not bool(self._queue)

    def enqueue(self, v):
        """Add an item to back of the queue"""
        self._queue += [v]

    def dequeue(self):
        """Remove item from front of the queue"""
        self._queue = self._queue[0:]
```

Although Python doesn't offer queue or stack primitives, it does offer a
built-in implementation for FIFO/LIFO objects through the `Queue` module 
that ships with the standard library that fulfill both the simple 
protocols above, with some extras. It would in general be wise to use this
rather than implement your own, as they will have taken care to optimise
the performance of the objects in comparison to the ones I've implemented
above.

What kind of problems can we use this technology for? Pretty much anything
where you want to access data in a consistent way, for 

### Analysis

Stacks and Queues operations to insert and delete data are both $O(1)$. 
This latter statement is a result of the fact that data access occurs in 
a pre-defined manner (LIFO/FIFO). Therefore, when implementing an algorithm
that relies on data access in some pre-defined manner, one should attempt
to use a stack/queue. For example I made use of a queue to implement 
[breadth first search](2018/12/05/sorting-algorithms.html),
where I needed to store the next nodes to consider in order of consideration.

## Linked Lists (Dynamic Arrays)

In most low level programming languages, you have to declare how much 
*stuff* can go in an array before putting in anything in by setting aside
some memory when you create an array object. In doing this though, you
quickly run into problems. Imagine you create an algorithm to process 
the results of some mathematical calculation, each time you get a result
you add it to an array, however setting your calculation off you quickly
eat up all the memory you've allocated to your initial array, and you 
have to create a new array object as a part of your algorithm. This could
quickly get messy, with references to new arrays being created dynamically.
(Singly) Linked Lists, or Dynamic Arrays, get around this problem by allowing you
to extend arrays as you go. One does so by modelling each datum in the 
array as being stored in a node, with pointer to the next datum.

This model is pretty good, though it is a little fiddly to deal with 
inserting and deleting data, the basic methods for a linked list, which
I've implemented below, are again to do with the insertion and removal of
data.

```python
class Node:
    """
    Node to store data, and pointer to next node in linked list
        as well as a pointer to the previous node.
    """
    def __init__(self, data):
        self._next = None
        self.prev = None
        self.data = data


class LinkedList:
    """Implement using Nodes and Pointers"""
    def __init__(self):
        self.head = Node(None)

    def __repr__(self):
        """Some Python sugar to print a representation"""

        x = self.head
        ll = []
        while x.data is not None:
            ll.append(x.data)
            x = x._next

        return str(ll)

    def insert(self, data):
        """Insert an element at the head of the list"""
        node = Node(data)

        node._next = self.head
        self.head.prev = node

        self.head = node
        self.head.prev = Node(None)

    def search(self, data):
        """Return the node that holds a given piece of data"""
        x = self.head

        while x.data != data:
            x = x._next

        return x

    def remove(self, data):
        """Splice out a given node"""

        x = self.search(data)

        x.prev._next = x._next
        x._next.prev = x.prev

        if x.data == self.head.data:
            self.head = x._next
```
This is all well and good, but luckily you probably won't have to ever
fiddle around with a similar implementation of your own, as Python has
you covered with the ever helpful list. Python lists amount to dynamic
arrays which you can (within reason) keep appending data to. They
are fantastic, versatile objects, with many more features than my little
Linked List above. Specifically, my `remove` method fails to account for
nodes with duplicate data, and my `insert` method can't place a node
anywhere but at the head of the linked list. Python lists allow you to 
place items in a list by index, which is much more versatile than the
above - which is the basic protocol for (singly) linked lists.

### Analysis

Linked lists are 'linear', i.e. the way to search through them happens
linearly - one node at a time. Therefore we can reason that the `search`
method, and therefore the `remove` method are both $O(N)$, whereas the
`insert` method is much cheaper as we always insert at the head of the
list, and is $O(1)$. Therefore, these data structures are more useful when
our algorithm needs to store data much more often than it needs to look
it up.

## Rooted Trees

We can use a similar node-pointer model to represent rooted trees,
in the context of this post we'll only consider **binary trees**
and **binary search trees**. As with linked lists, we'll assume each node
contains some data, the remaining attributes are the pointers to the other
nodes, which will inevitable vary with the type of rooted tree.

### Binary Trees

The nodes of binary trees can only point at most to two other nodes,
hence **binary**. We refer to nodes, and the ones they point to as 
'parents' and 'children', respectively,
In terms of implementation, it's a fairly common pattern I've seen
[elsewhere](http://cslibrary.stanford.edu/110/BinaryTrees.html) to specify
binary (search) trees in an abstract manner. Using just a `Node`
object, with pointers to a given node's parent, and two children. 

```python
class Node:
    """Implement a tree by specifying relations between nodes."""
    def __init__(self, data):
        self.parent = None
        self.left_child = None
        self.right_child = None
        self.data = data
```

In terms of manipulating this structure to view/insert/delete data we
can implement common tree-traversal methods. Below, I've implemented a
`lookup` method to find a given node in binary (search) tree using in-order
traversal:

```python
def lookup(node, value):
    """
    Lookup a given value in a binary tree, starting the search at
        a specified root node. Returns True if value is contained,
        within tree, False otherwise.
    """
    if node:
        if (
            node.data == value
            or lookup(node.left_child, value)
            or lookup(node.right_child, value)
        ):
            return True

    return False
```

### Binary Search Trees

Binary search trees have the additional property that all child nodes to
the left of a given node contain data in of smaller or equal value to the
datum in that node, and all nodes to it's right contain data of greater
value. For integers this is obvious, but less so for things like strings.
Where you may have to map the data you're storing back to unique integers
in order to store it. Using this we can write a simple method to insert
data into Binary Search Trees.

```python
# construct a simple BST with empty leaf nodes
n2 = Node(2)
n1 = Node(1)
n3 = Node(3)

n2.left_child = n1
n2.right_child = n3
n1.parent = n2
n3.parent = n2

n3.left_child = Node(None)
n3.right_child = Node(None)
n3.left_child.parent = n3
n3.right_child.parent = n3

n1.right_child = Node(None)
n1.left_child = Node(None)
n1.left_child.parent = n1
n1.right_child.parent = n1

 
def insert(node, value, pointer=None):
    """
    Insert data into Binary Search Trees
        Begin by looking up whether or not a node exists with this value
        in the tree. If it does not, we have to recurse down the tree to
        find an appropriate place to store the value.
    """
    if node and node.data:
        if value <= node.data:
            pointer = node.left_child
            insert(node.left_child, value, pointer)
        else:
            pointer = node.right_child
            insert(node.right_child, value, pointer)

    else:
        pointer.data = value
```

I've used a similar recursive strategy to implement `insert` and `lookup`
above. This is because recursion pops out fairly naturally when manipulating
trees, mostly beacuse we'll want to consider each node of a tree in a
linear manner, often based on a condition, but the data structure is 
itself clearly non-linear.

Python doesn't offer any tree primitives, so are they really that useful?
I think a lot people understimate the importance of trees. Famously the
author of [Homebrew](https://twitter.com/mxcl/status/608682016205344768)
complained about being rejected from Google for 'failing to invert a 
binary tree', which he thought was BS as he had created software that 
pretty much all Mac coders use. Initially I was inclined to side with him,
but I read this 
[blog post](https://thecodebarbarian.com/i-dont-want-to-hire-you-if-you-cant-reverse-a-binary-tree) 
that really changed my mind on the issue.

So why doesn't Python implement tree primitives despite them being so
important in Computer Science? The truth is it does, a more complex abstraction
such as a Python dictionary, or a JSON object is built on top of a tree like
structure. In terms of raw trees, much like my implementation of a linked
list earlier, Python's creators probably thought it was more useful in 
terms of solving problems to wrap the basic abstraction inside a higher
level concept, which when you think about it is. Do you really want to
implement a tree traversal method, every time you want to look up an item
in a `dict` like structure?

### Analysis

My recursive tree methods above go through each node, one by one,
operating on a constructed tree in-place. Therefore, they are of $O(1)$
in space, and of $O(log(N))$ in time. This comes from the number of levels
in a binary tree, which can be found by using a similar argument to that
of [merge sort](2018/12/05/sorting-algorithms.html). But as a binary tree
could degenerate into a linked-list, this time complexity in the worst
case would be $O(N)$. 

## Hash Tables

Hash tables are the most effective way of representing an abstratction
for `dictionary` like objects. i.e. where we want to read, update, and
insert data by reference to some name. In theory names (or hashes) are
difficult to create uniquely (search for hash collisions), but in practice
most hashing algorithms are sound enough to make collisions very rare indeed.
This lookup can be very efficient, $O(1)$, as long as we can guarantee
unique hashes.

I've built a fairly contrived implementation below, using Python lists
as the underlying data structure. I've used lists to handle potential
hash collision, i.e. if a hash for a particular value corresponds to
a hash for some other value, by using a list to store values you can
handle collisions.

```python
def small_hash(value):
    return hash(value) % 100

class HashTable:
    """Simple Hashtable implementation using Python lists"""

    def __init__(self):
        # buffer space
        self._table = [None for i in range(int(10e5))]
        self._values = []

    def __repr__(self):

        l = []
        for v in self._values:
            l.append((v, self._table[small_hash(v)]))

        return str(l)

    def __setitem__(self, value, item):
        key = small_hash(value)

        if value not in self._values:
            self._values.append(value)

        if not self._table[key]:
            self._table[key] = [] 

        self._table[key].add(item)

    def __getitem__(self, value):
        key = small_hash(value)

        return self._table[key]
```

In reality you would never bother implementing the above code in Python
as dictionaries are primitives in the language, and in many other languages
both interpreted and compiled.

### Analysis

If you're hashing algorithm is decent, you should be able to perform
lookups/inserts in $O(1)$ on average. 

## Conclusion

Data structures are a very deep topic, and I've scratched the surface
here with simple implementations, of fairly simple data structures. For
example, the hash table I've implemented is one in which I've not optimised
the hash function to reduce collisions, or considered really the type of
problem I'd wish to solve (and hence the underlying data structure it
should rely on). However, it's a good start for anyone interesting in
studying the topic, and CLRS is really the place to supplemement this
post.