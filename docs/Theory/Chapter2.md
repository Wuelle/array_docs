# How are matrices stored in memory?
As mentioned before, matrices are multidimensional. Since computer memory
is one-dimensional, we need to flatten the matrix.
This is where strides come into play. A stride is the number of bytes between elements
in memory.
The array stride is usually equal to the element size. (Unless the matrix is padded)

This means that a minimal array implementation contains the following data:

* The address of the first element
* The stride of the array
* The dimensions of the array

By knowing these three things, we can computer the address of any given index.

There are mainly two different ways to project a n-dimensional array into 1-dimensional space.

## Row-major Ordering (or "c" Ordering)
Concatenate all the rows together. This is the common approach for high-level languages.
It means that for a matrix $A$ of size $m\times{n}$ with a stride of 1, the elements
$A_{i, j}$ and $A_{i, j+1}$ are 1 byte apart and the elements $A_{i, j}$ and $A_{i+1, j}$
are n bytes apart.
Notable languages that use Row-major Ordering are C, C++ and Java.

## Column-major Ordering (or "f" Ordering)
Concatenate all columns together. This approach seems to be less popular.
It means that for a matrix $A$ of size $m\times{n}$ with a stride of 1, the elements
$A_{i, j}$ and $A_{i+1, j}$ are 1 byte apart and the elements $A_{i, j}$ and $A_{i, j+1}$
are m bytes apart.
Notable languages that use Row-major Ordering are Fortran and Julia

There is no difference in performance between the two. 
Both provide element lookup at $O(1)$.
The mapping you choose only affects implementation details,
i.e you should always traverse the array row by row if you use Row-major Ordering.
