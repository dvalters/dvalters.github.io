---
layout: post
title: Templating the LSDRaster object - DAV
categories: articles
tags: LSDRaster, templates
---

*Don't worry, I haven't actually changed LSDRaster in the trunk*

LSDRaster uses `float`s to store it's data arrays. Floats are fine for more purposes, (they take up less memory than arrays of `double`s for example, and you don't always need the extra precision of double anyway. But I wanted an `LSDRaster` object that used arrays of doubles to maintain compatibility with some other code, so I experimented with the best way of doing this. 

## Extra data members, overloaded functions

The easiest way is just to create a data member in the LSDRaster class that stores an array of doubles, and then write seperate functions to deal with creating this object, and manipulating it. You can overload functions if they take `TNT::Array2D<float>` arguments with the corresponding `<double>` typename. 

Unfortunately, this leads to a lot of code duplication, and extra data members in the LSDRaster class with funny names, `RasterData_dbl`, for example. It's fine, but a bit clunky. 

## Templates

So I figured that it might be possible to create a templated class for LSDRaster. Templates allow a class to be spawned for different data types at run-time. The  `TNT::Array2D<>` class is an example of a templated class, as you specifiy the data type when you create your object, e.g `TNT::Array2D< int >`, or whatever you like.

I had a shot a converting LSDRaster to a template class like so:

{% highlight cpp %}
class LSDIndexRaster;
class LSDShapeTools;

///@brief Main analysis object to interface with other LSD objects.
template<typename T = float> // default typename to float if unspecified
class LSDRaster
{
  public:

/// More declartions...

  protected:
  // Data members...
  TNT::Array2D<T> RasterData;

  // Replaces TNT::Array2D<float> RasterData;

};
{% endhighlight %}

So, in theory, we could now create LSDRaster objects with arbitrary type for the array data (Well, not completely arbitrary - it has to be supported by TNT::Array2D.). If we wanted an array of doubles, we could do:

{% highlight cpp %}
LSDRaster<double> MyDoubleRaster;
// Some integers?
LSDRaster<int> MyIntRaster; // Although we have LSDIndexRaster for this...
// etc...
{% endhighlight %}

This is good so far, but it raises a few issues. Firstly, the declaration for an `LSDRaster` object is now `LSDRaster<typename>` and any instance of `LSDRaster` in the code will not compile as the typename is incomplete, and your compiler will complain. Secondly, you cannot write the implementation of a template class in a separate `.cpp` file, like you can with a normal class. [See this SO answer](http://stackoverflow.com/questions/1724036/splitting-templated-c-classes-into-hpp-cpp-files-is-it-possible) for details. [There are ways of getting round this](http://stackoverflow.com/questions/495021/why-can-templates-only-be-implemented-in-the-header-file), for example by including the `.cpp` file in the header file, and so on.

But anyway, even if we got around the second issue above, there is still the problem that this new LSDRaster<> template has broken everyone's code that contains any use of `LSDRaster obj`. After some more reading, I though it might be possible to use a `typedef` to subvert this, and stop everyone's code from breaking.

{% highlight cpp %}
template<typename T = float> // default typename to float if unspecified
class LSDRaster
{
  public:
  
  typedef LSDRaster<> LSDRaster; 

 //...
};
{% endhighlight %}

`typedef` just says that we want to use a synonym `LSDRaster` to mean `LSDRaster<>` (Incidentally, `LSDRaster<>` without any type given inside the angle brackets, will default to float, as it was specified in the template definition as: `template<typename T = float>`)

Unfortunately, you can't do this within a class - a `typedef` can't have the same name as it's enclosing class, and we need this inside the class because some of our functions take `LSDRaster` objects (or references) as arguments. You also have to put the `typedef` in all the other classes that use `LSDRaster` objects. 

The only way of getting around this would be to give the LSDRaster class a different name, something like:

{% highlight cpp %}
template<typename T = float>
class LSDBaseRaster
{
  public:
  
  typedef LSDBaseRaster<> LSDRaster;
  ...
};
{% endhighlight %}

And this would be ok, but you'd still have to change a lot of people's code...

(TBC...)



