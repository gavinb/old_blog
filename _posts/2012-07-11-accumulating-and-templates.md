---
layout: post
title: "Accumulating and templates"
category: 
tags: [C++, STL]
---
{% include JB/setup %}

Let's say you have a vector of doubles, such as:

{% highlight c++ %}
std::vector<double> numbers;
{% endhighlight %}

and you want to add them up, there's a very useful function in the STL's `numeric` header called `accumulate` which does exactly that. However, there's a catch!  It is important to understand that the starting value of `std::accumulate` will determine the type of the result.  So using:

{% highlight c++ %}
std::accumulate(numbers.begin(), numbers.end(), 0);
{% endhighlight %}

will round off the fractions of all your doubles!  Instead, you should use:

{% highlight c++ %}
std::accumulate(numbers.begin(), numbers.end(), 0.0);
{% endhighlight %}

Why is it so?  Let's look at the definition of `std::accumulate` :

{% highlight c++ %}
template<typename _InputIterator, typename _Tp>
_Tp
accumulate(_InputIterator __first, _InputIterator __last, _Tp __init)
{
    for (; __first != __last; ++__first)
        __init = __init + *__first;
    return __init;
}
{% endhighlight %}

So the template parameter `_Tp` is the type of the initial value (`__init`), and determines the return type.  In the body of the function, we see that all it does is loop using the iterators and add the current value to the initial value.  If we specify 0 as the initial value, the sixth line would have the following types:

{% highlight c++ %}
int = int + double;
{% endhighlight %}

which would cause the roudning via the implicit conversion. (This seems to be covered by section 4.9 of the C++ standard, but it doesn't say the compiler needs to warn.)

Here's a trivial example that shows the difference:

{% highlight c++ %}
#include <iostream>
#include <numeric>
#include <vector>

int main(int argc, char* argv[])
{
    std::vector<double> numbers;

    numbers.push_back(1.1);
    numbers.push_back(2.1);
    numbers.push_back(3.1);
    numbers.push_back(4.1);
    numbers.push_back(5.1);

    std::cout << "Sum (int):    "
              << std::accumulate(numbers.begin(), numbers.end(), 0)
              << std::endl;

    std::cout << "Sum (double): "
              << std::accumulate(numbers.begin(), numbers.end(), 0.0)
              << std::endl;

    return 0;
}
{% endhighlight %}

The output of this little test is:

{% highlight c++ %}
Sum (int):    15
Sum (double): 15.5
{% endhighlight %}

When compiled with g++ 4.2 and clang 3.0, neither offer a warning, even with `-Wall Wconversion -pedantic` enabled.

Some other compilers may warn about this loss of precision, so YMMV.  It's a very subtle bug, as you could be getting something close to the expected value and not know why.  Another good argument for strict compilers!
