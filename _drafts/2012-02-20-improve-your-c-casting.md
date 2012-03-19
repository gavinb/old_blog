---
layout: post
title: "Improve your C++: Casting"
category: Coding
tags: [C++]
---
{% include JB/setup %}

Along with a host of other improvements - and complexities! -  over regular C, the C++ language introduces several features in the type system worth reviewing.  By taking advantage of these features while working with types, you can improve your C++ code and hopefully make it more readable and robust.

Sample Classes
==============

For the following example code, we will assume the following class declarations:

{% highlight c++ %}
class Base
{
	public:
		virtual void foo();
		virtual void baz();
};

class Derived : public Base
{
	public:
		virtual void foo();
};

class Related
{
	public:
		void 		bar();

	private:
		Derived*	d;
};
{% endhighlight %}

Static and Dynamic Pointer Types
================================

The *static* type of a variable is determined at compile time, and is simply the type according to the variable declaration.  For example:

{% highlight c++ %}
Base*       basePtr;
Derived*    derivedPtr;
Related*    relatedPtr;
{% endhighlight %}

The *dynamic* type of a variable is determined at run time, and is the type of the instance to which the pointer refers.  This is often - but by no means always - the same, as in these examples:

{% highlight c++ %}
Base*       basePtr = new Base;
Derived*    derivedPtr = new Derived;
Related*    relatedPtr = new Related;
{% endhighlight %}

Clearly these all have the same static and dynamic type.  So when can they be different?  Well, a variable can be assigned (almost!) anything, provided the types are compatible, or can be *coerced* (such as from `int` to `float`).

In the example classes above, any instance of `Derived` is also an instance of `Base`.  So we should be able to treat a `Derived` instance as if it were a `Base` instance - it is a compatible type.  And sure enough, that is exactly what happens when the static type is `Base*` and the dynamic type is `Derived*`:

{% highlight c++ %}
Base* basePtr = new Derived;
{% endhighlight %}

We can assign a derived pointer to a base pointer, but not the other way around.  It is invalid, because a `Derived` instance is a superset of a `Base` pointer.

{% highlight c++ %}
Derived* invalidPtr = new Base; // This will not work!
{% endhighlight %}

So what happens when we start calling some methods?  In the case of virtual methods, such as defined in the sample code, the correct method will be invoked at runtime according to the *dynamic* type, thus:

{% highlight c++ %}
Base* basePtr1 = new Base;
basePtr1->foo(); // calls Base::foo
Base* basePtr2 = new Derived;
basePtr2->foo(); // calls Derived::foo
{% endhighlight %}

Even though we are only holding a `Base` pointer, the runtime is clever enough to figure out (through the magic of virtual method tables or v-tables) the correct virtual method to call.  (If the methods were not declared `virtual`, it would use the static type.)  This of course is the wonderful world of *polymorphism*.

So, what does all this have to do with casting?

Static Cast
===========

A `static_cast` converts between pointers or references of related classes.  You can cast both up and down the inheritance hierarchy, provided you know it is safe to do so.  You can, as you would expect, cast a derived class to its base class.  However, you can also cast a base class to a derived type.  Because this is one of those "trust me, I know what I'm doing" things, the compiler will not complain, nor are there any runtime checks to see if you really did cast to the right type (see `dynamic_cast` below if you want this).

A common feature of frameworks, especially GUIs that employ callbacks, is to allow the caller to pass some form of user data to the callback, which is supplied when the event is triggered, along with the rest of the data.  So passing an object's `this` pointer as the "user data" for a callback lets you manage the event with an object (often with the aid of a glue method to reflect the call).  This is a common (but not necessarily the most elegant) technique in C++.  For example:

{% highlight c++ %}
void MyController::init()
{
  myButton->setClickedHandler(MyController::buttonClickedReflector, this);
  //...
}
static void MyController::buttonClickedReflector(unsigned buttonId, unsigned eventFlags, void* userData)
{
  MyController* obj = static_cast<MyController*>(userData);
  obj->buttonClicked(buttonId, eventFlags);
}
void MyController::buttonClicked(unsigned buttonId, unsigned eventFlags)
{
  // ... do something sensible
}
{% endhighlight %}

