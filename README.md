# Parallel Malloc
## Project Proposal
### Summary
The goal is to implement a parallel version of malloc compare its speedup with the conventional malloc implementation. This would be done using the GHC machines.

### Background
The project involves studying the different memory allocators present including malloc, tcmalloc, jemalloc and ptmalloc, understanding the trade offs and disadvantages of each. Memory management requires that the programmer provides ways to dynamically allocate portions of memory to programs, when requested, and free it for reuse when it is no longer needed. Having a bad memory allocator can be the reason why there are several contentions in the program. 

Based on the shortcomings of these custom memory allocators, we plan to implement our version of parallel malloc by improving some of these pitfalls as well as optimize it based on the application and access patterns. We expect to use the linked list version of the allocator by making it thread safe and to avoid using locks.

### Challenges
