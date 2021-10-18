# Multidimensional Arrays
## How it works
[Rust 1.51](https://blog.rust-lang.org/2021/03/25/Rust-1.51.0.html) introduced const generics,
which allow types that are generic over **values** instead of **types**.
This allows for efficient multidimensional array implementations. Every array has $N$ dimensions
with possibly different sizes. This can be represented using `#!rust [usize; N]` which is a usize-
array with $N$ elements. Implementing dimensions this way changes the `#!rust Array<T>` struct definition
to the following:

``` rust linenums="3"
/// A n-dimensional array
pub struct Array<T, const N: usize> {
    /// Pointer to the first element
    ptr: *mut T,
    /// Number of bytes between two elements
    stride: usize,
    /// Axis sizes
    dim: [usize; N],
}
```

## Indexing
Having arrays be n-dimensional means that to access a specific element, we need $N$ indices.
I decided to keep the old `#!rust Array::get` method but rename it to `#!rust Array::_get`
and make it private. The new `#!rust Array::get` takes an `#!rust [usize; N]` and converts
it to a one dimensional offset using `#!rust Array::_get_internal_ix`, then uses that
offset to get a reference to the element using `#!rust Array::_get`.

## Problems using const generics
One thing I haven't quite figured out yet is how to implement an `#!rust Array::reshape` function,
which converts `#!rust Array<T, N>` into `#!rust Array<T, M>`. This seems quite simple
at first (no data needs to be moved) until you realize const generics are constant.
(Who would have thought?), meaning you can't just change the value of $N$ at runtime, forcing
you to allocate an entirely new array.
It is probably possible to change N using unsafe code, but it won't be pretty.

## Other stuff i did
I added lots of documentation and unit tests. The crate actually requires you to document
every publicly exposed object and comment every `#!rust unsafe` block now.
