---
layout: post
title: "Searching & Sorting Algorithms for Humans"
date: 2018-12-05
---

Searching and sorting are atomic concepts in Computer Science, and 
something I didn't realise was going to be such a big thing before
I learned to code. Efficient methods for rifling through data, or sorting
it into order are gold dust. Thinking about the number of times
your implementation is completely reliant on accessing data, you quickly
realise how critical doing these basic operations quickly and accurately
can be in *any* algorithm.
 
As a beginner to CS, I'm going to attempt to summarise and analyse the
major algorithms one should be aware of in this post. I think they are
generally fairly easy to understand and implement, and critical for those
tech interviews where you have to reason between your quick and 
merge sorts.

I'm going to go through them one by one. In general optimal data structure
choice is critical, however wherever I can I'm going to try and implement
the above algorithms on a Python list. This is largely for simplicity. 


## Selection Sort

### Motivation

This is amongst the simplest sorting algorithms, however it's simplicity
comes with a cost, namely it's very expensive in terms of time to compute
in comparison with other more nuanced algorithms.

We'd probably never want to implement this algorithm on it's own, as it's
a bad choice compared to other sorting algorithms, but it could be useful
as a component in more complex sorts.

Let's motivate the algorithm with an example. Consider sorting a deck of
numbered cards. As a human, the easiest way to do this is by finding the
card with the lowest value and placing it at the beginning of the deck,
the proceeding to the card with the second lowest value, etc, until all
cards have been placed.

### Implementation

```python
def smallest(l):
    """
    Find the smallest element of a list, by iterating through the
        whole list.
    """
    idx_smallest = 0
    smallest = l[idx_smallest]
    found = False
    i = 0
    while not found:
        if l[i] <= smallest:
            smallest = l[i]
            idx_smallest = i

        i += 1

        if i == len(l):
            return idx_smallest, smallest

    return idx_smallest, smallest

 
def selection_sort(l):
    """
    Sort by continuously finding the next smallest element and putting
        that at the appropriate index.
    """

    sorted = False
    i = 0

    while not sorted:

        idx_smallest, val_smallest = smallest(l[i:])
        if val_smallest <= l[i]:
            l[i], l[i+idx_smallest] = l[i+idx_smallest], l[i]

        i += 1

        if i == len(l):
            sorted = True

    return l
```

### Analysis

As a minimum this algorithm will operate in O(N^2) in the worst and 
average case. We can see this as being due to the need to iterate through
the whole list (N-i) times at the ith timestep in the sort in order to
find the next smallest element. Giving a leading order complexity of O(N^2)
We can perform the entire sort in-place so we only take up O(1) memory.

## Merge Sort

This is an interview classic. Importantly merge-sort follows the 'divide
and conquer' paradigm for solving problems. This boils down to 'dividing'
your problem into approachable sub-problems, solving these, and combining'
the sub-problem solutions together into a final solution.

Merge sort closely follows this paradigm. Intuitively, it operates by
dividing an n-element list into two subsequences of n/2 elements each,
sorting the subsequences, and finally merging the results. The recursion
has a base case of when you have a list of only a single element which is
already sorted.

Our major motivation here is to save on time-complexity, and the recursive
division of our problem space gives us a clue to it being significantly
less than O(N^2) like for selection sort. However, an implementation
example should clarify this.

To understand the merge operation a didactic example is sorting a deck
of cards. Imagine two piles of cards, picking one up off one pile and
one off the the other pile, comparing the two values and placing face
down in a third pile. We do this until one of the first two piles are
empty, at which point we place the remaining cards from the non-empty
pile down on the third pile.

We can sort by dividing the array to be sorted into two, and recursively
calling sort on the two portions, finally merging the results.

### Implementation

```python
def merge(left, right):
    """
    Iterative merge operation.
    """
    i, j = 0, 0
    result = []

    while i < len(left) and j < len(right):

        if left[i] < right[j]:
            result.append(left[i])
            i+=1

        else:
            result.append(right[j])
            j+=1

    result.extend(left[i:])
    result.extend(right[j:])

    return result


def merge_sort(l):
    """
    Recursively defined sort.
    """
    if len(l) == 1:
        return l

    mid = len(l) // 2
    left = merge_sort(l[:mid])
    right = merge_sort(l[mid:])

    return merge(left, right)
```

### Analysis

