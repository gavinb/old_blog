---
layout: post
title: "Good Code Gardening"
category: 
tags: []
---
{% include JB/setup %}

During a code review at work recently, we had an interesting discussion about code maintenance.  You could say that coding is a bit like gardening: while you are planting new seedlings, do you weed nearby areas as you go, or save up all the weeding for the next sunny weekend?  Should code maintenance be a continual, gradual process, or does it warrant being scheduled as a task in its own right?

First, let us define "code maintenance" as editing source code to make *non-functional changes*, such as:

- fixing formatting
- fixing whitespace
- improving consistency
- correcting typos
- commenting
- minor refactoring
- removing dead code

In general, these sorts of changes are improving code hygiene, tasks which are not normally scheduled or allocated.

Now, the problem is this: imagine you are working on a large codebase, which has been around for quite some time, and has had a variety of authors modifying it.  As you are working on a new feature, you are adding new code, and refactoring existing code to enable the new code to work.  Suppose you come across some old, dead code. Should you remove it right now?  You see an obvious typo in an unrelated comment.  Should you correct it here?  You notice some inconsistently formatted code.  Should you update its layout before moving on with your main task?

There are only two answers to this seemingly simple question, but there are many important issues that surround it.  Either you fix it now (garden as you go), or save it for later (do it all on Weeding Day).

## Garden as you Go ##

Why should do tidy your code as you go?

 * You may forget to come back and do it later
 * You might run out of time to do later, as another bug or feature will take priority
 * A patch with a few extra whitespace changes or comment edits is trivial and takes hardly any time to review
 * As the code gradually expands, it is incrementally and constantly improved
 * Code hygiene improves code quality

## Weeding Day ##

Why should you do your weeding separately?

 * Mixing code maintenance in with new code in a single patch obscures the actual intent, and the patch is no longer one discrete piece of work
 * Unrelated changes may make it more difficult to merge
 * Reviewing code for unrelated changes can waste reviewers' time
 * Maintenance is important, and should be scheduled as a discrete work item
 * Maintenance needs to make the code more consistent, so it should be done in one hit

## The Verdict ##

The conclusion I came to is a kind of compromise, but ultimately driven by the notion that the sanctity of the patch is the most important factor.  One patch should always be a discrete piece of work, so mixing new features and gardening/maintenance into one patch is a bad idea.

**But** the argument that it is very hard to *schedule* maintenance time is a compelling one, as there is invariably a huge backlog of bug fixes or new features waiting which take priority.  With all the best intentions in the world, most managers will rather have you working on new features and bug fixes than touching code that already works.  So maintenance doesn't get scheduled.

So if you don't perform code maintenance as you go, when do you do it?  You do it as you go!  Let me explain...

You perform your code gardening/maintenance **as you go**, but you commit your maintenance changes **separately**.  Feature work goes into one patch (or changeset or commit), bug fixes go into their own patch (maybe with a regression test), and maintenance goes in its own patch.  That way, each patch can be reviewed without any distractions, merges are simpler, and you still get continual improvement.

## Howto: In Depth ##

A good craftsperson has good tools, and my proposed solution requires good tool support to be truly effective.  Ideally, You will be using a powerful version control system (VCS) such as [Mercurial](http://mercurial.selenic.com/) or [git](http://git.kernel.org/) (unfortunately [Subversion](http://subversion.tigris.org/) does not really do what we need, though it can be wrangled with some effort).  Both these tools allow you to check in changes not only at the level of an individual file, but at the level of changed *hunks within a file*.  This lets you more easily manage a *subset* of the current outstanding changes in your source tree.  So the idea is that you can go ahead and do maintenance all over the place, knowing your patch will end up nice and clean and separate from the maintenance work.

So how do you this?  Imagine you have a tool with the following files:

    fileio.c
    fileio.h
    parser.c
    parser.h
    main.c
    utils.c
    utils.h

You need to modify the file loading support to fix a bug with handling metadata in the headers.  So you modify `fileio.c` and `fileio.h`.  Along the way, you need to modify `render.c`, and along the way add some debug code to the rendering code.  This last change is of course throwaway test code that you will never check in.  And while you're hacking away adding tracing statements, you notice some dodgy comments, which don't match with the code.  So you fix those, and keep going - only to find someone managed to insert some hard tabs in there by mistake (after all, who would do that on *purpose*?).  So you use the Emacs `untabify` function to clean up that region too.  The renderer code uses some functions in the `utils` module, and you notice the docstring is missing an explanation of the parameters, so you add some `@param` statements while you're there.  Now your VCS will give you the following status (Mercurial output):

    M fileio.c
    M fileio.h
    M render.c
      render.h
      main.c
      utils.c
    M utils.h

And if you looked at the overall diff, it would mix in a bunch of unrelated changes in with your file loading fix.  You're two degrees removed from the original problem by the time you get to `utils`.  But all these changes are worthwhile, and improve the code.  You don't want to lose them.

So you simply don't check in `utils.h` at all.  That's easy (and even `svn` can cope with that).  And you commit all of the changes to `fileio`, as those make up the fix you were working on in the first place.  But what about `render.c`?  You've got no less than *three* types of changes in there:

 * temporary debug code (that will be removed later)
 * maintenance changes (to fix up the comments and whitespace)
 * bug fix changes (part of the file I/O fix)

If you're using Mercurial (my DVCS of choice), you can use the `record` command (a built-in extension)
to easily check in individual hunks (sub-parts of diffs).  You use `record` much like commit, except instead of committing all your outstanding changes, it will interactively prompt you on a per-file and then a per-hunk basis.  You can nominate to include all of a file's changes, skip it entirely, or choose hunks.  This way, you can add both the `fileio` files, skip `utils` and choose only the bug fix changes that ended up in `render.c`.

    $ hg record -m "Bug fix for #123, validate metadata type and length before loading"

You will end up with the changes *just* for the bugfix in this one commit, making it nice and easy to send off to review.  And then you have the other changes remaining:

      fileio.c
      fileio.h
    M render.c
      render.h
      main.c
      utils.c
    M utils.h

which you can then selectively commit at your leisure.

The following animation shows this process for a single file:

<img alt="Gardening.gif" src="http://antonym.org/Gardening.gif" width="393" height="385" class="mt-image-none" style="" />

If you're using `git`, there is a command that performs essentially the same thing.  You need first to add your desired changes to the "index" (or staging area).  You can even stage changes as you go, which is a very powerful (though potentially confusing) model.  This is performed with:

    $ git add --interactive

Once you have chosen the hunks that make up the desired patch, you can commit what is in the staging area and the diff for that changeset will be nice and clean.

Fortunately, there's also GUI support for these operations.  For example, the most excellent [GitX tool](http://gitx.frim.nl/) allows you to not only select individual hunks for commit, you can even split up hunks and control which individual lines are committed (or staged).

This discussion highlights the need for good tools to support your workflows (rather than have your tool dictate your workflow).  Stay tuned for an upcoming article on "Coding vs Patching", exploring the idea that the practice of coding is very much about shepherding clean patches from inception through to review and final merging.

So - go be a Good Gardener...
