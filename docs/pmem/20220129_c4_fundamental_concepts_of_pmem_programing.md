## Fundamental Concepts of Persistent Memory Programming

### What’s Different?
What’s different about persistent memory is, of course, that it is persistent, so all the considerations of both storage and memory apply. The application is responsible for maintaining consistent 
data structures between runs and reboots, as well as the thread- safe locking used with memory-resident data structures.

If persistent memory has these attributes and requirements just like storage, why not use code developed over the years for storage? This approach does work; using the storage APIs on persistent memory is part of the programming model we described
in Chapter 3. If the existing storage APIs on persistent memory are fast enough and meet the application’s needs, then no further work is necessary. But to fully leverage the advantages of persistent memory, where data structures are read and written in place on persistence and accesses happen at the byte granularity, instead of using the block storage stack, applications will want to memory map it and access it directly. This eliminates the buffer-based storage APIs in the data path.



### Atomic Updates
Each platform supporting persistent memory will have a set of native memory operations that are atomic. On Intel hardware, the atomic persistent store is 8 bytes. Thus, if the program or system crashes while an aligned 8-byte store to persistent memory is in-flight, on recovery those 8 bytes will either contain the old contents or the new contents.

### Transactions
Combining multiple operations into a single atomic operation is usually referred to as a transaction. In the database world, the acronym ACID describes the properties of a transaction: atomicity, consistency, isolation, and durability.

### Atomicity
As described earlier, atomicity is when multiple operations are composed into a single atomic action that either happens entirely or does not happen at all, even in the face of system failure.

For persistent memory, the most common techniques used are
* Redo logging, where the full change is first written to a log, so during recovery, it can be rolled forward if interrupted.
* Undo logging, where information is logged that allows a partially done change to be rolled back during recovery.
* Atomic pointer updates, where a change is made active by updating a single pointer atomically, usually changing it from pointing to old data to new data.


### Consistency
Consistency means that a transaction can only move a data structure from one valid state to another. For persistent memory, programmers usually find that the locking they use to make updates thread-safe often indicates consistency points as well. If it is not valid for a thread to see an intermediate state, locking prevents it from happening, and when it is safe to drop the lock, that is because it is safe for another thread to observe the current state of the data structure.

### Isolation
Multithreaded (concurrent) execution is commonplace in modern applications. When making transactional updates, the isolation is what allows the concurrent updates
to have the same effect as if they were executed sequentially. At runtime, isolation
for persistent memory updates is typically achieved by locking. Since the memory is persistent, the isolation must be considered for transactions that were in-flight when the application was interrupted. Persistent memory programmers typically detect
this situation on restart and roll partially done transactions forward or backward appropriately before allowing general-purpose threads access to the data structures.

### Durability
A transaction is considered durable if it is on persistent media when it is complete. Even if the system loses power or crashes at that point, the transaction remains completed.

For persistent memory, the same is true due to the programming model described in Chapter 3, where persistent memory is accessed by first opening a file on a direct access (DAX) file system and then memory mapping that file. However, a memory-mapped file is just a range of raw data;

### Flushing Is Not Transactional
It is important to separate the ideas of flushing to persistence from transactional updates. Flushing changes to storage using calls like msync() or fsync() on Linux
and FlushFileBuffers() on Windows have never provided transactional updates.

It is important to think of the cache flush operation as flush anything that hasn’t already been flushed and not as flush all my changes now.

### Start-Time Responsibilities
As a programmer, you may be tempted to map persistent memory and start using it, as shown in the Chapter 3 examples. For production-quality programming, you want to ensure these start-time responsibilities are met. For example, if you skip the checks in Figure 2-5, you will end up with an application that flushes CPU caches even when it is not required, and that will perform poorly on hardware that does not need the flushing. If you skip the checks in Figure 2-6, you will have an application that ignores media errors and may use corrupted data resulting in unpredictable and undefined behavior.

