---
layout: post
title: "Threading with Boost - Part II: Threading Challenges"
category: boost
tags: [boost,c++,threading]
---
{% include JB/setup %}

In [Part I](http://antonym.org/) of this series on [Boost](http://boost.org/), we looked at the basics of how to create and run threads using the Boost libraries.  But once you have more than one thread running in a process, you have to deal with the problems and challenges that threads can introduce.  So, before delving into the mechanics of how to use mutexes and other threading constructs, we look at what can go wrong - and how to avoid it.

Shared State
============

There is a simple solution to most major threading problems: have no shared state.  By shared state, I mean any data or resource (such as file handle, socket, graphics context, queue, buffer, etc) that is used by more than one thread at the same time.  If two threads are truly independent, they can safely run concurrently without care or consideration.  We wouldn't need sophisticated mechanisms for synchronisation if there's nothing to sync over.  Unfortunately, this restriction is not at all practical or realistic.  And as soon as you introduce shared state, you have to worry about atomicity, consistency, race conditions, and all sorts of issues.

So one of the first design considerations for concurrent systems is to try to minimise the amount of shared state between threads.  The less shared state, the less complexity there is to manage, and the less overhead imposed. Locks that protect shared resources can harm performance: each lock is a region that enforces sequential access, and thus reduces the opportunities for concurrent scheduling.

Simply put, any resource shared between more than one thread needs to be protected by a mutex, such that access is serialised.

When considering scalability, the theoretical maximum speedup of N cores is always less than N times, due to scheduling and synchronisation overheads.  Thus the most efficient algorithm designs consider these issues up front.

Problem: Atomicity
==================

An operation is *atomic* if the operation completes without interruption.  It is never partially complete, which may leave the system or data in an inconsistent or invalid state.  Atomic operations come up all over the place, and even things that we might *think* are atomic (ie. a single line in your source) probably isn't.  In databases, we use SQL transactions to ensure operations are atomic, such that a related set of changes are never partially applied.

The classic example of an atomic operation in a database is a bank transfer.  You decide to pay your landlord by direct deposit, and transfer $1200 to their account.  The normal sequence of operations is:

1. Ensure you have sufficient funds
1. Ensure the receiving account number is valid
1. Withdraw the $1200 in funds from your account
1. Deposit the $1200 in the landlord's account

Now the first two steps are really preconditions, and while they are necessary for the entire operation's success, if the operation was cancelled after step 1 or step 2, there would be no harm done, and nothing changed.  But once the money comes out of your account after step 3, you absolutely want step 4 to complete, otherwise you are $1200 out of pocket and you *still* owe the rent!  So steps 3 and 4 *must* either both happen successfully, or not at all.  If the network goes down between steps 3 and 4 such that the deposit cannot complete, you want to undo the withdrawal and try again later.  Steps 3 and 4 must be performed atomically.

While this example traditionally applies to database operations (and is thus resolved on-disk), the same concept applies to in-memory operations that must be atomic.  A *mutex* (for "mutual exclusion") or *critical section* can be used to serialise access to resources within a region of code that must run as an atomic operation. We would lock the mutex before the transaction, and unlock it afterwards.

Problem: Race Conditions
========================

A *race condition* is a general term for a class of problems, whereby the result of an operation is at the mercy of the timing of concurrent events (typically unknown or effectively random).  This is bad because your program becomes non-deterministic - it may not always produce the correct result!  They represent one of the most subtle and difficult to debug problems in software development, which is one of the main reasons why multi-threaded programming is considered difficult.

Example: A Counter
------------------

Even a simple increment operation, which is only a single operation in C++, may actually be the source of a race condition.  Consider:

    m_SequenceNumber++;

which actually translates (on the x86 architecture) to something like:

	movl	-12(%ebp), %eax
	incl	%eax
	movl	%eax, -12(%ebp)

which is a load, increment, then store.  So there are actually *two* potential points that a race condition could occur - between the load and the increment, and between the increment and the save.

Now imagine two threads, which are both using a shared object.  If both threads happen to call the method that performs the increment method at the same time, one thread could be preempted in the middle, such that the instructions are interleaved.  So for threads A and B incrementing the sequence number, thus:

<table>
  <thead>
    <th>Thread</th>
    <th>Instruction</th>
    <th>m_SequenceNumber</th>
    <th>Register</th>
  </thead>
  <tbody>
    <tr><td>A</td><td><tt>LOAD m_SequenceNumber, R0</tt></td><td>234</td><td>234</td></tr>
    <tr><td>A</td><td><tt>INCR R0</tt></td><td> 234 </td><td> 235</td></tr>
    <tr><td>B</td><td><tt>LOAD m_SequenceNumber, R1</tt></td><td>234</td><td>234</td></tr>
    <tr><td>B</td><td><tt>INCR R1</tt></td><td> 234 </td><td> 235</td></tr>
    <tr><td>B</td><td><tt>STORE R1, m_SequenceNumber</tt></td><td>235</td><td>235</td></tr>
    <tr><td>A</td><td><tt>STORE R0, m_SequenceNumber</tt></td><td>235</td><td>235</td></tr>
  </tbody>
</table>

So, instead of the sequence number being incremented twice, and being 236 as we would expect, the sequence number is only 235.  This is a subtle, nasty and rare bug and could be very difficult to track down for the unprepared.

Fortunately, there is a nice, simple solution to this particular problem.  Operating systems provide atomic functions for safely performing an increment (with a minimum of overhead), as well as a few other common operations.  Under Windows, you would use the [InterlockedIncrement()](http://msdn.microsoft.com/en-us/library/ms683614\(VS.85\).aspx), while under Mac OS X you would call [OSAtomicIncrement32()](http://developer.apple.com/mac/library/DOCUMENTATION/Darwin/Reference/ManPages/man3/OSAtomicIncrement32.3.html).  Under Linux, gcc provides [\_\_sync\_fetch\_and\_add()](http://gcc.gnu.org/onlinedocs/gcc-4.1.2/gcc/Atomic-Builtins.html) and friends.

Example: Accessing a Queue
--------------------------

An illustration of a race condition is where multiple threads are removing work items from a queue.  Imagine some code that looks something like this:

    void unsafeWorkerThread()
    {
        while ( unsafeWorkerThreadRunning )
        {
            // Warning: this is not thread-safe!
            if ( !queue.isEmpty() )
            {
                work_t item = queue.pop();
                process(item);
            }
        }
    }

This is not thread-safe!  Even if we assume that the queue operations themselves are atomic (which is usually *not* the case for most container classes!), there is a race condition waiting to trigger a failure.  Can you spot it?

Imagine there are two threads running, and there is one work item in the queue.  When `thread_A` calls `queue.isEmpty()`, it goes into the `if` block, as there is more data to process.  But then the thread is pre-empted, and `thread_B` gets a chance to run.  This time, it removes the work item and processes it.  But now the queue is empty, and `thread_A` thinks it isn't!  When `thread_A` resumes, it will call `queue.pop()` and trigger a stack underflow exception.  (And here, without a `try-catch` block, an exception would also cause the thread to exit without notice.) Now this code could run perfectly well for thousands of iterations, and never trigger these particular failure conditions.  But every so often, the timing will be just right (or rather wrong!) and your application will mysteriously fail.  You can see why many people say threading should be avoided at all costs!  The code is valid, usually works but every so often it fails in an unusual way.

The simplest solution is to protect the queue with a mutex, and lock it around the inner block.  (There's another improvement we can make after we discuss exceptions.)

    void workerThread()
    {
        while ( workerThreadRunning )
        {
            queueMutex.lock();
            &nbsp;
            if ( !queue.isEmpty() )
            {
                work_t item = queue.pop();
                process(item);
            }
            &nbsp;
            queueMutex.unlock();
        }
    }

(There are more esoteric solutions to this problem, such as lock-free data structures, but those are beyond the scope of this series.)

Problem: Deadlocks
==================

In a non-trivial application, there may be several threads and numerous mutexes.  When more than one thread locks more than one mutex, there arises the potential for a condition known as a *deadlock*.  This is where one thread is holding a lock while waiting for another to become available, while a second thread is holding the second lock and waiting for the first lock to become available.  Since they are both waiting for each other to finish, neither can run.  This deadlocked situation can be more involved, whereby several threads form a ring of holding and waiting for locks.  This is illustrated as follows, first in code then as a diagram:

    void threadA()
    {
        while (running)
        {
            mutexOne.lock();
            mutexTwo.lock();
            processStuff();
            mutexOne.unlock();
            mutexTwo.unlock();
        }
    }
    void threadB()
    {
        while (running)
        {
            mutexTwo.lock();
            mutexOne.lock();
            processStuff();
            mutexTwo.unlock();
            mutexOne.unlock();
        }
    }

<img src="http://antonym.org/boost/DeadlockAnimation.gif" alt="DeadlockAnimation.gif" border="0" width="561" height="249" />

Fortunately, the problem of deadlocks can largely be avoided by consistently locking mutexes in the same order, and unlocking them in reverse order.

Problem: Exceptions
===================

Exceptions in C++ can be a very effective mechanism for error handling.  However, like many C++ features, care must be taken, especially in threads.  Because a thread is destroyed when the thread function returns, an uncaught exception can cause the entire thread to exit, without warning or notice.  So if you have a thread that mysteriously disappears, an uncaught exception is a likely culprit.

As with non-threaded code, you should always catch the most specific exception type that you are able to handle.  But if you wish to avoid your thread being killed, you should add a top-level `try-catch` block.  At this point, depending on the application, you may simply log the uncaught exception then exit, or you may resume thread processing (but only if safe to do so!).

A skeleton thread function might look something like:

    void processThread()
    {
        while(keepProcessing)
        {
            try
            {
                // Do actual work
            }
            // Catch more specific exceptions first if you can
            catch (std::exception&amp; exc)
            {
                log("Uncaught exception: " + exc.what());
                // Maybe return here?
            }
        }
    }

Just as an errant exception can cause havoc with threads running, they can also cause problems with mutexes.  The next article shows how to use a *lock guard* to ensure mutexes are unlocked, even if an exception is thrown.

Other Problems
==============

Just to reinforce the idea that multithreaded programming is not a trivial undertaking, there are even more traps for the unwary, for which there is not sufficient time to delve into here.  Issues such as [starvation](http://en.wikipedia.org/wiki/Resource_starvation), [priority inversion](http://en.wikipedia.org/wiki/Priority_inversion), and high scheduling latency are all considerations for [concurrent programming](http://en.wikipedia.org/wiki/Concurrent_programming).  It is definitely worth consulting a good book or three.

Closing thoughts
================

Fortunately, there are well-known solutions to multi-threaded design issues, so it is entirely possible to write robust threaded code, provided one is careful and thoughtful.  Much of it starts with:

 - **Minimise shared state**

In days gone by, a multi-threaded system would often still only be running on a single core.  Certain classes of concurrency bugs were far less likely to happen.  But now multicore systems are standard, and we have truly concurrent execution, which brings into the fray problems such as memory consistency, cache latency, write-throughs, and so on.

Especially now that multicore systems are shipping as standard on desktops and laptops, concurrent programming is required to take advantage of the available processing resources.  So a thorough understanding of multithreading is increasingly important.

Some takeaway points to remember:

 - Minimise shared state
 - Program defensively
 - Assume your threading code can be preempted at *any* time
 - Identify shared resources and protect them with mutexes
 - Identify algorithms that need to be atomic
 - Identify code with dependent results that needs to be atomic
 - Minimise the number of resources shared between threads
 - Minimise the duration of locks
 - Ensure exceptions cannot disrupt synchronisation flow
 - Always acquire and release mutexes in the same order

The next article looks at the types of mutexes, and how to use them.
