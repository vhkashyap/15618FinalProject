# Parallel Malloc
## Project Proposal
### Summary
The goal is to implement a parallel version of malloc and compare its speedup with the conventional malloc implementation. This would be done using the GHC machines.

### Background
The project involves studying the different memory allocators present including malloc, tcmalloc, jemalloc and ptmalloc, understanding the trade offs and disadvantages of each. Memory management requires that the programmer provides ways to dynamically allocate portions of memory to programs, when requested, and free it for reuse when it is no longer needed. Having a bad memory allocator can be the reason why there are several contentions in the program. 

Based on the shortcomings of these custom memory allocators, we plan to implement our version of parallel malloc by improving some of these pitfalls as well as optimize it based on the application and access patterns. We expect to use the linked list version of the allocator by making it thread safe and to avoid using locks.

### Challenges
The first challenge would be to adopt the different implementations of malloc, check the feasibility with the test scripts and performance tools to identify the shortcomings of each implementation. The other challenge is to implement the malloc itself by making it thread safe as well as to use minimal locks while accessing the linked list. 

We feel that using the malloc code from the 213 course is a good starting point in terms of the implementation itself. The challenge here though would be the testing scripts, comparing performance on different machines and to develop a solution that suits most problem statements in terms of performance. Parallelizing this implementation to restrict contention along with maximizing the throughput and reducing fragmentation is an uphill task. 

### Goals and Deliverables

1. Implement a lock free linked list or a linked list with reduced number of locks in a multi threaded environment with sufficient testing.
2. Try the implmented data structure with machines of different architecutres such as the GHC, PSC, Shark machines.
3. Write test scripts or modify the existing 213 trace files in order to have a complete testing infrastructure.
4. Detailed analysis of the different customized malloc implmentations available such as jemalloc, tcmalloc, hoard allocator etc. to understand the advantages, disadvantages and how they are different from the conventional implmentation of malloc.
5. Identifying the pitfalls from the above allocators and trying to implement a version of malloc which overcomes some of these downsides. 
6. Improve the malloc implmented in the previous step by parallelizing portions of it.
7. Scale up parallelization in order to obtain significant speedup all while reducing contention in a shared memory environment.
8. Use data structures other than linked list and observe how well/worse the allocator works.
