---
layout: post
title: "Sorting structures in Go"
category:  go
tags: [go, golang]
---
{% include JB/setup %}

In the transition from early versions of Go to the final 1.0 release, a few things changed.  One of them was the removal of the `Vector` type from the `Containers` package.  I had written some text processing code which used `Vectors` in many places, and had to replace them with arrays.

The old code had a comparison function for sorting, while the new code needed to implement `sort.Interface`.  The details for this are as follows:

	// From sort.go:
	type Interface interface {
		// Len is the number of elements in the collection.
		Len() int
		// Less returns whether the element with index i should sort
		// before the element with index j.
		Less(i, j int) bool
		// Swap swaps the elements with indexes i and j.
		Swap(i, j int)
	}

Let's assume our structure is a simple 2D point:

	type Point struct { x, y float32 }

The first thing to do is replace the `Vector` types with arrays, so we define a new type:

	type PointArray []Point

Now all we have to do is implement the three methods required by the sorting interface (above) in terms of our array type:

	func (pts PointArray) Len() int {
		return len(pts)
	}

	func (pts PointArray) Less(left, right int) bool {
		if pts[left].y == pts[right].y {
			return pts[left].x < pts[right].x
		}
		return pts[left].y < pts[right].y
	}

	func (pts PointArray) Swap(left, right int) {
		pts[left], pts[right] = pts[right], pts[left]
	}

The most important past is the comparison function `Less`.  While the `Len` and `Swap` methods are essentially generic and won't change for different array types, `Less` is how you define the sort order specific to your `struct` type.  Here, we implement sorting of the points in raster order, most suitable for graphics rendering.

To put it all together, we can create an array of points, add some elements to the list and then sort the result:

	func main() {
		var pts PointArray

		pts = append(pts, Point{2.3, 4.5})
		pts = append(pts, Point{5.6, 7.8})
		pts = append(pts, Point{9.8, 4.3})
		pts = append(pts, Point{9.8, 4.4})
		pts = append(pts, Point{9.7, 4.3})

		sort.Sort(pts)

		for i := 0; i < len(pts); i++ {
			fmt.Println("Testing ", pts[i].x, pts[i].y)
		}
	}

Note the use of the `append` function, which is used to mutate the array slice.
