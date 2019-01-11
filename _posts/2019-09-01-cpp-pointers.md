---
layout: post
title: "C++ Pointers and References"
date: 2019-01-09
---

I've recently begun the journey from calm Python pastures to becoming a full
stack programmer. This involves learning a lot of new technologies, and after
teaching myself how to code reasonably well in Python I thought nothing of
learning a much more involved technology like C++. This might seem futile, or
pointless from the perspective of other developers whose focus may be on making
dollar ASAP. However C++ comes with the advantage of offering peaks behind
the compiler to begin to understand how your calculations are being crunched,
so a good understanding will be crucial if you are to ever call yourself an
expert programmer. Plus many other object-oriented languages are probably trivial
once you've wrestled with a massive C++ project, so I tell myself.

In any case I'm going to tackle one of the first concepts that seemed novel to 
me in my study which is that of pointers and references, and specifically how
C++ as a language handles these.


## Pointers

Just as in Python, C++ allows one to assign variables to some bit of memory, in 
C++ these are also typed:

```c++
int x = 7;
```

Assigns the variable `x` with the integer value `7`. We can also 'point' to the
location in memory where a value is stored, without restoring it somewhere else.
Python actually handles this for you implicitly whenever you use two variables
to refer to the same value (they point to the same location in memory) but in
C++ as with many other things one has to do this themselves. Pointers are also
strongly typed, much like variables.

```c++
int *ip;
```

We can then use this 'integer pointer' to point to a memory address containing
an integer:

```c++
ip = &x;
```

Here the ampersand is called the reference operator (or the address of an operator).
It returns the address of an object, for assignment to pointers.

We can 'dereference' pointers, returning the value they point to, by using the
pointer 'dereference' operator:

```c++
y = *ip;
```

Which is the same as the pointer initialisation operator, but different by
context.

## References

Consider the following code, there's quite a bit to unpack here. Firstly notice
how there's no need to use an asterisk to dereference a reference. `y` is just
called normally. Secondly, once a reference is defined **it cannot be changed**.
So when I set `y = 42` it doesn't 'change the value of `y`' it simply changes `x`!

```c++
#include <cstdio>

using namespace std;

int main() {
    int x = 7;
    int &y = x;
    printf("The value of x is %d\n", x);
    printf("The value of y is %d\n", y);
    y = 42;
    printf("The value of x is %d\n", x);
}
```

# Examples

So, with that basic knowledge in mind, let's go on a whirlwind tour of where this
comes into play in the C++ language.

## Pointing to/referencing arrays

Arrays are a primitive in C++, and very similar to the Python type. 

```c++
// Initialised array that can contain 5 values
int ia[5];

// Can assign by index like in Python
ia[0] = 1;

// Can also index elements as if the array was a pointer
// This statement is exactly the same as the previous statement
*ia = 1;

// Integer pointer pointing to an array
int *ip = ia;

// The address operator isn't needed above, because arrays can be
// Accessed like pointers. The following statement assigns a value to
// The first element of the array. Because the pointer was initialised
// to the address of the array, it points to the first element.
*ip = 2;
++ip;

// Can incremement the pointer to point to the second element! C++ pointers
// Are strongly typed, so it knows the size of the element it's pointing to
// The following statement assigns the value 3 to the second element in
// The array.
*ip = 3;

// Shortcut, assigning 4 to the 3rd array element:
*(++ip) = 4;
```

Using the above techniques, we can use pointers to loop through C++ array
primitives:

```c++
#include <cstdio>
using namespace std;

int main() 
{
    char s[] = "Sri";
    printf("The string being iterated over is:%s\n", s);
    
    for (char *cp = s; *cp; ++cp) {
        printf("char is %c\n", *cp)
    }
}
/*
Expected output:
char is:S
char is:r
char is:i
*/
```

How does this loop work? Well we've used a trick here, namely that C++ arrays
end with a `0`, which has a bool value of `False`. Therefore the loop basically
iterates whilst the condition is `True`, i.e. until the end of the string, starting
with the first character which is pointed to by the character pointer `*cp`.

## Passing arguments by reference

Pointers and references let you write functions to operate on objects inplace.
This is obviously useful when you have the potential to be copying around
objects of unknown size.

Here's a common usage pattern, consider a function that returns a reference,
and also takes a reference as its argument. What's the expected output?

```c++
#include <cstdio>

using namespace std;

int & f(int & n) {
    return ++n;
}

int main() 
{
    int i = 5;
    printf("f() is %d\n", f(i));
    printf("i is %d\n", i);
}
```

Well, what happens is the function operates on `i` inplace, by operating on
the reference to `i` and returning the same reference. This has obvious use cases
when we don't want to be copying around an object of potentially unknown size.

However there is also a potential for side effects here, from the fact that
unless you look at the function signature there is no way to actually tell from
just the main statement that the variable `i` is changed in place. 

It may be better to make use of pointers where we have side effects, as this
presents itself a little more clearly in the code. However, I've seen this
pattern used throughout the C++ standard library, so I suppose it's all about
personal taste.

Below I've written equivalent code to operate on a variable inplace using
pointers rather than references. As you can see, it makes it a little bit
clearer that the variable `n` is being operated on inplace.

```c++
#include <cstdio>

using namespace std;

int f(int * n) {
    *n += 1;
    return *n;
}

int main()
{
    int n = 5;
    int *np = &n;

    printf("n is %d\n", n);
    printf("f(n) is %d\n", f(np));
    printf("n is now %d\n", n);
    return 0;
}
```

There is a practical reason to do this to, rather than just aesthetic/neat.
Function variables are stored on the 'stack', this is a relatively small space,
passing values to a function causes them to be copied onto the stack. If they
are too large, it can cause the [stack to overflow](https://xkcd.com/979/), and
create security vulnerabilities in your software. This also means that we can't
really return anything particularly large in a C++ function either, as this could
also cause the stack to overflow.

However, how do you get rid of the potential side-effects, if they are not
desired, or intended - especially as your code gets more complicated than what
I've written above? You would use a `const` qualifier to protect your data:


```c++
#include <cstdio>
#include <string>

using namespace std;

void func(const string & s)
{
    printf("the value is %s\n", s.c_str());
}

int main()
{
    string s = "srinath";
    func(s);
    return 0;
}
```

Here, the `const` keyword protects the variable `s` from being changed in the
function `f`, whilst still giving us access to it's value (`srinath`).

## Function pointers

C++ let's you point to functions (they are first-class objects), much like any
other object:


```c++
#include <cstdio>
using namespace std;

void func()
{
    puts("This is the func()")'
}

int main()
{   
    // slightly odd declaration for function pointer
    void (*pfunc)() = func;
    // can then use as normal!
    pfunc();
}
```

Why would you want to do this? Well one pattern I found was the ability to store
a bunch of related functions together (although it could be argued that a class
might be a better approach).


```c++
#include <cstdio>
using namespace std;

void fa()
{
    puts("This is function a");
}

void fb()
{
    puts("This is function b");
}

// store function pointers in an array using initialiser list 
void (*funcs[])() = {fa, fb};

int main()
{
 funcs[0]();
 funcs[1]();
 return 0;   
}
```

## Conclusion

Understanding memory management is probably the biggest step from scripting to
proper compiled languages, and pointers and references are the first stepping
stone on that path. In a future post, I think I'll go through something a bit
more engineering-y in C++, specifically how to set up a development environment
and create a library - as this turns out to be super involved in comparison to
any scripting language like JavaScript or Python.