---
layout: post
title: Multidimensional Arrays in C++
tags: arrays, c++
---

## Multidimensional Arrays in C++
C++ does not have a native way of doing multidimensional arrays like some other languages do. There are several ways to workaround this though. Here are some ideas:

### Nested Vector method
Using the `std::vector` object, nest one vector within another like so:
{% highlight cpp %}
#include <vector>

std::vector< std::vector<int> > MyArray;

// You can index this with normal array syntax:
MyArray[i][j] = x;

{% endhighlight %}

Note: the above array has not been initialised with anything, nor does it have a predefined size at compile time as vectors can be allocated dynamically, therefore so can our makeshift 2D Array. It's effectively a vector of vectors with type int.

### Nested array (`C++11`) method
C++11 has a new feature in the standard library: `std::array`. Unfortunately it also does not support multidimenional arrays, but you can nest one within another.
{% highlight cpp %}
#include <array>

std::array< std::array<float, 3>, 3 > MyArray  = { { {5, 8, 2}, {8, 3, 1}, {5, 3, 9} } };
{% endhighlight %}

This is a 3x3 array as we've specified the dimensions in the declaration. We've also intialised the values in the array. Only available with C++11.


## Using the Template Numerical Toolkit (TNT)
Eventually, the workarounds above can get a bit tiresome, or difficult to read using higher dimensional arrays. In LSDTopoTools, we use a library called the Template Numerical Toolkit, which adds support for multidimensional arrays, without the awkward looking syntax used above. The 'user-friendly' documentation is a bit sparse, but the basic examples are [here](http://math.nist.gov/tnt/examples.html). 

The typical way to declare and intialise a TNT array is like so:

{% highlight cpp %}

TNT::Array2D <float> mynewarray;  // Empty 2d array
TNT::Array2D <float> myotherarray(4,5);  // 4x5 array
TNT::Array2D <float> mythirdarray(3,3, 0.0); // Initialise zeros.

{% endhighlight %}

It's not mentioned in the brief tutorial, but you can also do it like this:

{% highlight cpp %}
MyArray = TNT::Array2D <float> (3,3);
{% endhighlight %}

Note that assigning ArrayA to ArrayB only produces a shallow copy (a view of the other array). To get a deep copy, you have to do `B = A.copy()`. Also, in the LSDTT code, we tend to use a namespace declaration to avoid typing `TNT::Array...` all the time:

{% highlight cpp %}
#include "TNT/tnt.h"
using namespace TNT;

Array2D <int> somearray(3,3);
{% endhighlight %}

## Other Methods
You could define your own template (good luck!) or use C-style arrays (ugh...).

## Other Libraries
In the LSDTT code you might also see the Matrix Template Library (MTL). This is another matrix/array library with some more powerful functions for doing matrix algebra. There's also the [Eigen](http://eigen.tuxfamily.org/dox/) library, which can do a lot of things that the TNT can't, but is a much bigger library.



