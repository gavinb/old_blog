---
layout: post
title: "Debugging NSBezierPath drawing"
category: Coding
tags: [Cocoa, Mac, OSX]
---
{% include JB/setup %}

I was working on some Cocoa programming, making heavy use of `NSBezierPath`.  I wished there was an easy way to see just where my control points were ending up, and how the curves were being constructed.  So I wrote a category method to add such a thing to the `NSBezierPath` class.  It is here for all to share.

Here's an example of annotating a curve:

<img src="http://img.skitch.com/20080219-cp95n9wjm391ptfarkja1hg7xf.png" alt="BezTest1"/>

It's very simple, but very handy as you can easily switch from the normal stroking of the path to this annotated version in one line.  It puts blue dots on all the regular points, green dots for the control points, adds lines to join control points to their anchor point, and adds a label so you can see the direction of the control point (so "cp3_2" is anchored at point 3, on the side of point 2.  It also draws the bounding box in red, so you can see the extents of the full path.

<ul><li>Grab the code (MIT license): <a href="http://antonym.org/files/antonym/bezier_debug.zip">bezier_debug.zip</a> (2kB)</li></ul>

Enjoy!  I'd love to get feedback from anyone who tries this.
