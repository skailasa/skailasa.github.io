---
layout: post
title: Bitwise Operators
date: 2018-10-25
---

As a Physicist turned coder there are obviously many gaps in my knowledge. One of those, which I only really considered today, regards bitwise operators. It turns out they're a lot simpler than I thought, and have been avoiding them needlessly for the past couple of years since I started coding. 

Here's a quick summary, containing all you need to know to go out and frolic with these marvelous litle tools.

Unlike ordinary mathematical operators which operate on things we humans interpret as decimal numbers (integers and floats etc), e.g. '+, -, /, *' bitwise operators operate on individual bits. There are 6 useful bitwise operators, let's go through them one by one:

1) AND (`&`)
This takes two equal length binary representations of numbers, and performs a "logical and" on each pair of bits:

This is much easier to understand than it sounds from that scary sentence nicked from wikipedia, it basically means that the following truth table is implemented for a given pair of bits, which as we know in binary can only be 1 or 0:

| `A` | `B` | `A & B` |
| :---: | :---: | :---: | 
| 0 | 0 | 0 |
| 1 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 1 | 1 |

So for two input bits `A` and `B` we get the output bit shown by `A & B`

e.g. Taking `1 & 2` actually corresponds to:

```
AND 10  # decimal 2
    01  # decimal 1
    = 00  # decimal 0
```

Piece of cake right?


2) OR (`|`)
Similarly, there's a truth table for the OR operator. But like AND, the name is actaully fairly revealing. It implies that we get a value of true (or `1`) if one or the other  or both of the bits set to True! Resulting in the following truth table:


| `A` | `B` | `A & B` |
| :---: | :---: | :---: | 
| 0 | 0 | 0 |
| 1 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 1 | 1 |

```
OR 10  # decimal 2
   01  # decimal 1
   = 11  # decimal 3
```


3) NOT (`~`)
NOT basically flips a bit, or 'performs a logical negation' on it. Bits that were 0 become 1, and vice versa. This results in the following truth table:

| `A` | `~A` |
| :---: | :---: | 
| 0 | 1 |
| 1 | 0 |

This is probably the easiest one to understand conceptually, but for fellow Pythonistas beware there's an addtional bit of trickery that you have to keep in mind.
However we'll come back to this at the end having covered shifts `>>` and `<<`.


4) XOR (`^`)
The Exlusive Or (XOR) flips bits when either one in a given input are True, but not when BOTH are true:


| `A` | `B` | `A & B` |
| :---: | :---: | :---: | 
| 0 | 0 | 0 |
| 1 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 1 | 0 |

Simples.

5) Shifts `>>` and `<<`
Literally allows you to shift bits up or down a specified amount of places

e.g.
```
1 << 1 = 2
1 << 2 = 4
```

Now we can return to the aforementioned problem with NOT in Python. Python allows this operation on signed integers, meaning that the results are not immediately interpretable.
For example,

```python
>>> ~1
-2
```

In order to implement a NOT operation we have to use the bit shift operators to make our integer unsigned, at least until Python get their shit together.

```python
def bit_not(n, numbits=8):
    return (1 << numbits) - 1 - n
```

Now we'd get:

```python
>>> bit_not(1, 2)  # ~01 in binary
2  # 10 in binary
```

6) The order of precedence of bitwise operators is as follows:

    1. `~`
    2. `<<` and `>>`
    3. `&`
    4. `^`
    5. `|`


Ok, so now we've covered that, where can one actually use these? Well, I came across them when practising an interview question, which I just didn't understand.

Consider this list [1, 2, 2, 3, 4, 5, 4, 5], one knows that all digits are repeated but for one particular digit - write a function (in linear time) to find this digit.

It appears incredibly easy at first, must be some kind of loop...
It can't be nested, because then we'd get N^2 complexity ...
Can we use bitwise operators? Yes!

By noticing that the XOR of any number with itself is ALWAYS 0, and the XOR between any number and 0 is the number itself, we can simple iterate through the list as follows:

```python
_list = [1, 2, 2, 3, 4, 5, 4, 5]

>>> res = 0
>>> for l in _list:
    ... res = res ^ l

>>> print(res)
1
```

How good is that!

