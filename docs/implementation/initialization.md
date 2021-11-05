# Cloning and array Initialization
## Filling the array
Up until now, we had to create arrays using `#!rust Array::uninitialized` and manually 
assign values. This is obviously a terrible idea, since its both tedious and unsafe.
To at least have a temporary solution for debugging and further development,
I created the `#!rust Array::fill` function which creates an array where every element
is the same specified value.
Having this function finally allows me to implement traits like `#! Clone` which previously
was not possible, due to requiring clones of uninitialized memory.


## Implementing Clone
It is rare having to implement the `#!rust Clone` trait manually. Usually prefixing 
a type definition with `#!rust #[derive(Clone)]`, which recursively calls
`field.clone` on each field, is sufficient. However, since we are working with raw
pointers, cloning them would make both arrays point to the same memory address, which
obviously isn't what we want.
A custom `#! Clone` implementation for our array type looks like this:

```rust linenums="37"
impl<T: Copy, const N: usize> Clone for Array<T, N> {
    fn clone(&self) -> Self {
        let mut cloned: Self;
        // Safe because we won't be reading from uninitialized memory.
        unsafe {
            cloned = Array::uninitialized(self.dim);
        }

        // Safe because
        // * T is Copy
        // * self.ptr is valid for self.size() reads
        // * cloned.ptr is valid for self.size() writes
        // * both self and cloned are properly aligned
        // * self and cloned do not overlap
        unsafe {
            std::ptr::copy_nonoverlapping(self.ptr, cloned.ptr, self.size());
        }

        cloned
    }
}
```
Note that we only implement `#!rust Clone` if `T` also implements `#!rust Copy`.
This trait bound isn't very restrictive since basically all primitive types implement `#!rust Copy`
(and the ones that do not, like `#!rust String`, are unlikely to be used inside an array).
However, it ensures memory safety because `#!rust std::ptr::copy_nonoverlapping` performs a bitwise
copy **without** ensuring `#!rust Copy`. We don't implement `#!rust Copy` on `#! Array` because it
contains heap data which requires a deep copy (`#!rust Clone`).

## Performance
I actually got curious about the performance difference between calling `#!rust .clone()`
on each element. See [implentation/benchmarking] for more info.

```rust linenums="37"
impl<T: Clone, const N: usize> Clone for Array<T, N> {
    fn clone(&self) -> Self {
        let mut cloned: Self;
        // Safe because we won't be reading from uninitialized memory.
        unsafe {
            cloned = Array::uninitialized(self.dim);
        }

        // clone each element (can probably be done faster)
        for offset in 0..self.size() {
            // safe because offset will never exceed self.size()
            unsafe {
                *cloned._get_mut_unchecked(offset) = self._get_unchecked(offset).clone();
            }
        }
        cloned
    }
}
```
