---
layout: post
title: "STL Warts: When removing isn't"
category: Coding
tags: [C++,coding,STL]
---
{% include JB/setup %}

Pop quiz: quick - what does the `remove` function provided in the C++ STL `algorithm` package do?

    std::remove(list<T>::iterator begin, list<T>::iterator end, T& t);

Simple question, surely... Your answer?

If you said "it removes the value t from the list between the two iterators" then -- bzzzzt, thank you for playing.  It doesn't actually remove anything.  Quoting from the description:

	Reorders the range [first,last) to prepare for erasing all elements of equal value.

So the way to *actually* erase something from a `std::vector`:

	v.erase(remove(v.begin(), v.end(), item));

All `remove` actually does is shuffle the items to be deleted to the end, and return an iterator to them.  Then the <tt>erase</tt> method from the vector snips them off the end.

The above `erase/remove` line is a standard idiom for actually removing an item by value from a container.

This is one of several unfortunately named methods in the STL.  There are other interesting design warts which I'll discuss in future articles.
