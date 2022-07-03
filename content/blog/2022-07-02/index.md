---
date: "2022-07-02"
title: "Adapting the python context manager pattern for Go"
category: "programming"
---

I've recently started writing more Go recently, and one pattern I miss from Python was context managers,
which are especially useful when doing IO{{% sidenote %}}IO refers to input/output, which are interactions with resources outside your program like the file system, network calls, or database connections. {{% /sidenote %}}:

```python
with Path("data.txt").open() as f:
    # Read in 5 bytes
    data = f.read(5)
# No need to close the file here
```

This automatically closes the file once the nested `with` block exits.{{% sidenote %}}Python handles this by registering a call to the `__exit__` [magic method]({{< ref "../2021-07-23/index.md" >}}) that runs once the `with` statement exits.{{% /sidenote %}}

In Go, most examples show handling the closing of IO resources by using `defer` statements:

```go
f, err := os.Open("data.txt")
if err != nil {
    panic(err)
}
defer f.Close()

data := make([]byte, 5)
// Read in 5 bytes
numLines, err := f.Read(data)
```

The problem with `defer` however is that it is run only when the function returns, whereas multiple context managers can be used and closed inside the same function in python.

If we were to try and implement the python pattern directly in Go, we might end up with something like this:

```go
func WithFile(path string, fn func(f *os.File)) {
	f, err := os.Open(path)
	if err != nil {
		panic(err)
	}
	defer f.Close()
	fn(f)
}

// Read in 5 bytes
data := make([]byte, 5)
readFn := func(f *os.File) {
	numLines, err := f.Read(data)
}
WithFile("data.txt", readFn)
```

We create a small function so that we return after we've done our work with the file, relying on the `defer`ed call to `f.Close()` to close the file for us.

But this pattern is fragile, complicated, and it's difficult to pass around values correctly while continuing to match the function signature. We also lose control over when the resource is closed - it will _always_ be closed immediately after our function `fn` is called.

Instead, there's a more idomatic, straightforward way to go about this in Go. When we initially open the resource, we _also_ return a function that will handle the freeing of the resource. For example:

```go
func WithFile(path string) (f *os.File, closer func()) {
    f, err := os.Open(path)
    if err != nil {
      panic(err)
    }
    return f, func(){
        f.Close()
    }
}

f, closer := WithFile("data.txt")
// We now can choose to defer, or directly call closer() later
defer closer()

data := make([]byte, 5)
// Read in 5 bytes
numLines, err := f.Read(data)
```

This pattern offers us a _lot_ more flexibility.

We can still `defer` the closing of the file if we want, or we can immediately call the `closer` function once we're done with the file. We can also make multiple calls to `WithFile` inside the same function, or close the files in a different order from the order they were opened.
