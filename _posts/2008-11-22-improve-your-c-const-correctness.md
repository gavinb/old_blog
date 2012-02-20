---
layout: post
title: "Improve your C++: Const Correctness"
category: 
tags: []
---
{% include JB/setup %}

Writing "const-correct" code will improve the quality and maintainability of your code.  It is especially important and useful when writing Object-Oriented code, as objects are often passed around as constant references.  Properly declaring non-mutating methods as `const` allows you to safely call any `const` method on such a reference.  It is part of good type-safe practice and good code hygiene. So how do we do it?

Definitions
-----------

First, some definitions.

 - *mutating*: any function or method that alters the state of an object (ie. makes an assignment to any of its data members) is a 'mutating' operation. For example, calling a "setter" method is a mutating operation.
 - *non-mutating*: any operation that does not modify the object's data members.  For example, calling a "getter" method.
 - `const` *parameter*: if you have a constant reference or pointer to an object, you may not modify it or call any mutating methods on it.
 - `const` *method*: a method declared constant, promises not to modify the object when called.
 - `const` *correct*: writing code that consistently declares methods and parameters `const`, wherever possible

There are two simple rules of thumb to follow:

 1. If a method does not modify the object's state, the method should be declared `const`
 2. If a method or function does not modify a pointer or reference parameter, the parameter should be declared `const`

Together, these two rules ensures that objects are `const` as often as possible.  Why is this a good thing?  It gives the compiler more type information, which can catch potential bugs, and even give opportunities for optimization.

How do you declare a method to be `const`?  Just add the `const` keyword after the normal method declaration, thus:

	class A
	{
	public:
	    // ...
	    unsigned calculateSum() const;
	    // ...
	};

You declare a function or method parameter `const` in the usual way.  Note that in C++, references are usually preferred over pointers wherever possible (unless the parameter could be `NULL`).

	unsigned processTree(const node& root)
	{
	    // ...
	}

Less Good Example
-----------------

Let's use a simple Customer class as an example.  This is valid, but not really const-correct code:

	class Customer
	{
	    public:
	        Customer(std::string name) :
	            _name(name)
	        {
	        }
	        std::string getName()
	        {
	            return _name;
	        }
	       void setName(std::string aName)
	        {
	            _name = aName;
	        }
	        // ...
	    private:
	        std::string _name;
	        // ...
	};
	&nbsp;
	// ...and elsewhere...
	unsigned Invoice::addCustomer(Customer&amp; cust)
	{
	    // ...call any methods
	}
	//
	unsigned Invoice::getInvoiceCountFor(Customer&amp; cust)
	{
	    // ...call any methods
	}

There are several opportunities here to add `const` qualifiers to the code.  The accessor method for `name` does not modify the object state, so it can be made into a `const` method by adding the keyword at the end of the method signature.  The initial value for `name` in the constructor, along with the new name in the mutator can both take `const` references to strings (instead of passing by-value, which is less efficient).

Improved Example
----------------

So the above code becomes:

	class Customer
	{
	    public:
	        Customer(const std::string&amp; name) :
	            _name(name)
	        {
	        }
	        std::string getName() const
	        {
	            return _name;
	        }
	       void setName(const std::string&amp; aName)
	        {
	            _name = aName;
	        }
	        // ...
	    private:
	        std::string _name;
	        // ...
	};
	&nbsp;
	// ...and elsewhere...
	unsigned Invoice::addCustomer(const Customer&amp; cust)
	{
	    // ...call any const methods
	}
	//
	unsigned Invoice::getInvoiceCountFor(const Customer&amp; cust)
	{
	    // ...call any const methods
	}

Once you have a `const` reference to an object (such as the `cust` parameter above in `getInvoiceCountFor()`, you can only call `const` methods on that object.  For example, the compiler will flag it as an error if you try to call `setName()`, as it is a mutating method.

Advantages
==========

So why go to the extra effort?  Doesn't it mean you can do *less* with the objects if you go around declaring things `const`?  Potentially, yes.  But there are a number of benefits:

 - giving the compiler more type information can help catch potential bugs
 - establishes a "contract", so you know that certain methods or functions will not modify your objects
 - acts as a form of documentation, by declaring the behaviour of methods and treatment of parameters
 - provides opportunities for optimization by the compiler

Summary
-------

By following the two simple rules above, you can write tighter, safer code that is easier to manage.
