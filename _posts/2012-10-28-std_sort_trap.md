---
layout: post
title: "A trap with std::sort and custom comparison operators"
category: Coding
tags: [c++,stl]
---
{% include JB/setup %}

The Standard C++ library includes many useful features for processing containers, including `std::sort` for sorting vectors and lists.

To use `std::sort` in its most basic form, you simply pass it a vector (or rather iterators pointing to the start and end of the vector) of comparable items, such as `int`s:

{% highlight c++ %}
// ...
unsigned            N = 1000;
std::vector<int>    numbers;

for (unsigned i = 0; i < N; i++)
    numbers.push_back(random() % 100);

std::sort(numbers.begin(), numbers.end());
// ...
{% endhighlight %}

This will generate a sequence of random numbers, then sort the numbers in ascending order.  The implicit comparison function used in this case is `operator<()` (or more correctly, `std::less<T>`).

Now, what happens if you have a custom data type and you want to sort those?  Let's take a simple example of a sequence of points with an associated weight `(x, y, w)`:

{% highlight c++ %}
struct WeightedPoint
{
    int         x;
    int         y;
    float       weight;
};
{% endhighlight %}

The custom sort function *must* be equivalent to `operator<()` (now this is very important, as we shall see later!).  The signature should be of the form:

{% highlight c++ %}
bool my_compare(const T& a, const T& b);
{% endhighlight %}

So in our case, the comparison function would like something like:

{% highlight c++ %}
bool compare_weightedpoints_by_weight(const WeightedPoint& a, const const WeightedPoint& b)
{
    return a.weight < b.weight;
}
{% endhighlight %}

Now, if we run this code, everything works perfectly.  Great, so what's the problem?

Well let's says we want to revers the sort order - the obvious thing would be just to negate the return, like this:

{% highlight c++ %}
bool compare_weightedpoints_by_weight(const WeightedPoint& a, const WeightedPoint& b)
{
    // Actually WRONG!
    return ! (a.weight < b.weight);
}
{% endhighlight %}

This is a nice, simple function that reverses the sort order, right?  Well, as the comment in the source gives away the answer, it is certainly *wrong* - but why?

What most C++ library reference documentation don't tell you is that the semantics of the comparison operator are required to provide *strict weak ordering*, equivalent to `operator<()`.

Why does this matter?  Why should you have to know about set theory, antisymmetric this, and weak strict that, etc.   Because when it comes to comparing items that are *equal*, it is critical that these properties hold.  If not, the sorting code will potentially *corrupt your memory*.

Surely not!  How could getting the sorting order wrong corrupt memory? Surely it would just get things in the wrong order?  Well, the implementation of `std::sort` is heavily optimized, and relies on the comparison function providing precisely the correct semantics.  And if your comparison function gets it wrong, under certain conditions (when two elements have the same value) the sorting function can zoom past the end of your allocated memory and start writing over other parts of the heap.

There are numerous accounts of this happening in various contexts:

 * foo
 * baz
 * bar

The article on Wikipedia on [strict weak ordering](http://en.wikipedia.org/wiki/Strict_weak_ordering) goes into some great detail, but the takeaway message is this:

	4 < 9	T 	!(4 < 9)	F
	4 < 3	F 	!(4 < 3)	T
	4 < 4	F 	!(4 < 4)	T

	9 < 4	F 	!(9 < 4)	T
	3 < 4	T 	!(3 < 4)	F
	4 < 4	F 	!(4 < 4)	T

So - given a regular comparison function:

{% highlight c++ %}
bool my_compare(int a, int b)
{
    return a < b;
}
{% endhighlight %}

do *not* simply invert the comparison result to get the inverse sort order:

{% highlight c++ %}
bool my_compare_rev(int a, int b)
{
    // WRONG!!!
    return !(a < b);
}
{% endhighlight %}

you need to *reverse* the direction of comparison to preserve the ordering semantics:

{% highlight c++ %}
bool my_compare_rev(int a, int b)
{
    // Right
    return b < a;
}
{% endhighlight %}

Because this is such a subtle problem with quite disastrous results, I intend to improve the situation in two ways:

1. Submit patches to the relevant documentation for `g++` and the [C++ Reference](http://www.cppreference.com/) to make these requirements more clear.

2. See if there is some way to patch the implementation in `g++` to provide a static assertion in debug mode (ie when `_DEBUG` is defined) to help protect against memory corruption.  Since it has the .begin() and .end() iterators, it should be possible to guard against running off the end of the buffer, at least when working with vectors (which have contiguous storage).
