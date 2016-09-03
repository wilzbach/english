# Slices

Slices are objects from type `T[]` for any given type `T`.
Slices provide a view on a subset of an array
of `T` values - or just point to the whole array.
**Slices and dynamic arrays are the same.**

A slice consists of two members - a pointer to the starting element and the
length of the slice:

    T* ptr;
    size_t length; // unsigned 32 bit on 32bit, unsigned 64 bit on 64bit

### Allocating a new slice

If a new dynamic array is created, a slice to this freshly
allocated memory is returned:

    auto arr = new int[5];
    assert(arr.length == 5); // memory referenced in arr.ptr

The actual allocated memory is completely managed by the garbage
collector, the returned slice acts as a "view" on the underlying elements.

### Obtaining a slice to existing memory

Using the slicing operator one can also get a slice pointing to already
existing memory. Slicing operator can be applied to any existing slice, static
arrays, structs/classes implementing `opSlice` and few other entities.

In the example expression `origin[start .. end]` the slicing operator is used to get
a slice of all elements of `origin` from `start` to the element _before_ `end`:

    auto newArr = arr[1 .. 4]; // index 4 ist NOT included
    assert(newArr.length == 3);
    newArr[0] = 10; // changes newArr[0] aka arr[1]

Such slices generate a new view on existing memory. They *don't* create
a new copy. If no slice holds a reference to that memory anymore - or a *sliced*
part of it - it will be freed by the garbage collector.

Using slices it's possible to write very efficient code for e.g. parsers
that operate on one memory block and just slice the parts they
work on - without needing to allocate new memory blocks.

As seen in the previous section the `[$]` expression is a shorthand form for
`arr.length`. Hence `arr[$]` indexes the element one past the slice's end and
thus would generate a `RangeError` (if bound-checking hasn't been disabled).

### In-depth

- [Introduction to Slices in D](http://dlang.org/d-array-article.html)
- [Slices in _Programming in D_](http://ddili.org/ders/d.en/slices.html)

## {SourceCode}

```d
import std.stdio;

/**
Calculates the minimum of all values
in a slice recursively. For every recursive
call a sub-slice is taken thus we don't
create a copy and don't do any allocations.
*/
int minimum(int[] slice)
{
    assert(slice.length > 0);
    if (slice.length == 1)
        return slice[0];
    auto otherMin = minimum(slice[1 .. $]);
    return slice[0] < otherMin ?
        slice[0] : otherMin;
}

void main()
{
    int[] test = [ 3, 9, 11, 7, 2, 76, 90, 6 ];
    auto min = minimum(test);
    writefln("The minimum of %s is %d",
        test, min);
    assert(min == 2);
}
```
