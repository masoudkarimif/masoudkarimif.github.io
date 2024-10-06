---
date: "2024-10-05"
title: "Benchmarking Go Sort Functions"
tags: ["go", "sort", "performance", "algorithms"]
author: "masoudkf"
description: "Go's standard library comes with several sort functions to sort a slice. In this post, I'll go over some of them to see which one runs faster."
---

<br/>
<p align="center">
  <img src="https://res.cloudinary.com/mkf/image/upload/v1728180506/gopher-sorting_rzxv7b.jpg" width="600"/>
</p>
<br/>

## Sorting Slices with the Go Standard Library
The standard library comes with several functions to sort a slice. In this post I'll take a look at the following functions and benchmark them to see which one runs faster.

- `sort.Slice`
- `sort.SliceStable`
- `slices.SortFunc`
- `slices.SortStableFunc`
- `slices.Sort`

I'll be sorting a slice of integers in this post. In a later post, I'll add more complex types.

Note: All these functions sort the slice in-place and change the slice. 

---

### A Note on Stable vs. Non-Stable Sorting Algorithms
A stable sorting algorithm retains the relative order of equal items, but a non-stable doesn't guarantee to do that. For example, if you're sorting students based on the grade they got on their midterm and on the unsorted list, `Alice` appears before `John` although they both got 90/100, a stable sort will sort the grades but will make sure that `Alice` still appears before `John`. A non-stable one may change that order and have `John` appear first.

#### Does it Matter?
Although in some occassions that wouldn't be such a big deal, but in other situations it might be important. Take, for example, a recommendation system that shows you the products you may be interested in buying. Based on your preferences, you get shirts first, then pants, then wallets. Now assume you want to sort the recommendation based on price and it just so happens that some shirts have the same price as some pants and wallets. If you do a non-stable sort, the result may show you wallets first, then pants, and then shirts, which is not ideal since you're more interested to buy shirts. However, a stable sort will make sure that if there are shirts, pants, and wallets with the same price in the recommendation, they retain the relative order after the sorting is done; i.e. you will still see shirts and pants first, and then wallets.

On the other hand, if you're sorting a list (slice) of numbers, it won't matter. Generally speaking, non-stable sorting algorithms run a tad faster than stable ones, so it's important to pick the one that matches your use case best.

---

## Sorting Functions From the `sort` Package


