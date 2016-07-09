---
layout: post
title: Forward declaration of classes
categories: articles
tags: classes
---

*Based on email from SMM ages ago and some stuff I read off stack overflow...*

Some of the classes/objects in LSDTopoTools rely on access to other objects in different classes. For example, the some of the `LSDIndexRaster` member functions take `LSDRaster` objects as arguments passed to the functions. One way to make these external classes available is to `#include` the relevant header file that conains the class. This is OK, but can lead to problems with circular dependencies if two header files reference each other's classes. 

**LSDIndexRaster.hpp**
{% highlight cpp %}
#include "LSDRaster.hpp"
/// ...
class LSDIndexRaster
{
  // ..stuff
};
{% endhighlight %}

**LSDRaster.hpp**
{% highlight cpp %}
#include "LSDIndexRaster.hpp" //Oh no!

class LSDRaster
{
  public: 
  // ...
};
{% end highlight}

The solutuion to this cyclical problem is to use a forward-declaration of the class LSDRaster in the LSDIndexRaster.hpp file. Forward-declaration just tells the compiler that this class exists -- it doesn't tell the compiler about any of the class details, but it's sufficient for the purposes here. 

**LSDIndexRaster**
{% highlight cpp %}

class LSDRaster; //declaration of LSDRaster class

class LSDIndexRaster
{
  public:

  void someFunc(int x, LSDRaster& object);

  // etc...
};
{% end highlight %}



