# VVMalloc
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

### Platform Choice

We intend to use the GHC machines for development and do a comparison based on architecture using the shark machines as well as the PSC machines and if possible on the laptops which we use as well.


### Schedule

| Week | Goal |  
|---|---|
| 1  | Make the 213 malloc code threadsafe and modify the mdriver to spawn threads for each trace   | 
| 2  | Continue to modify driver for timing and utilization analysis. Literature review for existing all allocators to understand their implementation and pitfalls|
| 3  |Study lock free implementation of stacks and linked lists and integrate the same with the thread-safe malloc code | 
| 4  | Continue lock-free implementation | 
| 5  | Optimize and validate the implmentation with the test scripts/ trace files  | 

## Milestone

Following our schedule, we started by modifying the 213 code to setup the infrastructure for further development. The main idea here before trying to implment lock free linked lists was to first implement multithreading by naively locking the data structures. Although the 213 code has mini blocks as well as segmented lists, we use a common global lock to ensure that if each thread picks up a trace and validates it, we are still able to obtain the correct results. After having these global locks for the data structures, the next part of it was to emulate the same for calculating utilization and time taken. Utilization was fairly straightforward as compared to the timing calculation whose implementation in the 213 code was not thread safe. So we went ahead and made it thread safe and compare the serialized code with the multithreaded locked version. The challenging part of it was to understand the mdriver file itself which took us a few days to know what is going on before we could modify it. 

The next part of our work involved studying existing memory allocators. We went through the documentation details regarding the hoard allocator, TCMalloc and JEMalloc. Each of them have slightly different approaches to a common problem of memory allocation taking a very long time and thus creating a bottleneck in terms of performance. Listed below are some of the distinct features and takeaways from studying these allocators.

### **Hoard Allocator**
Hoard maintains per-processor heaps and one global heap. When a per-processor heap’s usage drops below a certain fraction, Hoard transfers a large fixed size chunk of its memory from the per-processor heap to the global heap, where it is then available for reuse by another processor. [1]

The Hoard Allocator mainly addresses 3 negative aspects of conventional memory allocations being
* Contention - When multiple threads allocate and deallocate memory, most allocators serialize this which is the primary bottleneck. This causes contention and the hoard allocator eliminates this bottleneck.
* False Sharing - Threads on different CPUs can end up with memory in the same cache line. Say two processors access different parts of the same cache line. Although they do not modify each others data, the fact that it is being shared can cause the processor to reload the entire line each time which takes 100s of cycles more. This is false sharing and is designed to be prevented using a Hoard Allocator.
* Blowup - This is a scenario when the memory consumption is much more than the actual required memory. This can also be prevented using a Hoard allocator.

### **JEMalloc**
This is another general purpose memory allocator which emphasizes on avoiding fragmentation and allows for concurrency which can be scaled up. JEMalloc uses different arenas and thread local caches to avoid contention and uses red black trees and an optimized slab allocator to avoid fragmentation. [2]

### **TCMalloc**
TCMalloc assigns each thread a thread-local cache. Small allocations are satisfied from the thread-local cache. Objects are moved from central data structures into a thread-local cache as needed, and periodic garbage collections are used to migrate memory back from a thread-local cache into the central data structures. [4]

