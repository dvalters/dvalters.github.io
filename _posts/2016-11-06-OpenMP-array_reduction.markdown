---
layout: post
title: Array sum reductions in OpenMP 4.5
tags: arrays openmp openmp45 
---
Some interesting new OpenMP functions in [OpenMP 4.5](http://developers.redhat.com/blog/2016/03/22/what-is-new-in-openmp-4-5-3/), including the potentially useful reduction on arrays for C and C++ now (this was previously supported for Fortran only).

You can now perform summations of arrays using the reduction clause with OpenMP 4.5.

Reductions can be done on variables using syntax such as:

{% highlight cpp %}

double total_sum = 0.0;
int imax = 100;

#pragma omp parallel for reduction(+:total_sum)
for (int i=0; i<imax; i++)
{
  total_sum += 42;
}
{% endhighlight %}

So each thread gets its own copy of total_sum, and at the end of the parallel for region, all the local copies of total_sum are summed up to get the grand total. 

Suppose you have an array where you want to summation of each array element (not the _entire_ array). Previously, you would have to implement this manually. 

{% highlight cpp %}
#include <iostream>

int main()
{
      
  int myArray[6] = {};

#pragma omp parallel
{
  int private_myArray[6] = {};
  #pragma omp for 
  for (int i=0; i<50; ++i)
  {
    double a = 2.0; // Or something non-trivial justifying the parallelism...
    for (int n = 0; n<6; ++n)
    {
      private_myArray[n] += a;
    }
  }
  
  #pragma omp critical
  for (int n = 0; n<6; ++n)
  {
    myArray[n] += private_myArray[n];
  }
}

  // Print the array elements to see them summed   
  for (int n = 0; n<6; ++n)
  {
    std::cout << myArray[n] << " " << std::endl;
  } 
}
{% endhighlight %}


Whereas now in OpenMP 4.5 you can do

{% highlight cpp %}
int main()
{

  int myArray[6] = {};

  #pragma omp parallel for reduction(+:myArray[:6])
  for (int i=0; i<50; ++i)
  {
    double a = 2.0; // Or something non-trivial justifying the parallelism...
    for (int n = 0; n<6; ++n)
    {
      myArray[n] += a;
    }
  }
  // Print the array elements to see them summed   
  for (int n = 0; n<6; ++n)
  {
    std::cout << myArray[n] << " " << std::endl;
  } 
}
{% endhighlight %}


Outputs:
{% highlight console %}
    100
    100
    100
    100
    100
    100
{% endhighlight %}

I compiled this with GCC 6.2. You can see which common compiler versions support the OpenMP 4.5 features here: http://www.openmp.org/resources/openmp-compilers/



