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
**stacks**, **queues**, **binary trees**, and **binary search trees**. 
This may sound like a lot for a single post, but as we'll see, a lot of 
the protocols kind of boil do to a single use case - storing data, and 
accessing it conveniently in order to *implement* different algorithms. 
Implementation of algorithms is key here, the design of data structures 
is intrinsically interesting for a maths perspective, but their design is
heavily influenced by the problems you can solve by using them.


Similar to the previous post on [algorithms](2018/12/05/sorting-algorithms.html)
I'm going to go through the above data structures one by one, and write a
simple Python implementation of the protocol. Furthermore, as a Pythonista
I'm also going to write a little section of how Python implements these
data structures as languag *primitives* (i.e. for free!) as it doesn't 
necessarily strictly adhere to the above protocols as described in CS 
[textbooks](https://en.wikipedia.org/wiki/Introduction_to_Algorithms)
but they do offer some convenient extras that are special to the language.
Importantly, I'm going to focus on implementing the protocol in full, 
my aim here is to make visible all the moving parts. Without further adieu
let's get to it.


## Stacks and Queues

### Stacks

Stacks and queues are well named, and would behave as you would expect
for such objects. Stacks operate with under 'Last In First Out' (LIFO), 
like a stack of plates - you would get to the bottom plate by first 
removing the plates on top of it.

Below I've implemented a stack object using the Python list.


```python
class Stack:

```

Python doesn't offer stack primitives, but they do encourage the use of
Python lists as stacks, as you can perform operations in a LIFO manner:


```python
l = list([1, 2, 3, 4])

# add to top of stack
l.append(5)

# pop from top of stack
l.pop()

```

## Linked Lists

In most low level programming languages, you have to declare how much 
*stuff* can go in an array before putting in anything in by setting aside
some memory when you create an array object. In doing this though, you
quickly run into problems. Imagine you create an algorithm to generate
a process some mathematical calculation, and then 