Because the framework knows nothing about our application types, it simply uses a `void*` to carry around a pointer to whatever data we like.  This is convenient, but requires a certain level of care to ensure you put in what you expect to get out!  (And also that the lifecycle of the objects ensures that the receiver is still around when the event is triggered - but that is for another article.)

A `static_cast` can also be used anywhere you would have used an explicit cast in C; for example, between ordinal types, like this:

{% highlight c++ %}
const float PHI = 1.718...;
uint16_t scaled = static_cast<unsigned int>(PHI * 65535);
{% endhighlight %}

Dynamic Cast
============

The `dynamic_cast` uses Run Time Type Information (RTTI) to check the validity of the cast at runtime.  This incurs an extra runtime cost for the checking, and also extra space in the binary for storing all the additional metadata, so it is not without cost.  It will return `NULL` if the cast is invalid.

A `dynamic_cast` will always successed when casting a pointer or reference of a class to one of its base classes:

{% highlight c++ %}
void process(Base* obj)
{
	Derived d = dynamic_cast<Derived*>(obj);
	if (d)
	{
		// treat obj as Derived
	}
	// whatever
}
{% endhighlight %}

However, if the types are determined at run-time to be incompatible, a `dynamic_cast` will return NULL.  As mentioned above, this is safer than a `static_cast` but incurs a performance penalty.

There are a few other subtleties...

Const Cast
==========

# Link: improve-your-c-const-correctness

An earlier article (TODO add LINK!) extolls the virtues of the appropriate use of `const`.  The `const_cast` operator breaks the rules, allowing you to add or remove "constness" as you wish.  This is most useful when you have a constant reference and need to pass it to a function or method which takes a non-constant pointer or reference, but you can't change the interface.  For example:

{%highlight c++%}
void process(Derived* d)
{
	d->foo();
}

int main(void)
{
	const Derived* cd = new Derived;

	process(const_cast<Derived*>(cd));

	return 0;
}
{%endhighlight%}

(It would be breaking the contract with the user if you actually modified a const reference, which is very poor form indeed.)

Casting away (or casting in) const-ness has its uses, but should be used sparingly.  If the code is your own, you are usually better off using `const` wherever appropriate and refactoring accordingly.  For those times when you cannot, judicious use of `const_cast` makes your intention explicit.

Reinterpret Cast
================

This is the ultimate "trust me" casting operation.  No checking whatsoever is performed, and the compiler will let you cast just about anything to anything (even classes to integers, provided they are big enough to hold a pointer).  All this really does is a straight copy of the binary value of the pointer.

It is exceedingly rare that you would need to use this feature - you would almost always use `static_cast` instead.  So before resorting to this, make sure it is *really* what you need.

{%highlight c++%}
void network_error_callback(const Error& e, void* userData)
{
	Manager* mgr = reinterpret_cast<Manager*>(userData);

	mgr->handleNetworkError(e.code, e.message);
}
{%endhighlight%}

Type Information
================

If you build your app with RTTI enabled, you can use the `typeid` operator to provide type information at runtime.  While you would ordinarily use polymorphism to adapt runtime behaviour, there are (very!) occasionally situations when this might come in handy.  This feature is much more likely to crop up in framework code than application code.

The `typeid` operator returns a string which can identify either the static or dynamic type.  If you pass a pointer or reference, it returns the static type.  If you pass a dereferenced pointer, it will return the dynamic type.

The string returned from `typeid` is implementation-specific, so you cannot rely on it being any particular value for a given class.  Defensive programming is in order...

Summary
=======

Modern C++ has many new features that make for more readable, solid code.  Some of the casting features above should be used with caution, but the static and dynamic casts are very useful, and can be used to replace virtually all existing C-style casts in your C++ code.

Advanced
========

Down-casting can be is dangerous.  Try to avoid wherever possible.  It may be a sign of poor design, and you need to rethink it.  Cross-Casting is probably evil.
