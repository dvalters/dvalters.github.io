---
title: Cython: Part 1
layout: post
tags: cython python optimisaton
categories: code optimisation
---

Cython is an extension to the Python language that gives it C-like extension and features, offering C-like speed by combining compiled C extensions with a Python interface. Cython is _not_ the same as _CPython_, which is an implementation of Python written in the C language (i.e. a Python interpreter written in C). Cython is both a Python module and an extended syntax that can be combined with regular Python to improve performance in compute-intensive sections of code. 

# Background

# A basic example

This example uses a specific piece of scientific visualisation code, used to render nice looking topographic maps from digital elevation model  data (terrain data obtain from either satellite imagery or laser scanning). The algorithm is referred to here as the _hillshade_ algorithm, as it is often used to create a pseudo-3D looking image of topography from elevation data, by applying grayscale shading based on the position of an imagined light source shining over the terrain. (A bit like the beautiful old-school Ordnance Survey maps that used to be shaded in a similar way to give the impression of topography rising out of the paper's surface.)

# Hillshade in pure Python

Firstly, we are going to look at the code in 'pure' Python, i.e. a basic implementation of the hillshade algorithm using standard Python. (no Cython yet.)

This is a dummy example, and we are going to use bog-standard Python to create a matrix/2-dimensional array to contain our terrain data and then produce the hillshade array as well. NOTE: This is a silly way to do things, since we could use numpy instead -- and we will quickly move on to a Numpy example afterwards -- but here is the pure Python version for illustrative purposes only (mainly to show how slow Python can be).

{% highlight python %}

{% endhighlight %}

## Sensible Numpy version

OK, using _just_ core Python is a bit silly, since Numpy should be the go-to choice when dealing with multidimensional arrays or matrix-like structures and doing numerical calculations with them. So here is a nicer Numpy version of the above.

## Cython version attempt 1

Now let's try a Cython version

## Cython version attempt 2

## Improving the Cython version some more

## Compile on the fly vs compiled library extension

## Further thoughts


