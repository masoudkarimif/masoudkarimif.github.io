---
date: "2024-10-07"
title: "Heaps Don't Lie; They Save Lives"
tags: ["go", "heaps", "performance", "algorithms"]
author: "masoudkf"
description: "Go's `container` package helps you create heaps, although not as easy as other languages like Python do. But that makes it more fun."
---

<br/>
<p align="center">
  <img src="https://res.cloudinary.com/mkf/image/upload/v1728266804/Gemini_Generated_Image_ls11eols11eols11_vtfsp9.jpg" width="500"/>
</p>
<br/>

If you're not familiar with (binary) heaps, they're a tree-__like__ data structure backed by arrays (or slices in Go). I've seen in some places where they call heaps tree-__based__ data structures, and I don't think that's accurate. You'll see pretty soon that heaps are basically lists, but the way they hold and arrange items in that list makes them look a lot like trees. You can use trees to describe them, and you can use trees to show all their operations; but at the end of the day, they're just lists, albeit a special one.

Because they act like trees, they can perform operations (like inserts and removes) in `O(logN)` where `N` is the size of the heap (or list). You may also see `O(h)` where `h` is the height of the tree. And because heaps can be shown as a near-complete binary tree, they're balanced and hence their height is `O(logN)` where `N` is the number of nodes. So, basically saying the same thing with a different notation.

### Who Cares?
You may be a software engineer and never have come across heaps in your career, and that's totally fine. It's not like heaps are used in everyday solutions. However, there are some situations where you may like to consider heaps. Situations where, at any time, you're only interested in finding the next maximum or minimum in a list of things. You don't care about the rest of the things, you only care what the maximum or minimum thing is at this very moment. And that thing could be anything. Could be a simple number, or a more complex structure, like an object, or `struct` in Go.

