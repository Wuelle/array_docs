# One-dimensional Array
## Prerequisites
This Implementation Chapter requires understanding of the following Theory Chapters:

* [What is an array?](../theory/Chapter1.md)
* [How are arrays stored in memory?](../theory/Chapter2.md)
* [Memory Alignment](../theory/Chapter3.md)

## Dependencies
Since we are going to be working with raw memory, we need to be able to allocate specific amounts of heap memory.
In rust, this can be done by using the `#!rust std::alloc` crate.
``` rust linenums="1"
use std::alloc::{alloc, dealloc, Layout};
```

## The Array Struct
To keep it simple, we will constrain our array to a single dimension for now (effectively
making it a vector).

``` rust linenums="3"
/// A one dimensional vector
pub struct Array<T> {
    /// Pointer to the first element
    ptr: *mut T,
    /// Number of bytes between two elements
    stride: usize,
    /// Number of elements within the vector
    dim: usize,
}
```

Usually, rust will take care of allocating and deallocating memory as needed. In this case however,
we need to do it ourselves.
To do this, we need to do two things:

* Define a Constructor (to allocate memory)
* Define a Destructor (to deallocate memory)

## Allocation
Allocating memory is actually pretty simple. We just need to tell the compiler

* How much memory to allocate
* How to align the allocated memory
``` rust linenums="13"
impl<T> Array<T> {
    /// Create an uninitialized Array<T>
    pub fn new(len: usize) -> Self {
        let stride = std::mem::size_of::<T>();
        let ptr = unsafe {
            let layout = Layout::from_size_align_unchecked(len, stride);
            alloc(layout) as *mut T
        };
        Self {
            ptr: ptr,
            stride: stride,
            dim: len,
        }
    }

}
```
Using `#!rust Layout::from_size_align_unchecked` is unsafe, but provides perfomance benefits
over `#!rust Layout::from_size_align` because we skip runtime checks.
Not sure yet if i will keep this.


## Deallocation
To control memory deallocation, we need to implement the `#!rust Drop` trait for `#!rust Array`.
This trait only contains the `#!rust Drop::drop` function, which is called when the value
goes out of scope. My implementation looks like this:

``` rust linenums="62"
impl<T> Drop for Array<T> {
    fn drop(&mut self) {
        unsafe {
            dealloc(
                self.ptr as *mut u8,
                Layout::from_size_align_unchecked(self.dim, self.stride),
            )
        };
    }
}
```

## Reading and Writing
Being able to allocate memory is nice, but it's also completely useless as long as we can't read
from it or write to it.
!!! warning
    After calling `#!rust alloc(layout)`, the memory is **not** initialized. This means that
    reading from it will cause undefined behaviour!

When adding to a pointer in rust, it will be incremented with a stepsize equal to the size
of the element it is pointing to. This is convenient, because it means we dont actually
have to multiply `#!rust ix` with `#!rust self.stride`.
``` rust linenums="28"
    /// Get a immutable reference to an object within the array
    pub fn get(&self, ix: usize) -> Option<&T> {
        if ix < self.dim {
            unsafe {
                Some(&*self.ptr.add(ix))
            }
        } else {
            None
        }
    }
```
and the same for a mutable reference:
``` rust linenums="44"
    /// Get a mutable reference to an object stored within the array
    pub fn get_mut(&mut self, ix: usize) -> Option<&mut T> {
        if ix < self.dim {
            unsafe {
                Some(&mut *self.ptr.add(ix))
            }
        } else {
            None
        }
    }
```
I also implemented `#!rust Array::get_unchecked` and `#!rust Array::get_mut_unchecked` which simply
skip the runtime boundary checks.
