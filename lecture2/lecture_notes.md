# RPC and Threads
## Why Golang?

* nice in built RPC package
* GC - Combinationn of Threads and GC is particularly important. It is always an issue for cleanup in case of non GC language.

## Threads

* main tool to manage concurrency
* one program  needs to talk to a bunch of several systems
* goroutines = threads
* thread means a speperate set of program counters and stack
* stacks are in one address space

* ### Reasons to use Threads
  * I/O Concurrency: Threads waiting to I/O operations
  * MultiCore Parallelism: Compute heavy tasks consume more CPU cycles. Goroutines which run these on cores and truly in parallel.
  * Servers might need the above to be able to use CPU parallelism.
  * Background/Periodic Convenience: Workers still alive etc.
* Event Driven programming is a single threaded program. Waiting for events listeners. Async programming is an event driven.
* Threads are generally more convenient.
* Event driven does give us I/O Concurrency but not CPU parallelism.
* Threads might get quite expensive if we have tonnes of them.
* In above cases event driven comes in as useful stripped version of code.
* Threads vs Process: Process contains Threads. One Unix Process houses several memory areas for each thread.
* Threads are able to share same memory space within a program/process.
* ### Challenges with threads
  * Shared Data between threads, say shared Cache read and update.
  * Race conditions
  * If one of the CPU's has started executing then it is a race with the other CPU as to who can finish faster
  * Locks help us to solve this, some strategy around locking as a developer
  * Coordination Threads interacitng with one another
  * in GoLang we have channels and Sync.Conditions and waitGroups
  * DeadLock: general problem T1 -> T2 (waiting for, to release lock) -> T1( to do something)