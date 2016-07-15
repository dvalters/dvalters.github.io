---
layout: post
title: Using C libraries and headers in C++ programs
tags: c++ c extern
---

C++ can make use of native C libraries and header files. (As long as there is no incompatible stuff in the C implementation that will not compile as valid C++, there are only a few of these exceptions).

*Example files*
{% highlight console %}
my_cpp_main.cpp
my_c_source.c
my_c_header.h
{% endhighlight %}

A normal `#include "c_header.h"` will not work, however. Instead use the `extern` keyword like so:

{% highlight cpp %}

// normal C++ includes
#include <iostream> // or whatever...

extern "C"
{
  #include "c_header.h"
}

int main()
{
  call_some_func_c_header();

  // blah etc.
}
{% endhighlight %}


Then compile as follows:

`g++ -c my_c_source.c my_main.cpp -o myExec.out`

(*remember to have the sources in the right order and before the executable!*)