The implementation above is very neat, and pretty much directly translated
from the pseudocode in [CLRS](https://en.wikipedia.org/wiki/Introduction_to_Algorithms) 
(a book you should check out beyond this post).

What kind of complexity can we expect with this algorithm? Well, to get
an idea we can estimate the number of elements in the recursion tree. This
will give us an indication of the number of operations taking place, and
therefore a time complexity. Inductively, we can infer that there are
log(N) + 1 levels in the recursion tree. For example, a list of length 1
would have log(1) + 1 = 0 levels, as we only call the `sort` function
once. Furthermore, the complexity of the merge operation is O(N) as we
may have to iterate through the whole list in order to merge. So total
complexity can be seen to be O(N(log(N) + 1)) = O(Nlog(N)). Space
complexity is O(N) due to the need to store the result of the merge
operation.

## Quick Sort

Merge sort has a wasteful bit of memory usage in that we need to keep a
record of the merged results at each step. Quick sort lets us avoid this
by sorting in place. It's worst case runtime is O(N^2), however it has
an average case runtime of O(NlogN) - same as merge sort, and the worst
case can be avoided by a clever strategy on behalf of the programmer,
however this will be much easier to explain in code, so let's start with
the implementation.

Similar to merge sort, quick sort applies the 'divide-and-conquer'
paradigm. However, in order to understand exactly how, we need to
understand the 'partition' procedure first, which is critical to the
implementation of quick sort.


Partitioning is easier to grasp through an example. Consider the following
array:

```python
# unpartitioned array
array = [2, 1, 10, 3, 4, 9, 5]
```

Let's choose the last element 5, as the 'pivot'. We're going to try and
arrange the array such that when 5 is placed in it's correct index all
elements to it's left are smaller than or equal to it, and all elements
to it's right are greater than or equal to it:

```python
# partitioned array
array = [2, 1, 3, 4, 5, 9, 10]
```

Note that the array is still unsorted at this point.

The following function implements this partitioning:

```python
def partition(l, low, high):
    """Partition function capable of operating on sub-arrays"""
    pivot = l[high]
    i = low - 1
    for j in range(low, high):
        if l[j] <= pivot:
            i += 1
            l[i], l[j] = l[j], l[i]

    l[i + 1], l[high] = l[high], l[i + 1]
    
    return i + 1
```

I like to think of this as moving a dividing line up the array from left
to right (essentially the `i` index above), and checking if everything to
the left of this line is smaller than the pivot element, which we've chosen
to be the last element of the array. Once the for loop has completed `i`
will equal the number of swaps, hence elements, that are smaller than the
pivot - therfore will tell us where to place the pivot element, which
happens in the last line. Furthermore, this all happens in place on the array,
we just return the index of the pivot element.

Using this functionality, we can implement quick sort by recursively 
calling quicksort on partitioned arrays. The base case of partitioning an
array with a single/no elements just returning itself.

### Implementation

```python
def quick_sort(l, low, high):
    """Recursive quick sort implementation."""
    if low < high:
        p = partition(l, low, high)
        quick_sort(l, low, p-1)
        quick_sort(l, p+1, high)
```

### Analysis

We can see that we've again used the divide-and-conquer paradigm, where
we divide with the partition function rather than naively down the middle
like with merge sort. But what's the runtime for quick sort? It's doing
quite a lot of complicated stuff, especially the partition function, so
we should be a little more careful in our analysis.

Starting with the partitioning, in the worst case. This occurs when we
the procedure produces a sub-problem to two with n elements and 0 elements
respectively for every partition i.e. our array is anti-sorted `[5, 4, 3, 2, 1]` 
Then the partitioning will be of O(N^2). Therefore in the worst case
quick sort is as rubbish as insertion sort. However, this can be helped
with clever choices of pivot - for example just picking a random pivot
each time, or something that reflects the data you are actually trying to
sort just so that you aren't faced with this problem.

In the best case, we'd get two partitioned sub-problems of equal size.
This is now of the same order of complexity as merge sort, as something
approximately the same is happening, just differently! This results in a
quick sort complexity of O(NlogN).

In fact, as detailed in [CLRS](https://en.wikipedia.org/wiki/Introduction_to_Algorithms)
sub-problems have to be extremely unbalanced to not achieve something
approaching the best case performance, i.e. even if we get two sub-problems
of size 9N/10 and N/10 we'd still get O(NlogN) performance, however this
is definitely something to detail in another post. For now I hope you're
happy to use this as a fact.

## Binary Search

Binary search is very easy to understand and implement, and follows common
sense. Assuming our array is sorted (which we are now armed to implement!). 
We first check the middle item of an array if our element is there we return,
otherwise if our element is greater than this we check the top half using the
same procedure, or vice versa the bottom half, until we find our element.
This is quite naturally defined recursively.

### Implementation

```python
def binary_search(l, element):
    """Recursive binary search implementation."""
    mid = len(l) // 2
    
    if l[mid] == element:
        return mid
    
    elif l[mid] > element:
        binary_search(l[:mid-1], element)

    else:
        binary_search(l[mid+1:], element)
```

### Analysis

We are cutting our problem space in two every time as we head down the
recursion tree. Therefore, the number of operations till we find our
element is the sum of N + N/2 + N/4 + ... + 1, where for N elements we do
2^k operations, where k is the step. Hence we'd have the relationship 2^k = N 
between the number of elements in our array and the number of steps, so
the runtime is O(log(N)).

A good take away from this is that if your problem space is reducing in
size with each step it's complexity is likely to be log-ish.

## Graph Search

It is possible, probably, to implement these algorithms using Python lists
but to be honest it's not worth the effort, as it might just obfuscate
the logic of the algorithm - which is what I'm trying to get across. So
for this section I'm going to mix things up a little, and implement the
following algorithms defined against the following `Graph` data structure
I've just made up. It's important to note that this is a convenience object
and doesn't really serve any purpose other purpose than didactic.

```python
from collections import defaultdict


class Graph:
    """Represent graph using adjacency list"""

    def __init__(self):
        self._graph = defaultdict(list)

    def __iter__(self):
        return iter((k, v) for k, v in self._graph.items())

    def __getitem__(self, item):
        return self._graph[item]

    def __setitem__(self, u, v):
        self._graph[u].append(v)

    def __repr__(self):
        return str(self._graph)


g = Graph()  # create graph object
g['a'] = ['b', 'c', 'd']  # add nodes
```

If you're interested in how graphs are implemented check out [CLRS](https://en.wikipedia.org/wiki/Introduction_to_Algorithms)
, I can't sell this book enough. Here I've chosen to represent it with a
adjacency list, using Python's default dict object, which basically allows
one to have a dict with keys which are lists. Here the keys represent nodes
and the items in their lists represent adjacent nodes.

## Depth First Search

The defining feature of depth first search is that it exhausts a given
route through a graph until it tries the next one. I.e. it will look for
the children of a given node, and then one child's children and so on
until it's reached the end, before moving onto another branch. In terms
of a useful example, this would be a useful algorithm if we wanted to know
when two distant nodes are connected, in contrast to breadth-first search as 
we'll see, as it would prioritise getting to the target node in any way
possible ASAP.  

### Implementation

```python
def visit(v, visited):
    visited[v] = True


def dfs(root, graph, visited=None):
    """Operates on Graph objects"""
    if not isinstance(visited, dict):
        visited = {k: False for k, v in graph}
    
    visit(root, visited)

    for node in graph[root]:
        if not visited[node]:
            dfs(node, graph, visited)
```

### Analysis

Depth-first search visits every vertex once and checks every edge in the
graph once. Therefore, DFS complexity is O(V+E) This assumes that the graph
is represented as an adjacency list.

## Breadth First Search

Consider the motivating example for depth first search, finding whether
two distant nodes are linked, similarly we might want to solve the converse
problem - are two nearby nodes linked - for which depth first search
would be rubbish. It might search most of the graph before realising that
they are in fact next to each other! Breadth first search solves this
problem by checking all of the child nodes of a given node first.

### Implementation

We can implement the algorithm using a queue, a very common error for
candidates is apparently to assume that this can be implemented recursively.
However if you think about it this can't make sense, because you would
need to step side-ways in your recursion tree rather than downwards,
introducing another variable to keep a track of where you are! This could
get very complicated, and it's much much simpler to go iteratively.

```python
from queue import Queue


def visit(v, visited):
    visited[v] = True


def bfs(root, graph, visited=None, queue=None):
    """Operates on Graph objects"""

    if not isinstance(visited, dict):
        visited = {k: False for k, v in graph}
       
    if not isinstance(queue, dict):
        q = Queue()
    
    visit(root, visited)
    
    while q:
        node_to_visit = q.get()

        # visit child nodes, then put them in queue
        for node in graph[node_to_visit]:
            if not visited[node]:
                visit(node, visited)
                q.put(node)
```

### Analysis

This will actually have the same average runtime as breadth first search as
the same operation is taking place. The worst case will depend on your
application though, as discussed above.

## Conclusion

I hope you've enjoyed this whirlwind tour of some concepts in Computer
Science, in all honesty this has served as a revision exericise for me.
However I had fun writing it, and explaining something really makes you
question your own understanding. I will be doing another post in the near
future, doing the same thing but for atomic data structures.