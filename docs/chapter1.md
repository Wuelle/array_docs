# What is a matrix

A matrix is a multidimensional collection of objects.


## How are matrices stored in memory?
As mentioned before, matrices are multidimensional. Since computer memory
is one-dimensional, we need to flatten the matrix.
This is where strides come into play. A stride is the number of bytes between elements
in memory.
This is usually equal to the element size. (Unless the matrix is padded)
This means that for a matrix $A$ of size $m\times{n}$ with a stride of 1, the elements
$A_{i, j}$ and $A_{i+1, j}$ are 1 byte apart and the elements $A_{i, j}$ and $A_{i, j+1}$
are m bytes apart.
