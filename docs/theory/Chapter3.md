# Memory Alignment
## What is a Word?
A word is defined as the amount of data the processor can load into its registers 
with a single instruction.
The CPU cannot load data from every memory address. 
Instead, it can only load words starting at an address in memory that is a word boundary.

The size of a word is highly dependent on the CPU Architecture, with 32 and 64bit being the
most common in modern processors.
Because of this limitation, the CPU can only access the memory in chunks, transferring 
one chunk at a time using either a `#!assembly LDR` or a `#!assembly STR` Instruction

## Implications
This means that if a value in memory does not start at a word boundary, the processor will
have to load more words, resulting in more instructions and lower performance.
If the CPU has to perform more than one instruction, the whole process is no longer atomic,
meaning that other processes may interfere with the data as it is being read, causing undefined
behaviour.

## Alignment
A value is aligned if it starts at a memory address that is a multiple of the processors
word size. If that is not the case, it is misaligned.
Aligning your data is always a tradeoff: If you perfectly align everything you will end
up with a lot of padding to fill the empty space to the next word boundary, thereby wasting
memory.
If you tightly pack the data and don't align anything, you will use less memory but decrease
performance.
In general, aligning values is to be **strongly** preferred. While accessing unaligned values
will *only* decrease performance on x86, some ARM chips might fault.



