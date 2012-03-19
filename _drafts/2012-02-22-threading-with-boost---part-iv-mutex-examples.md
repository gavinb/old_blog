---
layout: post
title: "Threading with Boost - Part IV: Mutex Examples"
category: Boost
tags: [Boost, C++, Threading]
---
{% include JB/setup %}

This article continues the series on threading with Boost, by looking in depth at several different example programs which illustrate different aspects of using mutexes.  We look at the code, and discuss how it is implemented.

Mutex Examples
==============

The full source for the sample applications is provided below.  It is designed to be fully portable, and has been tested on Mac OS X and Linux.  It should work fine on Windows (please send a patch if it doesn't!).  Download the source to follow along with the discussion below.

 - `boost_mutex.zip` (2kB zip file)

You can also clone my Mercurial [Boost Examples repository](http://hg.antonym.org/src/boost_mutex) for the latest:

    hg clone http://hg.antonym.org/src/boost_mutex

(See the [Mercurial DVCS homepage](http://www.selenic.com/mercurial/) for full details on this wonderful source control system.

Running `bjam` will build all the examples for you, which you can run from the appropriate subdirectory (depending on your toolchain and architecture).

Trylock with Queueing Threads
=============================

Illustrates two cooperating threads: a producer placing work items in a queue, and a consumer which removes work items from the queue.  The shared queue is protected by a mutex.  Each thread locks the mutex when pushing or pulling work items, to protect against concurrent access.  Work items are arbitrarily represented by random numbers.  Each thread holds the lock for a random time delay to simulate processing.

Since the producer may be holding the lock when the consumer wants to access the queue (or vice versa), the consumer performs a `try_lock` on the mutex.  So instead of blocking until the mutex is free (as a regular call to `lock()` would), the call fails immediately if the mutex is already locked, and the thread resumes.

Each thread prints out the stage it is executing, to enable analysis of the interaction.

(For a much better implementation of the Producer-Consumer pattern, see the article on Boost Condition variables later in this series.)

Mutex Locking with Timeout
==========================

Shows how to use `try_lock` on a mutex.  A 'holding' thread idles for a short time, then grabs the mutex and holds it for another short time before unlocking it.  The second thread is the 'trying' thread, in that it idles and then *tries* to acquire a lock on the mutex.  But it specifies a timeout by calling the `timed_lock()` method, and can fail if the holding thread hasn't released the mutex in time.  If it manages to grab the mutex, it holds it for a short time also.  The idle and holding times are different, to ensure the threads don't run in lockstep.  Note that `unlock()` is only called if the lock succeeds!

Recursive Lock
==============

Illustrates recursively locking a mutex.  A singleton class, ResourceManager, can register and unregister clients.  Since this may be called from the context of any worker thread, it uses a mutex when accessing the dictionary of client information.  Since retain/release management also requires the mutex be held during updates, this serves as an illustration of recursive locking.  These methods can be called individually, or from within the register/unregister methods, which also hold the mutex.  For example:

    registerClient()                  lockCount = 0
        mMutex.lock()                 lockCount = 1
        retainClient()                lockCount = 1
            mMutex.lock()             lockCount = 2
            mMutex.unlock()           lockCount = 1
        mMutex.unlock()               lockCount = 0