For example, let's say you're implementing a shortest-path algorithm (like the Dijkstra's algorithm). Chances are, at each iteration, you want to get the next smallest path (or edge) to process. Heaps to the rescue! A min-heap can help you out here. It specializes in giving you the next minimum in `O(logN)` time. Fun fact: a very popular implementation of the Dijkstra's algorithm uses a min-heap.

Or say you want to build some software to be used in emergency rooms. Patients come at different times and each has a priority assigned to them based on how critical their situation is. e.g. priority 5 being the highest and most critical situation and priority 1 being the least urgent one. What's important here is to get the next patient in the highest critical situation and assign them to a doctor. Enter max-heaps! A max-heap can help you get the next highest priority patient in `O(logN)` time.

The examples are abound. Another example is task scheduling, or even leaderboards.

If instead of a heap, you use a simple slice (or list), each time you're looking for the next maximum or minimum, you'd need to spend `O(N)` time to find it; so, using heaps could help you achieve better performance. 

## Heaps in Go
Unlike Python, where you can start with a [heap](https://docs.python.org/3/library/heapq.html) pretty quick, Go asks you to do a bit more before having a functional heap. The `container` package comes with a [`heap`](https://pkg.go.dev/container/heap@go1.23.2) package to get you started, but you need to do some of the leg-work.

### Heap Interface
Go lets you turn any slice of primitive or custom type into a heap if you implement the [heap interface](https://pkg.go.dev/container/heap@go1.23.2#Interface):

```go
type Interface interface {
	sort.Interface
	Push(x any) // add x as element Len()
	Pop() any   // remove and return element Len() - 1.
}
```

As you see, the [`sort.Interface`](https://pkg.go.dev/sort#Interface) is also embedded, which means you need to satisfy that as well:

```go
type Interface interface {
	Len() int
	Less(i, j int) bool
	Swap(i, j int)
}
```

So, basically, you implement these 5 functions and you got yourself a heap. Let's build one.

### Emergency Room Heap
Let's build a heap that an emergency room can use. Patients are described by the following struct:

```go
type Patient struct {
	index    int
	priority int
}
```

Where higher priority means more critical. So, in other words, we need a max-heap.

Remember my discussion at the beginning that heaps are basically slices (some say __backed__ by slices), so we need a slice of `Patient`s. Here it is:

```go
type PatientHeap []*Patient
```

I declared it as a new type so I can add method on it to satisfy the interface.

#### Implementing the Heap Interface
Let's implement the interface. Remember that we need to implement 5 functions.

```go
// Len returns the size of the heap.
func (ph PatientHeap) Len() int {
	return len(ph)
}

// Swap swaps item i with item j in the heap.
func (ph PatientHeap) Swap(i, j int) {
	ph[i], ph[j] = ph[j], ph[i]
	ph[i].index = i
	ph[j].index = j
}

// Less returns true if item i is bigger that item j.
// This is how we build a max-heap. To build a min-heap replace > with <.
func (ph PatientHeap) Less(i, j int) bool {
	return ph[i].priority > ph[j].priority
}

// Push adds an item to the heap.
// It simply appends a new item to the slice.
// Needs to be pointer receiver because it 
// changes the size of the slice.
func (ph *PatientHeap) Push(x any) {
	item := x.(*Patient)
	item.index = len(*ph)
	*ph = append(*ph, item)
}

// Pop returns and removes the minimum.
// Needs to be pointer receiver because it 
// changes the size of the slice.
func (ph *PatientHeap) Pop() any {
	n := len(*ph)
	item := (*ph)[n-1]
	item.index = -1
	(*ph)[n-1] = nil
	*ph = (*ph)[0 : n-1]

	return item
}
```

The `Less`, `Len`, and `Swap` functions are very straightforward. The only gotcha is the `Less` function. If you want a max-heap, you do it like the example above; if you want a min-heap, you need to use a `<` operator instead.

The last two functions, `Push` and `Pop`, use a pointer receiver because they resize the underlying slice, so watch out for that. The `Push` function simply appends a new item to the slice, however, the `Pop` function is more complicated. 

At the first sight, you can see it removes and returns the __last__ item of the slice (it also assigns `nil` to the last pointer so GC can pick it up). The part that confused me at the beginning, though, was why do we remove the __last__ item and not first one? 

If you read any article on heaps, you'll see that the item you want to pop, whether it's the min or max, is always at the root of the tree, or first item of the list. Even the Go docs confirms this:

> The minimum element in the tree is the root, at index 0.

But if you try to remove and return the first item, it won't work. It will compile, but you'll get wrong results!

So, I had to look at the source code [here](https://cs.opensource.google/go/go/+/refs/tags/go1.23.2:src/container/heap/heap.go;l=59) to find what was going on behind the scene. Here's the code:

```go
func Pop(h Interface) any {
	n := h.Len() - 1
	h.Swap(0, n)
	down(h, 0, n)
	return h.Pop()
}
```

You see that before calling `Pop` on the heap, the code swaps the root (item at index `0`) with the last item (item at index `h.Len() - 1`). In other words, before __our__ `Pop` function is called, the root, which the item we want to remove and return, is moved to the __end__ of the slice. So, be careful about that. (If you're wondering what `down` does, you can take a look at the source code. But in a nutshell, it heapifies the heap, meaning it makes sure the heap is organized properly now that the root is gone)

#### Using the Heap
Now that we have implemented the heap interface, let's use it:

```go
func main() {
	// Make the heap slice
	ph := make(PatientHeap, 0)
	// Initialize the heap
	heap.Init(&ph)

	for i := 0; i < 100; i++ {
		// Push items into the heap
		heap.Push(&ph, &Patient{
			priority: rand.Intn(10),
			index: i,
		})
	}

	for ph.Len() > 0 {
		// Pop items from the heap
		item := heap.Pop(&ph).(*Patient)
		fmt.Println(item.priority)
	}
}
```

You start by making a slice to back the heap, then initialize the heap using `heap.Init()` and then you can `Push` and `Pop` to and from the heap. Just remember to use the methods through the `heap` package, so `heap.Push()` and `heap.Pop()`.

And there it is. Our beautiful heap, which will save lives at any emergency room. A life-saving heap, if you will.