---
layout: post
title: "Threading with Boost - Part III: Mutexes"
category: Boost
tags: [Boost, C++]
---
{% include JB/setup %}

In [Part I](http://antonym.org/) of this series on [Boost](http://boost.org/) threading, we looked at the basics of how to create and run threads using the Boost libraries.  Then we reviewed the main issues encountered with multithreading code in [Part II: Threading Challenges](http://antonym.org/). One of the biggest challenges is safely managing concurrent access to a resource.  A Mutex provides a way to serialise access to a shared resource, to ensure your data is consistent.  In this article, we look at how to create and use Boost mutexes.

Mutual exclusion - Mutex
========================

One of the main solutions to race conditions and other threading problems is the *mutex*, short for "mutual exclusion".  Think of it as a special kind of lock, which is used to protect shared resources.  A mutex is special because it is guaranteed to be held by *at most* one thread at a time.  While one thread holds a lock on a mutex, no other thread is able to.  Thus mutexes can be used as a protection mechanism for resources and data, by locking before using shared resources and unlocking when finished. A mutex can be used to avoid race conditions, and implement atomic operations.

If a mutex is not locked by a thread, then any thread can lock it.  Once it is locked, if any other thread attempts to lock it, it will typically wait until the mutex is unlocked by the owning thread.  It is also possible to return if the lock fails, or wait for a certain period of time before giving up.  These variations are discussed below.

Mutex Scope
-----------

A mutex is typically declared alongside the resource it protects.  The mutex should be in the same scope as the resource, or an enclosing scope.  For example, a data member in a C++ object that can potentially be accessed by more than one thread at a time should have a mutex declared alongside.  A class instance (static data member) should have a similarly class-scoped mutex.  The shared resource should never have a greater scope than the mutex, otherwise it may not always be protected.  Let us not speak of the blight known as global variables, surely a symptom of deficient design.

In simple cases, a single mutex per object may be sufficient, but this is totally dependent on the algorithms used.  If there are multiple resources being accessed independently, having a single mutex may lead to too much contention.  In this case, a separate mutex for each resource may be more efficient.  Only analysis and performance tests will reveal the more efficient choice.

Any time the resource is accessed (whether for reading or writing), the mutex should *always* be locked before use, and unlocked after the operation.

Types of Mutexes
----------------

There are four flavours of mutexes available in Boost, depending on your requirements.  Each are described in detail below, and feature a sample program illustrating its use.  The full source code is available for browsing, downloading or even cloning from my Boost Theading Examples Mercurial repository on [BitBucket](http://www.bitbucket.org/gavinb/boost_examples/).

### Regular Mutex

The simplest form of mutex is a regular `boost::mutex`.  You lock and unlock it, and only one thread can lock the mutex at a time.  Any thread that calls `lock()` on a mutex held by another thread will block indefinitely (an important factor when considering synchronisation).  This is worth repeating: calling `lock()` can block *indefinitely*.  This may be the desired behaviour, as your thread may need to wait an undetermined time for something else to happen (but this then raises issues of interruption).  It should be obvious then that this must be carefully managed to avoid program hangs, a common symptom of concurrency failure.

Regular mutexes also have a `try_lock()` method, which will return immediately with a failure status if the mutex cannot locked.  Since most threads run in a loop, it would be terribly inefficient to try to lock a mutex and then continue around the loop without performing anything else, especially when it is unlikely that the mutex will become immediately available.  In these cases, it *may* be appropriate to do a short sleep (but see below for a better solution).  Even then, this may be a sign that your algorithm needs improvement.  Unfortunately, there are no one-size-fits-all solutions.

### Timed Mutexes

The `boost::timed_mutex` class is a subtype of `boost::mutex`, which adds the ability to specify a timeout.  For example, you may wish to try to lock the mutex but give up after a certain time if you cannot obtain a lock.  This takes either an absolute time, or a relative time.  If the mutex cannot be obtained within the time specified, the call will return false and the mutex is not held.  If the mutex is locked within the timeout period, it returns true.

### Recursive Mutexes

Normally a mutex is locked only once, then unlocked.  Depending on the structure of your application, there may be times when it would be useful to be able to lock a mutex multiple times *on the one thread* (in very special circumstances, such as nested method calls).  For example, you have two (or more) methods which may be called independently, and another method which itself calls one of the other two.  If they all need the mutex held to function safely, this can cause complications when determining when the mutex needs to be locked and released.  However, by using a recursive mutex, it can be locked as many times as necessary, provided it is unlocked the same number of times.  Thus all these methods can be called individually, as they all lock the resources, and in a nested fashion, as the mutex can be locked multiple times.  Provided the mutex is unlocked the same number of times (which is a matter of care and thought), the mutex will be correctly released by the end of the nested operation.

### Shared Mutexes

Some concurrency scenarios involve having one writer and many readers.  For example, one thread may be downloading data from the network, while another thread is displaying the data on the screen, and a third thread is saving the data to a database.  So the downloading thread will be locking for writing, and the other two only for reading.  There is therefore no reason why the display thread and the database thread (which both only read the shared resource) need to exclude the other; concurrent reading is perfectly safe.  It is only if the network updating thread needs to write to the shared data that the other two need to be locked out.  This process is shown below:

<!-- @todo Insert diagram here -->

Any time the reading threads need to read the resource, they obtain a read-lock.  This allows other read-locks to successfully access the resource, but will prevent a write-lock (since you don't want updates to occur while someone else is reading - the data must be consistent).  When an update comes in on the networking thread, it must wait until the readers have finished before it can obtain a write-lock.  This will prevent any readers from accessing the resource for the duration of the update.  In this way, concurrent access is increased, contention is reduced, and the resource is always consistent.

<!-- @todo Insert diagram here -->

One of the example programs shows this in action.

### Summary

The table below summarises the difference mutex types, and their main distinguishing features:

<table>
	<tr>
		<td><tt>boost::mutex</tt></td>
		<td>Normal mutex, most commonly used, blocking lock</td>
		<td>has <tt>lock()</tt> and <tt>try_lock()</tt></td>
	</tr>
	<tr>
		<td><tt>boost::timed_mutex</tt></td>
		<td>Adds ability to timeout waiting for a lock</td>
		<td>adds <tt>timed_lock()</tt> to wait for lock with a timeout</td>
	</tr>
	<tr>
		<td><tt>boost::recursive_mutex</tt></td>
		<td>`lock()` can be called multiple times on the *same* thread</td>
	</tr>
	<tr>
		<td><tt>boost::shared_mutex</tt></td><td>lock can be upgraded to allow multiple readers or a single writer</td><td>can upgrade locks (R->W)</td>
	</tr>
</table>

Lock duration
-------------

It is important that the mutex only be locked for the shortest possible time to ensure data integrity is maintained.  If a lock is held for too long, it may cause other threads to wait excessively, thus stalling processing and negating the benefits of concurrent processing.  Getting the granularity of threading right takes experience and judgement, so learning by studying existing multithreaded code is an excellent way to pick up design patterns.

The next article shows some timing diagrams which illustrate lock contention in simple apps, so you can see just how much time a thread is waiting.

Lock/Unlock Pairing
-------------------

If a mutex is not released due to a logical error (such as an uncaught exception), this may cause the program to lock up or behave strangely, and is probably not recoverable.  Thus it is vitally important that all lock/unlock operations appear in pairs.

To protect against this class of problem, the `lock_guard` object was introduced.  It locks the mutex for you in its constructor, and unlocks it in the destructor.  (This safety-conscious approach is known as [RAII](http://en.wikipedia.org/wiki/RAII), or Resource Acquisition Is Initialisation, whereby the lifecycle of a resource is tied to the lifecycle of an owning object).

This simple example code shows potentially unsafe method implementation.

	void writeTotalsToDatabase() throw (myapp::SQLException)
	{
	    // does SQL stuff, and throws on error
	}
	
	unsigned applyTotals(unsigned count)
	{
	    mTotalsMutex.lock();
	    
	    mTotals.globalCount += count;
	    
	    writeTotalsToDatabase();
	    
	    mTotalsMutex.unlock();
	}

This code looks at first glance to be safe.  It simply makes the updating and saving of the totals atomic, right?  Well, yes - except if there's an exception thrown by `writeTotalsToDatabase()`.  If that happens, the normal flow of execution goes out the window, and the program unwinds the stack in search of a suitable `catch` statement.  And the lock that we have acquired will never be released!  This is very dangerous, and can happen in seemingly "safe" code.  While you could certainly put a `try/catch` around the database method, a better way is to use a lock guard.

	unsigned applyTotals(unsigned count)
	{
	    boost::lock_guard    totalsLock(mTotalsMutex);
	    
	    mTotals.globalCount += count;
	    
	    writeTotalsToDatabase();
	}

The `totalsLock` will acquire (or wait to acquire) the mutex at the start, and when it goes out of scope at the end of the method, it will unlock the mutex.  If an exception is thrown, the lock's destructor will be invoked as part of unwinding the stack, and there the mutex will be safely released.  This ensures the operation is atomic, and can safely handle error conditions.

Avoiding Deadlocks
------------------

So far, we've looked at using one mutex at a time.  But a non-trivial application may involve many threads, and even more mutexes.  And what happens when there is contention for mutexes between threads?  There is a fatal error known as a deadlock, and in its simplest form, two threads are holding a resource each, which the other also wants.  Since neither will release one until it gets the other, the threads get stuck waiting forever (or at least until the user gets sick of waiting and kills the process).

Shared Object with One Mutex
----------------------------

Imagine you have a shared object, such as a singleton, that may be accessed by multiple threads in the system.  A simple solution to protect its mutating (ie. non-`const`) methods is to give the object a mutex, and use a lock guard in every method.  For example:

	class FooManager
	{
	    public:
	        FooManager();
	        void registerFoo(const Baz&);
	};

A note on performance
---------------------

Under Linux, the Boost threads implementation uses pthreads and pthread mutexes.  These may use a spinlock at the kernel level, for optimal performance.

Under Windows, a real "Mutex" object at the operating system level is actually a fairly expensive, heavyweight and slow construct (relatively speaking), mainly intended for intra-process synchronisation.  If you only need serialisation within your process, a "critical section" is a far better choice, as it is significantly faster.  Fortunately, Boost uses a CritSec for process-level mutexes.  If you need inter-process mutexes, look at the [Boost IPC library](http://boost.org/).

Other types of Mutexes
----------------------

There are a few different flavours of Mutexes, which can be used in more sophisticated scenarios. These are beyond the scope of this article (which is already long enough!).

Up Next
-------

The next article will focus on a series of examples showing the use of different mutex types. The source has been published on BitBucket in my [Boost Sample Code](http://bitbucket.org/gavinb/boost_samples/) repository.  Feel free to use, fork, and share!
