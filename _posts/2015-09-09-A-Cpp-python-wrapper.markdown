---
layout: post
title: Writing a Python wrapper for C++ object
tags: c++ cython python
categories: code
---

# Writing a simple Python wrapper for C++

I wanted to write what is essentially a wrapper function for some C++ code. Looking around the web turned up some results on Python's ctype utility (native to Python), the Boost::Python C++ libraries, and the Cython package, which provides C-like functionality to Python. I went with Cython in the end due to limtations with ctypes and warnings about magic in the boost library.

The Cython approach also is completely non-interfeing with the C++ code -- i.e. you don't have to go messing with your C++ source files or wrapping them in extern "C" { }-type braces, like you do in ctype, and strikes me as a awkward to go around modifying your C++ code.

You need to have the Cython and distutils modules installed with your Python distribution for this. Examples here use Python 2.7, but there's no reason I know of why Python 3.x won't work either.

## The C++ program

For this example, I'm using a little C++ program called Rectangle.cpp which just calculates the area of a rectangle from a Rectangle object. The example is basically lifted from the Cython docs, but the explanation is padded out a bit more with working scripts and source files. (Unlike the cython.org example which I found almost impossible to understand)

**Rectangle.cpp**
{% highlight cpp %}
#include "Rectangle.hpp"

namespace shapes 
{ 
  Rectangle::Rectangle(int X0, int Y0, int X1, int Y1) 
  { 
    x0 = X0; y0 = Y0; x1 = X1; y1 = Y1; 
  } 
  Rectangle::~Rectangle() { } 
  
  int Rectangle::getLength() { return (x1 - x0); } 
  int Rectangle::getHeight() { return (y1 - y0); } 
  int Rectangle::getArea() { return (x1 - x0) * (y1 - y0); } 
  void Rectangle::move(int dx, int dy) { x0 += dx; y0 += dy; x1 += dx; y1 += dy; }
}
{% endhighlight %}

**Rectangle.hpp**
{% highlight cpp %}
namespace shapes { class Rectangle { public: int x0, y0, x1, y1; Rectangle(int x0, int y0, int x1, int y1); ~Rectangle(); int getLength(); int getHeight(); int getArea(); void move(int dx, int dy); }; }
{% endhighlight %}

##  The Python (and Cython) files
From the python side of things, you'll need 3 files for this set up:
0. The rectangle_wrapper.pyx cython file.
1. The setup.py file.
2. For testing purposes, the test.py file.

The Cython file rectangle_wrapper.pyx is the Cython code. Cython code means C-like code written directly in a Python-like syntax. For this purpose, it is the glue between our C++ source code and our Python script which we wantto use to call the C++ functions. The Cython file is a go-between for Python and C++.

The setup.py file will handle the compilation of our C++ and Cython code (no makefiles here!). It will build us a .cpp file from the Cython file, and a shared library file that we can import into python scripts.

**rectangle_wrapper.pyx**
{% highlight python %}
This is a Cython file and extracts the relevant classes from the C++ header file.
In [ ]:
# distutils: language = c++
# distutils: sources = rectangle.cpp

cdef extern from "Rectangle.hpp" namespace "shapes":
    cdef cppclass Rectangle:
        Rectangle(int, int, int, int)
        int x0, y0, x1, y1
        int getLength()
        int getHeight()
        int getArea()
        void move(int, int)

cdef class PyRectangle:
    cdef Rectangle *thisptr      # hold a C++ instance which we're wrapping
    def __cinit__(self, int x0, int y0, int x1, int y1):
        self.thisptr = new Rectangle(x0, y0, x1, y1)
    def __dealloc__(self):
        del self.thisptr
    def getLength(self):
        return self.thisptr.getLength()
    def getHeight(self):
        return self.thisptr.getHeight()
    def getArea(self):
        return self.thisptr.getArea()
    def move(self, dx, dy):
        self.thisptr.move(dx, dy)
{% endhighlight %}

**setup.py**
{% highlight python %}
from distutils.core import setup
from distutils.extension import Extension
from Cython.Distutils import build_ext

setup(ext_modules=[Extension("rectangle_wrapper", 
                             ["rectangle_wrapper.pyx", 
                              "Rectangle.cpp"], language="c++",)],
      cmdclass = {'build_ext': build_ext})
{% endhighlight %}

You now have all the files needed to build the module. You can build everything using the setup.py script by doing:
`python setup.py build_ext --inplace`

This generates two extra files: the .cpp source code file and the linked library file (.so in linux.) You can now run the test.py file below or experiment with the module in an interactive console. Note that this does not install the module into your python installation directory -- you need to run the script from the same directory as your linked library files, or add the directory to the pythonpath.

**test.py**
{% highlight python %}
#Note how you can just import the new library like a python module. The syntax in the Cython file has given us an easy to use python interface to our C++ Rectangle class.

import rectangle_wrapper

# initialise a rectangle object with x0, y0, x1, y1 coords
my_rectangle = rectangle_wrapper.PyRectangle(2,4,6,8)

print my_rectangle.getLength()
print my_rectangle.getHeight()
print my_rectangle.getArea()
{% endhighlight %}
