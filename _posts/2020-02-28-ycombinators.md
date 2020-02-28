---
layout: post
title: "What the hell is a Y Combinator anyway?"
date: 2020-02-20
---

A storm is raging outside in London, and the world is on the brink of 
a Coronavirus pandemic, but you have always wanted to know just what a Y
Combinator was didn't you? The phrase has been popularised by the startup
school of the same name, and there's actually some very interesting mathematics
behind it.

## 1) What is it for?

It helps to define recursive functions in the lambda calculus.
Let's break that down. Recursion is the cornerstone of control flow in the
functional style of programming. However, it's not possible to define recursive
functions in the lambda calculus. The lambda calculus is just a set of simple
rules that underpin functional programming, that allows one to express 
arbitrary computations in terms of the application of functions to inputs and
get outputs, it's so simple that it doesn't make space for recursive
functions.

## 2) Lightspeed Lambda Calculus.

The lambda abstraction allows you to express an anonymous function in the lambda
calculus, and will be very familiar with Python programmers who've used Python's
anonymous functions.

Consider the increment function written using Python lambdas,

```python
lambda x: x + 1
```
And an equivalent definition using the lambda calculus (with a dyadic addition
operator).

$$
\lambda x . x + 1
$$


You can see the obvious resemblence. The variable 'x' is bound to the function
in the sense that it's not making reference to an 'x' defined elsewhere in your
lambda expression, (or your code if you were programming).


## 3) Reduction in the Lambda Calculus

Just like in other forms of calculus (in my definition, justmethods of calculating things) we need
some way of simplfying expressions we write down. The lambda calculus defines 3 basic ways of doing this, and they are called 'reduction' methods.

$\alpha$ reduction basically says that you can rename variables, and your 
lambda expressions' meaning remains unchanged.

$$
\lambda x. x \rightarrow \lambda y. y
$$

$\beta$ reduction says that when you apply your lambda abstraction to an
expression, all free instances in your abstraction's expression of the abstraction's parameter are replaced with the new expression. That's kind of
a mouthful, but it's easy to see visually.

$$
(\lambda x. x + 1) \> 4 \rightarrow 4+1 = 5
$$

This looks a lot like (and in fact is) like currying a function for
partial application. The instances of 'x' in the lambda abstraction's expression (x+1) are replaced with 4.

Finally there's $\eta$ reduction. Which states that when there are no bound
instances of the lambda abstraction's parameter in it's expression and it's
expression is a function, the following identity holds,

$$
\lambda x. F \rightarrow F
$$

## 4) The Y Combinator

Believe it or not, that's enough to at least see where the Y Combinator comes
from, if not understand it particularly deeply. Our lambda calculus helps us
write functions in terms of inputs and outputs, and we also have a way to
simplify (reduce) complex lambda abstractions, into simpler ones (normalise).

But in functional programming we face a problem, which is that control flow
is often defined by recursion, and we've no way to account for this using the
simple tools above.

Consider this function written recursively to find factorials,

```python
def FAC(n):
    if n == 0:
        return 1
    return n*FAC (n-1)
```
The lambda expression for this is

$$
\text{FAC} = (\lambda n. \text{if} \> (= \> n \> 0) \> 1 \> (* \> n \> (\text{FAC} (- \> n \> 1))))
$$

This admittedly is a little more complicated than the examples above. But by
noting that the arithmetic operators (+/=/* etc) are partially applied to two
arguments, it should be possible to read that it's doing the same operation
as the python code above.

Let's simplify this by writing it as,

$$
\text{FAC} = \lambda n. (...\text{FAC}...)
$$

We can do a reverse $\beta$ reduction (known as a $\beta$ 'abstraction' - confusingly) on this to get,

$$
\text{FAC} = (\lambda \text{fac}. \>  (\lambda n. \> (...\text{fac}...))) \text{FAC}
$$

which you can verify. Why does this help, this looks worse! However, we can
now see that the above expression is of the form,

$$
\text{FAC} = \text{H} \> \text{FAC}
$$

Where $\text{H}$ is some function that is an ordinary lambda abstraction with no recursion involved. The recursion is defined entirely outside H. If we feed the input $\text{FAC}$ into
this function we return $\text{FAC}$. Compare with ordinary algebra, this is like feeding 0 into the function f(x) = x. In mathematics, this is known as a
fixed point.

So we now know that the $\text{FAC}$ function must be a fixed point of $\text{H}$ whatever it is. So we now are looking for a fixed point of $\text{H}$ in
order to write our algorithm in the lambda calculus. We don't know how to do
this, but let's pretend that we do. Let's invent a function $\text{Y}$ that
takes any other function, and returns its fixed point. This allows us to write
the following expression by definition of $\text{Y}$, 

$$
\text{Y H} = \text{H (Y H)}
$$

this $\text{Y}$ is the famous Y combinator, great but we're not done yet. We
are yet to see how this effects recursion. The magic is that Y can be expressed
without recursion. Despite the above expression seemingly showing recursion
in it's purest form.

In fact, it's shown that $\text{Y}$ actually has the following non-recursive definition,

$$
\text{Y} = (\lambda h. \> (\lambda x. h (x \> x))) (\lambda x. h (x \> x)))
$$

We can really easily verify this, applying the reduction rules above to show
that it satisfies the recursive property.

$$
\text{Y H}= (\lambda h. \> (\lambda x. h (x \> x))) (\lambda x. h (x \> x))) \> \text{H}
$$

Using $\beta$ reduction

$$
\rightarrow (\lambda x. \text{H} (x \> x))) (\lambda x. \text{H} (x \> x))
$$

Using $\beta$ reduction again,

$$
\rightarrow \text{H} ((\lambda x. \text{H} (x \> x))) (\lambda x. \text{H} (x \> x)))
$$

Which is the same as,

$$
= \text{H (Y H)}
$$

## Conclusion

There you have it, the Y Combinator, or as it's also known the fixed point
combinator - for obvious reasons. As I've tried to show you, it allows you to
define recursive functions in the lambda calculus, and that really is it.
Somewhat magically, it has a non-recursive definition, and it's also very
curious that it can be written in the lambda calculus as a result. However,
understanding its more complex properties is still definitely beyond my
current skill level, and more something to ponder about at the moment.
