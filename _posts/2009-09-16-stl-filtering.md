---
layout: post
title: "STL: Filtering"
category: 
tags: []
---
{% include JB/setup %}

The STL makes it easy to create lists, iterate over lists, and apply a function to each member of a list.  So how do you filter a vector according to some criteria?  It's not hard, but the obvious solution isn't *quite* enough.  Here's how.

Say you have a vector of values, and you want to apply a filter to the list.  That is,  remove elements that satisfy some criteria.  It's quite a common requirement, so you'd think there would be an obvious and simple way to do it.  If you go searching for the word "filter" in the STL documentation, you may not find much relevant in the `std::vector` method reference.  If you browse around in STL algorithms, eventually you'll come across a method called `remove_if`.  It takes two iterators to specify the range (first and last) of the operation, and a predicate function.  Sounds perfect!

Let's use it to filter a list of numbers, removing even numbers.  Given the following input:

    1 2 3 4 5 6 7 8 9

we expect to see the result:

    1 3 5 7 9

A predicate function is essentially a test for a property, and returns whether or not the parameter has this property.  So our predicate function will take an integer value (matching the element type of our vector) and return a boolean, denoting whether or not to remove the given element.  So we'd have:

	bool is_even(int N)
	{
	    return N % 2 == 0;
	}

Easy!  Now, to store our numbers, we would start with a simple vector of integers, thus:

	typedef std::vector<int> vector_t;
	&nbsp;
	vector_t    numbers;

We read in the numbers in a loop and save them in the array:

    while (true)
    {
        int n;
        &nbsp;
        std::cin >> n;
        &nbsp;
        if (!std::cin.good())
            break;
        &nbsp;
        numbers.push_back(n);
    }

This will take care of reading in the data, as it will stop as soon as it reads something that isn't an integer, or reaches the end of file.  Now we just apply the filter, and then print the result:

    remove_if(numbers.begin(), numbers.end(), is_even);
    &nbsp;
    vector_t::iterator it;
    for (it = numbers.begin(); it != numbers.end(); ++it)
    {
        std::cout << *it << std::endl;
    }

There, we're done!  Now, we just run the completed program, and give it some test data:

    $ ./filter 
    1 2 3 4 5 6 7 8 9 -

But wait, the output we see is wrong!

    1
    3
    5
    7
    9
    6
    7
    8
    9

Oh, no!  There must be a bug in the libraries!  Curses, foiled again!

Actually, no - the sky isn't falling, and there is no bug.  Now is when we probably should be doing a bit of RTFMing.  Let's check the docs and see what's going on.  According to the [g++ library documentation](http://gcc.gnu.org/onlinedocs/libstdc++/latest-doxygen/):

> "Elements between the end of the resulting sequence and 'last' are still present, but their value is unspecified."

Er, ok... that's handy - not.  Looking back at the output, we do notice that the first 5 values are correct, but there's some junk after them.  And sure enough, that's the documented behaviour - we are just seeing the "unspecified" values left behind.

It is worth pointing out right about now that the fine folk who designed the STL were fairly obsessed with efficiency, and the STL is designed to be low-level, powerful and fast, sometimes at the expense of user friendliness.  It is more efficient to implement filtering as a generic algorithm that can be applied to any container.  And sure enough, that is the intent, as we see in the docs:

> "@return   An iterator designating the end of the resulting sequence."

Ah, we were actually supposed to do something with the return value!  The implementation of `remove_if` moves (or actually copies) the values to be kept to the head of the list, and returns an iterator pointing to the start of the "junk" (just after the filtered list), in preparation for this tail to be snipped off.  So, let's use the `erase` method to do just that:

    numbers.erase(remove_if(numbers.begin(), numbers.end(), is_even), numbers.end());

Now, running the test application again, we get the output:

    1
    3
    5
    7
    9

Huzzah!  It is really a matter of getting a feel for the design patterns behind the STL, and the very low-level API that it provides.

This process can be visualised thus:

<img alt="STL Filtering Example" src="http://antonym.org/images/STLFilteringExample.png" width="532" height="670" />

It is also worth mentioning at this point that if you are doing a lot of adding and removing of  elements, a `std::list` is probably a better choice, as inserting and deleting elements in vectors is very expensive.

Now, you're probably wondering why `remove_if` is a function in the STL `algorithms` collection, rather than a method in `std::vector` itself.  Well, the folks who designed the STL are very, very clever.  The obvious thing would be to put a `remove_if` method in every single container class.  But the implementation would be *almost* the same, right?  Well, they reasoned, what if all container classes provided a common, minimal interface, we could implement `remove_if` just once, and it could apply to almost any container!  It's even more generic that way.  So there are a whole series of algorithms that apply to a wide variety of different containers, and use informal interfaces to decouple things.

The (platform-neutral) sample code is available below...

 * Sample C++ code: <a href="http://antonym.org/files/stl_filter.zip">stl_filter.zip</a> (4.5kB zip)
