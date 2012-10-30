---
layout: post
title: "Coding vs Patching"
category: Craft
tags: []
---
{% include JB/setup %}

Something occurred to me the other day, while hacking away on some code in [Emacs](http://www.emacswiki.org/) (the world's greatest editor).  What we, as professional Software Engineers, really do on a daily basis is to work with *patches* (or if you prefer, *diffs*).  Sure, we write lots of code.  But it isn't just random changes - or at least, it shouldn't be!  We're typically managing multiple patches concurrently, probably even multiple source trees in various states of completeness, shepherding them toward release.  So, in my humble opinion, a very important skill in the art of software development is crafting patches (as opposed to simply cutting code).  Yet strangely enough, this perspective and related techniques don't seem to be taught formally.

The common meaning of the word "patch" is to mend something with a piece of cloth, make a temporary electrical connection or a makeshift fix.  This  makes it sound like what we do is write layers of hacks that we plaster on as we go, rather than crafting an elegant solution or extension to the existing features.  So - rather than thinking of a patchwork quilt, imagine instead we are weaving and re-weaving a fine tapestry, trying to make our alterations invisible and seamless.

When you first start programming, you write small programs.  You probably hack on them until they work successfully on your carefully chosen test data, then you clean up most of the mess and move on to the next problem.  The unit of work here is an entire program.  And you're probably producing a continuum of loosely related changes to the codebase, from inception to completion.  You're probably not even using version control (which to me is unthinkable now, even for small projects).  But ultimately you produce a big chunk of code for each completed project.

However, as a professional developer, you are most likely working as part of a team on a large code-base, with a long lifecycle.  There's releases out in the field, and new versions under development.  You are tasked with one or more features to implement, and bugs to fix.  Each of these is a discrete piece of work, with its own scope and impact.  And there are normally constraints in place.  For example, the code must compile without warnings, pass unit tests, go through a peer review process, and so on.  So from the time you change the first line, to the time you commit your changes, you are effectively working with a patch or diff against the main source tree.

The patch goes through its own lifecycle too.  Consider the patch as a constantly evolving set of changes applied to the clean checked out version.  Initially, it may be messy and experimental, contain tracing and debug statements, extra assertions, code that gets `#ifdef`'ed out, potential fixes that turn out to be unnecessary, and finally changes that you actually want to keep as they fix the problem or add the feature required.  And these changes also will need cleaning up and commenting before being checked in.  The code is in a constant state of flux, and so is the patch.  But the goal is always to have a clean, discrete patch by the end of the process to commit.

If you went about your day making changes to every possibly relevant file, intermingling changes for multiple features and bugs you had on your task list, your workspace would be a complete mess.  You would be unable to readily isolate the changes for a bug fix, so that you could make a point release for a shipping product.  You would be unable to produce a discrete set of changes in order to put up your code for review.  The changes required to implement or fix these issues may touch multiple source files, and you need to ensure that the code does not regress (ie. break something that worked before), and that the changes make sense.

It is worth noting at this point that this discussion is conceptually at a level above version control systems, which I have avoided discussing (until how).  However it becomes clear that good tool support is a must for managing these patches.  This is why [Mercurial](http://www.selenic.com/mercurial/) and [git](http://www.kernel.org/pub/software/scm/git/) are gaining such a great deal of traction these days.  We spend so much time managing patches, tidying them up, queueing them for review, applying them to different trees, and so on.  So tools that facilitate the process of managing patches save us a great deal of time and effort (not to mention helping avoid mistakes!).

Good Practices
==============

I have tried to collect here some good practices for producing patches, as I have not seen this detailed elsewhere (that I can recall).  If you have suggestions of your own, please contribute in the comments.  This isn't intended to be a comprehensive list, let alone prescriptive, but hopefully a good start.

 * A patch should address a single feature or bug fix.  Do not mix different changes in the one patch.
 * Do not make unnecessary or irrelevant changes (even whitespace, unless required).
 * Ensure there is no whitespace inadvertently changed (eg. tabs to spaces) or added (eg. to the ends of lines).
 * Do not leave temporary test code or debug code in the patch (unless it is really destined to be committed).
 * Run the `diff` command in your VCS frequently, to see just what changes are in your working tree.
 * Try to make changes in small, localised chunks where this can be done cleanly.
 * Look at the diff from the perspective of a reviewer.  If you have 30 single line changes, it is going to be harder to see what is actually changed versus 9 lines removed and 21 lines added.
 * Ensure that you follow the coding conventions of the code that you are patching.  Maintainability is badly affected by a mishmash of different coding styles.
 * Run unit tests to ensure the patch does not break (regress) existing functionality.
 * Write unit tests for bugs and add them to the suite as part of the patch, so regressions can be caught down the track.
 * Document your changes, adding comments as appropriate to make it clear *why* you made the changes.
 * Take the time to write a coherent description in the checkin message in your VCS.  Reference the bug or feature number for traceability.

## Bad Patches ##



## Good Patches ##

## Good Gardening ##
