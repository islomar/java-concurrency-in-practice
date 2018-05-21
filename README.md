# Java concurrency in practice

Notes and examples from the book "Java concurrency in practice" by Brian Goetz, published in 2006.

* http://jcip.net/
* Errata: http://jcip.net/errata.html

## Preface
### Chapter 1: Introduction
* OS evolved to allow more than one program to run at once, running individual programs in *processes*: isolated, independently executing programs to which the OS allocates resources such as memory, file handles, and security credentials. If they needed to, **processes could communicate with one another through a variety of coarse-grained communication mechanisms: sockets, signal handlers, shared memory, semaphores, and files.**
* Finding the right balance of sequentiality and synchrony is ofthen a characteristic of efficient people - and the same is true of programs.
* Threads allow multiple streams of program control flow to coexist within a process. **They share process-wide resources** such as memory and fil handles, but each thread has its own program counter, stack, and local variables.
* Multiple threads within the same program can be scheduled simultaenously on multiple CPUs.
* Threads are sometimes called *lightweight processes*, and modern OS usually treat them as the basic units of scheduling.
* **Benefits of threads**
    * Threads can improve the perforamance of complex applications.
    * Threads are useful in GUI applications for improving the responsiveness of the user interface, and in server applications for improving resource utilization and throughput. They also simplify the implementation of the JVM: the GC usually runs in one or more dedicated threads.
* Using threads is a good way to better use the reources on a multi-processor. On single-processor systems it can help as well, since the processor remains idle while it waits for a synchronous I/O operation to complete.
* A complicated, async workflow, can be decomposed into a number of simpler, sync workflows each running in a separated thread, interacting only with each other at specific synchronization points. This is exploited e.g. by servlets: when a servlet's *service* method is called, it can process the request synchronously as if it were a single-threaded program.
* Single-threaded server applications are force to use nonblocking I/O, which is far more complicated and error-prone than sync I/O. However, if each request has its own thread, then blocking does not affect the processing of other requests.
* OS support for larger numbers of threads has improved significantly, making the thread-per-client model practical (you only had several hundred before).
* Modern GUI frameworks, such as the AWT and Swing toolkist, replace the main event loop with an *event dispatch thread* (EDT). The point is that the threads execute very short-lived tasks.
* If the long-running task is executed in a separate thread, the event thread remains free to process UI events, making the UI more responsive.
* In the absence of sufficient synchronization, the ordering of operations in multiple threads is unpredictable and sometimes surpising.
* Annotations @NotThreadSafe, @ThreadSafe, @GuardedBy, @Immutable
* **Race conditions**: because threads share the same memory address space and run concurrently, they can access or modify variables that other threads might be using. That makes data sharing much easier than would other inter-thread communications mechanisms. But... with a significant risk.


## Fundamentals
### Thread Safety
TBD

## Structuring concurrent applications
TBD

## Liveness, Performance and Testing
TBD

## Advanced Topics
TBD