---
layout: post
title: Generators or List comprehensions
tags: python memory optimisation
categories: code
---

# Optimisations with python data structure generation

This post explores some of the differences in using list comprehensions and generator expressions to build lists in Python. We are going to look at the memory and CPU performance using various Python profiling tools and packages.

Firstly a list comprehension can be built in python using:

{% highlight python %}

mylist = [x for x in somedata]

{% endhighlight %}

which constructs the list for every item in ``somedata``. The list comprehension may also be built out of a function that returns an item on return, e.g. ``[x for x in somefunc()]``. The important thing to note in list comprehensions is that the whole list is evaluated at once, this is in contrast to the generator expression which is "short-circuiting" and will exit early if the expression permits it so. Generators can be a useful alternative where an algorithm is likely to finish early if certain conditions are met. For example:

{% highlight python %}

# Using a list comprehension
mybool = any([x for x in somefunc()])

# Using a generator expression
mybool = any(x for x in somefunc())

{% endhighlight %}

In the case of the list comprehension the entire list is evaluated first, and then run through the ``any()`` function. In the generator case, the ``any()`` test is evaluated every iteration of somefunc(), and if it returns true the any test can return early without having to build the entire list.

In theory then, generator expressions offer a potential performance benefit compared to their list comprehension counterparts. But how does it play out in practice?

## Testing

We're going to use an example that builds lists of random strings. Here's a function that returns a random string:

{% highlight python %}
import random
import string

def randomword(length):
   return ''.join(random.choice(string.lowercase) for i in range(length))

{% endhighlight %}

Now we need a function that builds the lists using each method. First the list comprehension way:

{% highlight python %}

def list_strings_comprehension(length):
    list_strings = [randomword(5) for i in range(length)]
    list_strings.sort()
    return list_strings

{% endhighlight %}

And now a function that uses the generator approach:

{% highlight python %}

def list_strings_generator(length):
    listgen_strings = sorted(randomword(5) for i in range(length))
    return listgen_strings

{% endhighlight %}

Let's also create some functions for testing ints, just to see if there is any difference with the data type.

{% highlight python %}
def list_ints_comprehension(length):
    list_ints = [i for i in range(length)]
    list_ints.sort()
    return list_ints

def list_ints_generator(length):
    listgen_ints = sorted(i for i in range(length))
    return listgen_ints

{% endhighlight %}
## timeit

Now we are going to test these methods with the ``timeit`` command, built in to the python interpreter. Using IPython, this can be run using the command: ``%timit [FUNCTION_NAME(ARGS)]``. Help for this command is accessed with ``%timeit?``.

Using our integer list building methods:

{% highlight console %}
%timeit list_ints_comprehension(100000)
#>> 100000 loops, best of 3: 8.74 ms per loop

%timeit list_ints_generator(100000)
#>> 100000 loops, best of 3: 11.2 ms per loop

{% endhighlight %}

So, it would appear at first approximation, the generator approach is slower, at least for building a list of integers this size.

## Memory usage

Now let's investigate the impact on memory use. Memory use is tricky to measure in Python, as objects can have a deeply nested structure, making it diffcult to fully trace the memory footprint of objects. The python interpreter
also performs garbage collection atcertain intervals, meaning it can be difficult to reproduce tests of memory consumption.

First we're going to use the built in ``sys.getsizeof()``

{% highlight python %}

# Eight
listy = list_strings_comprehension(8)
print "Eight Listy: ", sys.getsizeof(listy)

genny = list_strings_generator(8)
print "Eight Genny: ", sys.getsizeof(genny)

#>> Eight Listy:  136
#>> Eight Genny:  168

# Ten
listy = list_strings_comprehension(10)
print "Ten Listy: ", sys.getsizeof(listy)

genny = list_strings_generator(10)
print "Ten Genny: ", sys.getsizeof(genny)

#>> Ten Listy:  200
#>> Ten Genny:  168

# 100
listy = list_strings_comprehension(100)
print "Small Listy: ", sys.getsizeof(listy)

genny = list_strings_generator(100)
print "Small Genny: ", sys.getsizeof(genny)

#>> Small Listy:  920
#>> Small Genny:  992

# 1000
listy = list_strings_comprehension(1000)
print "Medium Listy: ", sys.getsizeof(listy)

genny = list_strings_generator(1000)
print "Medium Genny: ", sys.getsizeof(genny)

#>> Medium Listy:  9032
#>> Medium Genny:  8552

# One million
listy = list_strings_comprehension(1000000)
print "Big Listy: ", sys.getsizeof(listy)

genny = list_strings_generator(1000000)
print "Big Genny: ", sys.getsizeof(genny)

#>> Big Listy:  8697472
#>> Big Genny:  8250176

{% endhighlight %}

Interestingly, the generator performs better in most cases, execpt for the smallest example with eight strings. With larger lists than this, the generator approach consistently outperforms the list comprehension method in terms of its memory footpring, when building lists of strings and measuring with the ``sys.getsizeof()`` function.

## pympler ``asizeof()``

The pympler package is reportedly more accurate at deteriming the true memory footprint of a Python object. USing the ``asizeof()`` method with the same tests as above, we get:

{% highlight python %}
from pympler import asizeof

listy = list_strings_comprehension(1000)
print "Medium Listy: ", asizeof.asizeof(listy)

genny = list_strings_generator(1000)
print "Medium Genny: ", asizeof.asizeof(genny)

#>> Medium Listy:  57032
#>> Medium Genny:  56552

listy = list_strings_comprehension(100000)
print "Big Listy: ", asizeof.asizeof(listy)

genny = list_strings_generator(100000)
print "Big Genny: ", asizeof.asizeof(genny)

#>> Big Listy:  5624472
#>> Big Genny:  5679848

listy = list_strings_comprehension(1000000)
print "Million Listy: ", asizeof.asizeof(listy)

genny = list_strings_generator(1000000)
print "Million Genny: ", asizeof.asizeof(genny)

#>> Million Listy:  56697472
#>> Million Genny:  56250176

{% endhighlight %}

## memory_profiler

Another option is the ``memory_profiler`` package. This provides another IPython magic command: ``%memit``, which can be used like so:

{% highlight python %}
import gc
gc.collect() # Run the garbage collector first.

%memit -i 0.000001 list_strings_comprehension(1000000)
#>> peak memory: 230.33 MiB, increment: 48.00 MiB

gc.collect()
%memit -i 0.000001 list_strings_generator(1000000)
#>> peak memory: 233.61 MiB, increment: 51.27 MiB

{% endhighlight %}

Pympler's ``asizeof()`` says the list comprehension is bigger, ``memory_profiler`` says the generator sees the bigger memory footprint...TBC
