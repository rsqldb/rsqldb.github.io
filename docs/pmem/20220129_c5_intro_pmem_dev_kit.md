## Introducing the Persistent Memory Development Kit

To make your job easier, Intel created the Persistent Memory Development Kit (PMDK). 

## Choosing the Right Semantics
The PMDK offers two library categories:
* Volatile libraries are for use cases that only wish to exploit the capacity of persistent memory
* Persistent libraries are for use in software that wishes to implement fail-safe persistent memory algorithms.

## Volatile Libraries
Volatile libraries are typically simpler to use because they can fall back to dynamic random-access memory (DRAM) when persistent memory is not available. This provides a more straightforward implementation.

### libmemkind
The memkind library, called libmemkind, is a user-extensible heap manager built on top of jemalloc. 

It enables control of memory characteristics and partitioning of the heap between different kinds of memory. The kinds of memory are defined by operating system memory policies that have been applied to virtual address ranges.
The jemalloc nonstandard interface has been extended to enable specialized kinds to make requests for virtual memory from the operating system through the memkind partition interface.

* When to use it?
  Choose libmemkind when you want to manually move select memory objects to persistent memory in a volatile application while retaining the traditional programming model. The memkind library provides familiar malloc() and free() semantics.
  This is the recommended memory allocator for most volatile use cases of persistent memory.
  
  When using memkind with file-based kinds, such as PMEM kind, physical space is still only allocated on first access to a page and the other described techniques no longer apply. Memory allocation will fail when there is no memory available to be allocated, so it is important to handle such failures within the application.

### libvmemcache
libvmemcache is an embeddable and lightweight in-memory caching solution that takes full advantage of large-capacity memory, such as persistent memory with direct memory access (DAX), through memory mapping in an efficient and scalable way.

libvmemcache has unique characteristics:
* An extent-based memory allocator sidesteps the fragmentation problem that affects most in-memory databases and allows the cache to achieve very high space utilization for most workloads.
* The buffered least recently used (LRU) algorithm combines a traditional LRU doubly linked list with a non-blocking ring buffer to deliver high degrees of scalability on modern multicore CPUs.
* The critnib indexing structure delivers high performance while being very space efficient.

The cache is tuned to work optimally with relatively large value sizes. The smallest possible size is 256 bytes, but libvmemcache works best if the expected value sizes are above 1 kilobyte.

* When to use it?
  Use libvmemcache when implementing caching for workloads that typically would have low space efficiency when cached using a system with a normal memory allocation scheme.

### libvmem
libvmem is a deprecated predecessor to libmemkind. It is a jemalloc-derived memory allocator, with both metadata and objects allocations placed in file-based mapping. 


## Persistent Libraries
Persistent libraries help applications maintain data structure consistency in the presence of failures. In contrast to the previously described volatile libraries, these provide new semantics and take full advantage of the unique possibilities enabled by persistent memory.

### libpmem
libpmem is a low-level C library that provides basic abstraction over the primitives exposed by the operating system. It automatically detects features available in the platform and chooses the right durability semantics and memory transfer (memcpy()) methods optimized for persistent memory. Most applications will need at least parts of this library.

* When to use it?
  * Use libpmem when modifying an existing application that already uses memory- mapped I/O. Such applications can leverage the persistent memory synchronization primitives, such as user space flushing, to replace msync(), thus reducing the kernel overhead.
  * Also use libpmem when you want to build everything from the ground up. It supports implementation of low-level persistent data structures with custom memory management and recovery logic
    
### libpmemobj
libpmemobj is a C library that provides a transactional object store, with a manual dynamic memory allocator, transactions, and general facilities for persistent memory programming. This library solves many of the commonly encountered algorithmic and data structure problems when programming for persistent memory. 

* When to use it?
Use libpmemobj when the programming language of choice is C and when you need flexibility in terms of data structures design but can use a general-purpose memory allocator and transactions.
  

### libpmemobj-cpp
libpmemobj-cpp, also known as libpmemobj++, is a C++ header-only library that uses the metaprogramming features of C++ to provide a simpler, less error-prone interface to libpmemobj. It enables rapid development of persistent memory applications by reusing many concepts C++ programmers are already familiar with, such as smart pointers and closure-based transactions.

This library also ships with custom-made, STL-compatible data structures and containers, so that application developers do not have to reinvent the basic algorithms for persistent memory.

* When to use it?
  When C++ is an option, libpmemobj-cpp is preferred for general-purpose persistent memory programming over libpmemobj.
  
### libpmemkv
libpmemkv is a generic embedded local key-value store optimized for persistent memory. It is easy to use and ships with many different language integrations, including C, C++, and JavaScript.
This library has a pluggable back end for different storage engines. Thus, it can be used as a volatile library, although it was originally designed primarily to support persistent use cases.

When to use it?
This library is the recommended starting point into the world of persistent memory programming because it is approachable and has a simple interface. Use it when complex and custom data structures are not needed and a generic key-value store interface is enough to solve the current problem.

### libpmemlog
libpmemlog is a C library that implements a persistent memory append-only log file with power fail-safe operations.

* When to use it?
Use libpmemlog when your use case exactly fits into the provided log API; otherwise, a more generic library such as libpmemobj or libpmemobj-cpp might be more useful.

### libpmemblk
libpmemblk is a C library for managing fixed-size arrays of blocks. It provides fail-safe interfaces to update the blocks through buffer-based functions.

* When to use it?
  Use libpmemblk only when a simple array of fixed blocks is needed and direct byte- level access to blocks is not required.
  
## Tools and Command Utilities

### pmempool
The pmempool utility is a tool for managing and offline analysis of persistent memory pools. Its variety of functionalities, useful throughout the entire life cycle of an application, include
* Obtaining information and statistics from a memory pool
* Checking a memory pool’s consistency and repairing it if possible
* Creating memory pools
* Removing/deleting a previously created memory pool
* Updating internal metadata to the latest layout version
* Synchronizing replicas within a poolset
* Modifying internal data structures within a poolset
* Enabling or disabling pool and poolset features

When to use it?
Use pmempool whenever you are creating persistent memory pools for applications using any of the persistent libraries from PMDK.

### pmemcheck
The pmemcheck utility is a Valgrind-based tool for dynamic runtime analysis of common persistent memory errors, such as a missing flush or incorrect use of transactions.

When to use it?
The pmemcheck utility is useful when developing an application using libpmemobj, libpmemobj-cpp, or libpmem because it can help you find bugs that are common in persistent applications. We suggest running error-checking tools early in the lifetime of a codebase to avoid a pileup of hard-to-debug problems. The PMDK developers integrate pmemcheck tests into the continuous integration pipeline of PMDK, and we recommend the same for any persistent applications.

### pmreorder
The pmreorder utility helps detect data structure consistency problems of persistent applications in the presence of failures. It does this by first recording and then replaying the persistent state of the application while verifying consistency of the application’s data structures at any possible intermediate state. 

* When to use it?
Just like pmemcheck, pmreorder is an essential tool for finding hard-to-debug persistent problems and should be integrated into the development and testing cycle of any persistent memory application.