![image](https://user-images.githubusercontent.com/80923050/162667535-ac1538b8-75a6-4a9d-8fc9-73d0f0e15c68.png)

TCMalloc performs the action of allocation based on the size of the request. If the requested data is below a certain threshold, then the allocator services it from the local thread cache and if this request is above a certain threshold, then it services it from the central heap as show in the above diagram. 

This allocation of small sized objects is somewhat closely related to the 213 implementation of segregated lists. TCMalloc allocates a class to an object based on the range of size it falls under. For instance if the size of the object is 963 bytes, it is rounded up to the next available class of 1024 bytes. This is the idea behind a segregated list as well. <br/>
For larger object sizes, the same idea is scaled up to pages. The heap managed by TCMalloc consists of a set of pages. A run of contiguous pages is represented by a Span object. A span can either be allocated, or free. If free, the span is one of the entries in a page heap linked-list. If allocated, it is either a large object that has been handed off to the application, or a run of pages that have been split up into a sequence of small objects. If split into small objects, the size-class of the objects is recorded in the span. [4]
When an object is deallocated, the corresponding span object is looked up. This provides the information regarding the object size regarding which class it belongs to or which page it belongs to in case they are large objects. In case these are small objects, the chunk of memory is added to the free objects list in the linked list of the per thread cache. In case it is above a certain size, then a garbage collector is run from time to time which moves the unused objects to the central free list. <br/>

Given these features of TCMalloc and the baseline infrastructure setup of our malloc drawn from 213 base code implementation, we believe that this is the way forward. We would ideally want to somehow integrate TCMalloc to our infrastructure as well develop our version of malloc further by implmenting lock free data structures and then proceed with further optimizations in order to compare this with TCMalloc. 

### **Lock Free Implementatiopm**
The current thread safe version of our custom malloc has been implemented naively using single mutex locks around the critical section. As this implentation creates a major bottleneck in a mutithreaded process, our next step would be to modify the current thread-safe code to make major portions of the code flow lock free. We are planning to achieve this by following the atomic compare-and-swap method as covered in the PCA lectures. Our custom malloc consists of segregated free lists along with a singly linked free list to store the mini blocks. The lists are not the only shared structures in our implementation. Considering this, the challenges we are likely to face while adopting a lock-free implementation are:

1. For every malloc and free request, the block headers have to be modified right after inserting or deleting a node from the list. As the heap is a shared structure, this operation too belongs to the critical section. Avoiding locks for the same would require an approach different from the conventional use of CAS. 
2. To optimize memory utilization, we perform coalescing after every free and malloc request. As coalescing is part of the criticial section, we need to check the possibility of using CAS for this part of the code and analyse the complexity of it's implementation. We will also try to explore other ways to emliminate locks given the scenario that CAS is not suitable.  
3. Other than eliminating locks,the existing malloc code need to be modified in order to make a lock free implementation possible. We will also need to analyze how the future modification would impact the malloc performance in terms of utilization and throughput.
4. As we are dealing with stacks here, one of the major concerns is the ABA problem which needs to be catered for during the CAS implemention.

### **Updated Goals and Deliverables**

We see the progress so far to be on track with respect to what we planned to achieve. Although this is true, we felt that it was better to revisit our goals given that we now have a clearer picture in terms of understanding the challenges, the amount of work that is needed for adaption, literature survey and other such tasks revolving around this project. Listed below are the goals and deliverables:

_Plan to achieve_

1. Write test scripts or modify the existing 213 trace files in order to have a complete testing infrastructure.
2. Literature survey on lock free data structures, ABA problem, memory allocators and hazard pointers.
3. Implement a lock free linked list or a linked list with reduced number of locks in a multi threaded environment with sufficient testing.
4. Try the implmented data structure with machines of different architecutres such as the GHC, PSC, Shark machines.
5. Compare the implementation of our memory allocator with TCMalloc and the baseline implementaion and draw inferences from the same regarding different performance metrics.

_Hope to achieve_

1. Thread local linked lists to reduce contention and false sharing.
2. Based on the performance analysis of our allocator v/s TCMalloc, we plan to optimize the code to perform better or understand the gap between our implementation and TCMalloc.

_Poster Session_

We see the poster session to be an opportunity for us to explain the different approaches taken as well as to have sufficient data and comparison charts showing the performance of the baseline malloc, TCMalloc and our version of the updated malloc in terms of different metrics. This will also hopefully be extended to check performance on different machines and try to infer results from here as well. 

### **Timeline**

| Dates      | Task | Completion status    |
| :---        |    :----:   |          ---: |
| 12/4 - 14/4      | Literature review for lock-free implementation to build on PCA lectures    |  |
| 15/4 - 16/4 | Modifying the baseline implementation of malloc code to adopt CAS implementation        |      |
| 17/4 - 22/4   | CAS implementation for linked lists and stacks along with tackling the ABA problem       |      |
| 23/4 - 24/4   | Run performance analysis        |       |
| 25/4 - 27/4   | Decide further optimizations based on current performance        |      |
| 28/4 - 29/4   | Write extensive final report        |      |



## References
[1] https://people.cs.umass.edu/~emery/pubs/berger-asplos2000.pdf <br/>
[2] https://codearcana.com/posts/2012/05/11/analysis-of-a-parallel-memory-allocator.html <br/>
[3] https://github.com/jemalloc/jemalloc <br/>
