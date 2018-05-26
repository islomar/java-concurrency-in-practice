# Java concurrency in practice

Notes and examples from the book "Java concurrency in practice" by Brian Goetz, published in 2006.

* http://jcip.net/
* Errata: http://jcip.net/errata.html

## Preface
### Chapter 1: Introduction
* OS evolved to allow more than one program to run at once, running individual programs in *processes*: isolated, independently executing programs to which the OS allocates resources such as memory, file handles, and security credentials. If they needed to, **processes could communicate with one another through a variety of coarse-grained communication mechanisms: sockets, signal handlers, shared memory, semaphores, and files.**
* Finding the right balance of sequentiality and synchrony is often a characteristic of efficient people - and the same is true of programs.
* Threads allow multiple streams of program control flow to coexist within a process. **They share process-wide resources** such as memory and file handles, but each thread has its own program counter, stack, and local variables.
* Multiple threads within the same program can be scheduled simultaneously on multiple CPUs.
* Threads are sometimes called *lightweight processes*, and modern OS usually treat them as the basic units of scheduling.
* **Benefits of threads**
    * Threads can improve the performance of complex applications.
    * Threads are useful in GUI applications for improving the responsiveness of the user interface, and in server applications for improving resource utilization and throughput. They also simplify the implementation of the JVM: the GC usually runs in one or more dedicated threads.
* Using threads is a good way to better use the resources on a multi-processor. On single-processor systems it can help as well, since the processor remains idle while it waits for a synchronous I/O operation to complete.
* A complicated, async workflow, can be decomposed into a number of simpler, sync workflows each running in a separated thread, interacting only with each other at specific synchronization points. This is exploited e.g. by servlets: when a servlet's *service* method is called, it can process the request synchronously as if it were a single-threaded program.
* Single-threaded server applications are forced to use nonblocking I/O, which is far more complicated and error-prone than sync I/O. However, if each request has its own thread, then blocking does not affect the processing of other requests.
* OS support for larger numbers of threads has improved significantly, making the thread-per-client model practical (you only had several hundred before).
* Modern GUI frameworks, such as the AWT and Swing toolkit, replace the main event loop with an *event dispatch thread* (EDT). The point is that the threads execute very short-lived tasks.
* If the long-running task is executed in a separate thread, the event thread remains free to process UI events, making the UI more responsive.
* In the absence of sufficient synchronization, the ordering of operations in multiple threads is unpredictable and sometimes surprising.
* Annotations @NotThreadSafe, @ThreadSafe, @GuardedBy, @Immutable
* **Race conditions**: because threads share the same memory address space and run concurrently, they can access or modify variables that other threads might be using. That makes data sharing much easier than would other inter-thread communications mechanisms. But... with a significant risk.
* **safety**: nothing bad ever happens.
* **Liveness**: something good eventually happens.
* **Liveness failure**: it occurs when an activity gets into a state such that it is permanently unable to make forward progress.
    * E.g.: infinite loops, deadlock, starvation, livelock.
* **Context switches**: the scheduler suspends the active thread temporarily so another thread can run. The more threads, the more cost: saving and restoring execution context, loss of locality, CPU time spent scheduling threads instead of running them.
* Every Java application uses threads. When the JVM starts, it creates threads for JVM housekeeping tasks (garbage collection, finalization) and a main thread for running the *main* method.
    * The AWT and Swing UI frameworks create threads for managing user interface events. *Timer* creates threads for executing deferred tasks.
    * Servlets and RMI create pools of threads and invoke component methods in these threads.
    * Servlets must be prepared to be called simultaneously from multiple threads.
* Frameworks introduce concurrency into applications by calling application components from framework threads.
* Frameworks that create threads and call your components from those threads, leaving you the responsibility of making your components thread-safe: 
    * *Timer*, *Servlet*, *JSP*, *RMI*, *Swing/AWT*... but you need to be sure that **all** code paths accessing the application state is thread-safe.
    * RMI: you don't know in which thread of the remote JVM your object will be executed, so it should better be thread-safe.
* **Stateless objects are always thread-safe**
* **Race condition**: it occurs when the correctness of a computation depends on the relative timing or interleaving of multiple threads by the runtime.
    * The most common type of race condition is *check-then-act*.
    * A common idiom that uses check-then-act is *lazy initialization*.


## Fundamentals
### Thread Safety
* Writing thread-safe code is, at its core, about managing access to *state*, and in particular to *shared*, *mutable state*.
* We want to protect *data* from uncontrolled concurrent access.
* Primary mechanism for synchronization in Java: the *synchronized* keyword.
* Three ways to fix a program without proper synchronization:
    * Don't share the state variable across threads.
    * Make the state variable *immutable*
    * Use synchronization whenever accessing the state variable
 * The better encapsulated your program state, the easier it is to make your program thread-safe.
 * "First make it work, then make it fast": pursue optimization only if your performance measurements and requirements tell you that you must.
* *Correctness* means that a class conforms to its specification. A good spec defines *invariants* and *postconditions*.
* A class is thread-safe when it continues to behave correctly when accessed from multiple threads.
* `java.util.concurrent.atomic`package contains *atomic variable* classes.
* When practical, use existing thread-safe objects, like *AtomicLong* to manage your class's state.
* *AtomicReference*: thread-safe holder class for an object reference.
* **Thread safety** requires that invariants be preserved regardless of timing or interleaving of operations in multiple threads.
* An **invariant** is a condition that can be relied upon to be true during execution of a program, or during some portion of it.
* When you have several variables in an invariant, they are not independent: you must update all of them *in the same atomic operation*.
* To preserve state consistency, update related state variables in a single atomic operation.
* **Intrinsic locks** or **monitor locks**: `synchronized` block. E.g. `synchronized (lockObject) { // access or modify shared state guarded by lock }`
    * Any object can act as a lock.
    * They act as *mutexes* (or mutual exclusion locks), which means that at most one thread may own the lock.
    * Intrinsic locks are *reentrant*: locks are acquired on a per-thread rather than per-invocation basis (it means that a thread trying to get a lock already held, succeeds). Reentrancy is implemented by associating with each lock an acquisitions count and an owning thread.
    * Reentrancy saves us from deadlock sometimes, e.g. when calling a synchronized parent method from itself.
* If synchronization is used to coordinate access to a variable, it is neede *everywhere thtat variable is accessed*
* For each mutable state variable that may be accessed by more than one thread, all accesses to that variable must be performed with the same lock held. In this case, we say that the variable is guarded by that lock.
* Acquireing the lock associated with an object does **not** prevent other threads from accessing that object - the only thing that acquiring a lock prevents any other thread from doing is acquiring that same lock.
* Every shared, mutable variable should be guarded by exactly one lock.
* A common locking convention is to encapsulate all mutable state within an object and to protect it from concurrent access by synchronizing any code path that accesses mutable state using the object's intrinsic lock. 
* For every invariant that involves more than one variable, all the variables involved in that invariant must be guarded by the same lock 
* Synchronizing every method can lead to liveness or performance problems.
* Acquiring and releasing a lock has some overhead, so it is undesirable to break down *synchronized* blocks *too* far. 
* There is frequently a tension between simplicity and performance.
* Avoid holding locks during lengthy computations or operations at risk of not completing quickly such as network or console I/O.


### Sharing Objects
TBD


## Structuring concurrent applications
TBD

## Liveness, Performance and Testing
TBD

## Advanced Topics
TBD