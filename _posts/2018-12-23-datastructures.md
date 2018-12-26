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
**stacks**, **queues** and **rooted trees**.

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
    """Node to store data, and pointer to next node in linked list
        as well as a pointer to the previous node."""
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

### Binary Search Trees

Binary search trees have the additional property that all child nodes to
the left of a given node contain data in of smaller or equal value to the
datum in that node, and all nodes to it's right contain data of greater
value.


```python
class Node:
    def __init(self, data):
        self.parent = None
        self.left_child = None
        self.right_child = None
        self.data = data
```

I've only bothered implementing the `Node` object, and only partially out
of laziness. Firstly it's unlikely that you'll ever have to implement a
full binary in

## Hash Tables




