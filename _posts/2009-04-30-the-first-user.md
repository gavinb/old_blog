---
layout: post
title: "The First User"
category: Craft
tags: []
---
{% include JB/setup %}

When you are developing a new piece of software, you typically first spend quite some time setting up your development environment.  As you progress, your application becomes a very cozy inhabitant of this environment, a safe happy cocoon that has evolved as you make lots of small changes to the application or your system.  What could possibly go wrong?

Eventually you will have something to actually test, and the time comes to send your little application out into the "Real World", to be tested in the some unknown, potentially harsh environment for which it may be ill prepared.

Your first user comes along when you run your software on any machine other than the one it was created on.  Getting your application to run on another machine is a very important milestone in the overall lifecycle, and doing it earlier rather than later can help flush out all sorts of potential problems.  (Getting it to build on another machine is a topic for another day.)

So what sort of issues do you need to consider?

## System Requirements

What kind of hardware requirements does it have?  Does it test for system requirements on startup?  Does it produce sensible diagnostic messages if anything is lacking?  What is the working set (typical memory usage) or your application?  Can you estimate memory requirements?  What disk space, beyond the installed files, does it require?  Is it performance-sensitive, and if so, have you profiled it to determine what baseline CPU is required?  Do you require any particular drivers to be present?

## Packaging and Dependencies

Do you need an installer program?  Can it run standalone, from a network drive, or a USB stick? What are the runtime dependencies?  In other words, which shared libraries or DLLs does it need to run?  Do you need to provide the redistributable C or C++ runtime?  What resources does it need (ie. images, sounds, help files, shaders, scripts, etc)?  What configuration settings does it require?  Does it have sensible defaults for any preferences that aren't set in a fresh installation?  Does it use optional features that are only available in a certain version of the OS?  Does it assume that particular files are to be found in a particular directory?  The current directory?  The same directory as the executable?  Any assumptions you make during development will surely be tested.  Just getting the application to launch, let alone run correctly, is the first hurdle.

## Logging

You will most likely be providing a Release build to external users.  However, this typically disables any debug logging you might have placed in your code.  If anything should go wrong (against all odds!), you will really save time if you can leave logging on in a Release build, so you can get your guinea pigs to email you the output in the event of the unthinkable.

## Versioning

Give each release a unique and easily identifiable version number.  Show it in the About box, or even in the title bar.  Embed it in the resource metadata of the executable.  Make it easy to answer the question "which version is <em>that</em> you're running?"  If it is supported, you can even add it as a suffix of the executable filename, so you can have multiple versions in the one directory, which can be quite handy.

## Release Notes

Make sure you describe what features work, what features are missing, and what changed in the most recent release.  Don't forget to describe known bugs, too.  Otherwise you will be peppered with "well X doesn't work!" comments, when you know already, you're just not up to that bit yet!  Also provide at least a date and version number, which can be correlated with the software.  Any bugs that get filed will obviously require this information, and it helps with tagging in your version control system too.

## And more...

As I was writing the above, it became clear that each one of these topics could become an article on its own.  Please comment below if you would like to see more detailed discussion on the above, or have some thoughts to share on the topic.