### `sort.Slice`
This [function](https://pkg.go.dev/sort#Slice), sorts a slice given a less function. It will panic if the input is not a slice. Here's the signature, where `x` is the slice to be sorted, and `less` is the function that shows if item with index `i` is smaller (less) than item with index `j`:

```go
func Slice(x any, less func(i, j int) bool)
```

Here's an example:

```go
sort.Slice(mySlice, func(i, j int) bool {
	return mySlice[i] < mySlice[j]
})
```

This less function will sort the slice ascendingly. To do the reverse (or descendingly), change the less function to return `false` if item `i` is bigger than item `j`:

```go
sort.Slice(mySlice, func(i, j int) bool {
	return mySlice[i] > mySlice[j]
})
```


### `sort.SliceStable`
Same as `sort.Slice` with only one difference: it performs an stable sort on the slice and therefore retains the relative order of the equal items. Here's the signature:

```go
func SliceStable(x any, less func(i, j int) bool)
```

Everything else, including the usage, is the same as `sort.Slice`


## Sorting Functions From the `slices` Package
Aside from sort functions from the [`sort`](https://pkg.go.dev/sort) package, we have a number of sort functions from the [`slices`](https://pkg.go.dev/slices) package, too.

### `slices.SortFunc`
This is the [function](https://pkg.go.dev/slices#SortFunc) that the Go documentation recommends to be used instead of the older `sort.Slice`. Here's the quote from the docs:

> Note: in many situations, the newer `slices.SortFunc` function is more ergonomic and runs faster [than `sort.Slice`].

Here's the signature:

```go
func SortFunc[S ~[]E, E any](x S, cmp func(a, b E) int)
```

It can sort any slice given a `cmp` function that shows if item `a` is bigger than `b` or not. For an ascending sort, the function must return a negative number if `a < b`, a positive number if `a > b`, and `0` if they're equal. Note that this is a bit different that the less function we saw earlier. That function was supposed to return a `bool`, but this `cmp` function is supposed to return an `int`. Another difference is that the less function takes the indices of two items, but the `cmp` function takes the actual items. So, remember that.


Here's an example:

```go
slices.SortFunc(mySlice, func(a, b int) int {
	return a - b
})
```

If you want to sort the slice in descending order, change `return a - b` to `return b - a`.


### `slices.SortStableFunc`
Same as `slices.SortFunc`, but stable. Here's the signature:

```go
func SortStableFunc[S ~[]E, E any](x S, cmp func(a, b E) int)
```

Everything else is the same as `slices.SortFunc`.


### `slices.Sort`
This is a special [function](https://pkg.go.dev/slices#Sort). First, it can only sort the slice in ascending order. Second, the slice must have items that satisfy the [`cmp.Ordered`](https://pkg.go.dev/cmp#Ordered) constraint. Here's the constraint:

```go
type Ordered interface {
	~int | ~int8 | ~int16 | ~int32 | ~int64 |
		~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 | ~uintptr |
		~float32 | ~float64 |
		~string
}
```

As you can see, it only contains primitive types that support these operators: `< <= >= >`. In other words, if you want to sort your slice in descending order, or sort a slice whose items are not primitive (e.g. they include custom types), you can't use this function and need to use another one such as `slices.SliceFunc`. But for our example here which is sorting a slice of integers, this will work.

---

## Benchmark
For the benchmark, I create a slice of `int` with 1,000,000 random numbers. I use the same seed for all the functions to make sure the benchmark is fair. Here's the code:


{{< gist masoudkarimif c899f7464c1dbf19315abffa93c45763>}}

I ran the benchmark using `go test -bench=. -benchmem -run=X` on a MacBook Air 2024 (with Apple M3 Chip, 8 cores, 16 GB RAM) many times and every time I got the same ranking, although the actual numbers were different. Here's one of them: 

```
goos: darwin
goarch: arm64
pkg: sort-bench
BenchmarkSortSlice-8                         579           2163180 ns/op           31163 B/op          2 allocs/op
BenchmarkSortSliceStable-8                   312           3454498 ns/op           57784 B/op          2 allocs/op
BenchmarkSlicesSortFunc-8                    618           2186051 ns/op           29144 B/op          0 allocs/op
BenchmarkSlicesSortStableFunc-8              493           2334498 ns/op           36534 B/op          0 allocs/op
BenchmarkSlicesSort-8                       2019            575471 ns/op            8921 B/op          0 allocs/op
```

And here's the chart for visual-lovers:


<br/>
<p align="center">
  <img src="https://res.cloudinary.com/mkf/image/upload/v1728179639/bench-2_azlrwa.png" width="600"/>
</p>
<br/>

### Conclusion
One thing that's clear is that the stable sorts run slower than the non-stable ones: `sort.SliceStable` runs slower than `sort.Slice`, and `slices.SortStableFunc` runs slower than `slices.SortFunc`.

Also, `slices.SortStableFunc` runs faster than `sort.SliceStable`, which is what the Go docs promises. However, `slices.SortFunc` seems to be more or less the same as `sort.Slices`. But all in all, and given that this is just one example of one type (`int`), I think we should stick to the documentation and use the newer functions from the `slices` package over the older ones from the `sort` package.

The interesting one here is the `slices.Sort` that runs __way__ faster than the others. It's probably because it doesn't take a function to call on every comparison unlike the other ones; so lower overhead during sorting. But remember that this function only does ascending order and only works for slices of primitive types, such as `int` and `float32`. So, there's a chance we can't use this function in many occassions.

Finally, if you're interested in what is happening behind the scenes for these sort functions, the non-stable ones use a custom Quick Sort, and the stable ones use Merge Sort. Can you find them in the source code [here](https://cs.opensource.google/go/go/+/refs/tags/go1.23.2:src/)?

Here's the one for [`slices.SortFunc`](https://cs.opensource.google/go/go/+/refs/tags/go1.23.2:src/slices/sort.go;l=30).