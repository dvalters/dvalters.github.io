---
layout: post
title: Array sorting template C++
tags: c++, arrays
---
In some languages, there is a useful method that will sort the contents of an array based on the order of another:

For example, C# has this:

{% highlight c# %}
Array.sort(array, array)
{% endhighlight %}

This doesn't *quite* translate easily into C++. But don't worry, you can use this handy template:

{% highlight c++ %}
template <class T1, class T2, class Pred = std::less <class T> >
struct sort_pair_second
{
  bool operator()(const std::pair<T1,T2>&left, constd std::pair<T1,T2>&right)
  {
    Pred p;
    return p(left.second, right.second);
  }
};

// Usage:

std::sort(v.begin(),v.end(), sort_pair_second <int,int>() );
{% endhighlight %}
