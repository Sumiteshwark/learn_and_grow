**Last Updated:** November 2025  
**Author:** SUMITESHWAR KUMAR

---

# 🧵 Go Concurrency Terms Cheat Sheet

## Table of Contents

1. [Goroutine](#1-goroutine)
2. [Channel](#2-channel)
3. [Select](#3-select)
4. [WaitGroup](#4-waitgroup)
5. [Mutex](#5-mutex)
6. [RWMutex](#6-rwmutex)
7. [Atomic Operations](#7-atomic-operations)
8. [Semaphore](#8-semaphore)
9. [Context](#9-context)
10. [Pipeline](#10-pipeline)
11. [Fan-out / Fan-in](#11-fan-out-fan-in)
12. [Buffered Work Queue](#12-buffered-work-queue)
13. [Deadlock](#13-deadlock)
14. [Race Condition](#14-race-condition)
15. [Memory Visibility / Happens-Before](#15-memory-visibility-happens-before)
16. [Thread Pool / Worker Pool](#16-thread-pool-worker-pool)
17. [Owner & Ownership Transfer](#17-owner-ownership-transfer)
18. [Backpressure](#18-backpressure)
19. [Starvation](#19-starvation)
20. [Block vs Non-blocking](#20-block-vs-non-blocking)
21. [Cancellation](#21-cancellation)
22. [close(ch)](#22-closech)
23. [Fan-out concurrency limit](#23-fan-out-concurrency-limit)
24. [CPU-bound vs IO-bound](#24-cpu-bound-vs-io-bound)
25. [Scheduler (G-M-P Model)](#25-scheduler-g-m-p-model)
26. [Race-Free Patterns](#26-race-free-patterns)

---

## 🧵 Go Concurrency Theory - Comprehensive Explanations

## 1. Goroutine

Goroutines are the fundamental unit of concurrency in Go, representing lightweight threads managed by the Go runtime. Unlike traditional operating system threads, goroutines are multiplexed onto a smaller number of OS threads, making them extremely efficient.

**Theoretical Foundation:**
- Goroutines follow the concept of "green threads" or "user-space threads"
- They have their own call stack but share the address space with other goroutines
- The Go scheduler implements an M:N threading model where M goroutines are scheduled onto N OS threads
- Goroutines start with a minimal stack (2KB) that grows dynamically as needed
- They enable massive concurrency - it's common to have thousands or tens of thousands of goroutines running simultaneously

**Key Properties:**
- Cooperative scheduling: goroutines yield control at specific points (I/O operations, channel communications, runtime calls)
- Extremely low overhead compared to OS threads (initial stack ~2KB vs ~1MB for OS threads)
- Communication through channels rather than shared memory
- Non-preemptive: a goroutine must explicitly yield control

**Memory Model:**
- Each goroutine has its own stack and local variables
- Global variables and heap-allocated data are shared
- The happens-before relationship ensures memory visibility guarantees

**Code Example:**
```go
go func() {
    fmt.Println("Hello from goroutine!")
}()
time.Sleep(time.Millisecond) // Allow goroutine to run
```

---

## 2. Channel

Channels are typed conduits that allow goroutines to communicate and synchronize their execution. They provide a way for goroutines to send and receive values safely without explicit locking.

**Theoretical Foundation:**
- Based on CSP (Communicating Sequential Processes) by Tony Hoare
- Channels are first-class values that can be passed to functions
- They enforce a message-passing concurrency model over shared memory
- Channel operations are blocking by default, providing natural synchronization points

**Types of Channels:**
- Unbuffered channels: synchronous communication (send blocks until receive happens)
- Buffered channels: asynchronous communication up to buffer capacity
- Directional channels: send-only or receive-only for type safety

**Channel Axioms:**
- Send to a nil channel blocks forever
- Receive from a nil channel blocks forever
- Send to a closed channel panics
- Receive from a closed channel returns zero value immediately
- Closing a nil or closed channel panics

**Code Example:**
```go
ch := make(chan int)
go func() { ch <- 42 }()
value := <-ch // Receive value
fmt.Println(value) // 42
```

---

## 3. Select

The select statement provides a way to wait on multiple channel operations simultaneously, allowing goroutines to multiplex I/O operations and implement timeouts.

**Theoretical Foundation:**
- Select is inspired by the select system call in Unix-like systems
- It allows non-deterministic choice between multiple communication operations
- Select statements are non-blocking if any case is ready, otherwise block until one becomes ready
- The case chosen when multiple are ready is selected pseudo-randomly to avoid starvation

**Select Semantics:**
- If multiple cases are ready, one is chosen at random
- The default case executes immediately if no other case is ready
- Select with no cases (select{}) blocks forever
- Select can be used with timeouts, cancellations, and multiplexing

**Common Patterns:**
- Multiplexing multiple input sources
- Implementing timeouts with time.After()
- Non-blocking operations with default case
- Fan-in pattern for merging multiple channels

**Code Example:**
```go
select {
case msg := <-ch1:
    fmt.Println("Received from ch1:", msg)
case msg := <-ch2:
    fmt.Println("Received from ch2:", msg)
case <-time.After(time.Second):
    fmt.Println("Timeout")
}
```

---

## 4. WaitGroup

WaitGroup is a synchronization primitive that allows one goroutine to wait for a collection of goroutines to finish executing.

**Theoretical Foundation:**
- Implements a counting semaphore pattern
- Maintains an internal counter that's manipulated atomically
- Add(n) increments the counter, Done() decrements it, Wait() blocks until counter reaches zero
- Thread-safe and can be used across multiple goroutines

**Key Operations:**
- Add(delta): adds delta to the counter (can be negative)
- Done(): decrements counter by 1 (equivalent to Add(-1))
- Wait(): blocks until counter becomes zero

**Usage Patterns:**
- Waiting for a fixed number of goroutines to complete
- Coordinating startup/shutdown sequences
- Ensuring all workers finish before proceeding

**Code Example:**
```go
var wg sync.WaitGroup
wg.Add(2)
go func() { defer wg.Done(); work() }()
go func() { defer wg.Done(); work() }()
wg.Wait() // Wait for both goroutines
```

---

## 5. Mutex

Mutex (mutual exclusion) provides exclusive access to shared resources, preventing race conditions by ensuring only one goroutine can access a critical section at a time.

**Theoretical Foundation:**
- Implements the critical section problem solution
- Binary semaphore that allows only one holder at a time
- Lock() acquires the mutex (blocks if already held), Unlock() releases it
- Mutexes in Go are not reentrant - a goroutine cannot lock the same mutex twice

**Locking Semantics:**
- Lock() will block until the mutex is available
- Unlock() must be called by the same goroutine that called Lock()
- Attempting to unlock an unlocked mutex causes a panic
- Mutexes should be used sparingly as they serialize execution

**Best Practices:**
- Keep critical sections as small as possible
- Avoid holding locks during I/O operations
- Use defer to ensure unlocks happen
- Consider RWMutex for read-mostly workloads

**Code Example:**
```go
var mu sync.Mutex
counter := 0

mu.Lock()
counter++
mu.Unlock()
```

---

## 6. RWMutex

RWMutex (Read-Write Mutex) allows multiple readers or a single writer to access a shared resource concurrently, optimizing for read-heavy workloads.

**Theoretical Foundation:**
- Extends the mutex concept with reader/writer semantics
- Multiple readers can hold the lock simultaneously
- Only one writer can hold the lock at a time (exclusive access)
- Writers have priority over readers to prevent writer starvation

**Lock Types:**
- RLock(): acquires read lock (multiple allowed)
- RUnlock(): releases read lock
- Lock(): acquires write lock (exclusive)
- Unlock(): releases write lock

**Key Properties:**
- Read locks are shared and non-blocking for other readers
- Write locks are exclusive and block all other operations
- Attempting to acquire write lock while holding read lock causes deadlock
- More efficient than regular mutexes for read-heavy workloads

**Code Example:**
```go
var rwmu sync.RWMutex

// Multiple readers
rwmu.RLock()
value := readData()
rwmu.RUnlock()

// Single writer
rwmu.Lock()
writeData(newValue)
rwmu.Unlock()
```

---

## 7. Atomic Operations

Atomic operations provide lock-free, thread-safe access to shared variables at the hardware level, ensuring operations complete without interruption.

**Theoretical Foundation:**
- Built on CPU atomic instructions (CAS, LL/SC, etc.)
- Operations appear to execute instantaneously from other threads' perspective
- No context switches can occur during atomic operations
- Provide memory barriers ensuring proper ordering

**Common Operations:**
- Load: atomic read of a value
- Store: atomic write of a value
- Add: atomic increment/decrement
- CompareAndSwap: conditional update if value matches expected
- Swap: atomic exchange of values

**Memory Ordering:**
- Relaxed: no ordering guarantees
- Acquire: prevents reordering after the operation
- Release: prevents reordering before the operation
- Acquire-Release: both acquire and release semantics

**Code Example:**
```go
var counter int64
atomic.AddInt64(&counter, 1)           // Increment atomically
value := atomic.LoadInt64(&counter)    // Read atomically
atomic.StoreInt64(&counter, 42)        // Write atomically
```

---

## 8. Semaphore

Semaphores are synchronization primitives that control access to a shared resource by maintaining a counter that limits concurrent access.

**Theoretical Foundation:**
- Invented by Edsger Dijkstra in 1965
- Generalization of mutexes allowing multiple holders
- Binary semaphores act like mutexes (0 or 1)
- Counting semaphores allow up to N concurrent accesses

**Operations:**
- Acquire/P(): decrements counter (blocks if counter would go negative)
- Release/V(): increments counter (may wake waiting goroutines)
- TryAcquire(): non-blocking acquire attempt

**Types:**
- Binary semaphores: limit to 1 concurrent access
- Counting semaphores: limit to N concurrent accesses
- Weighted semaphores: allow different "weights" for different operations

**Code Example:**
```go
sem := make(chan struct{}, 3) // Allow 3 concurrent operations

go func() {
    sem <- struct{}{}        // Acquire
    defer func() { <-sem }() // Release
    doWork()
}()
```

---

## 9. Context

Context provides a way to carry request-scoped values, cancellation signals, and deadlines across API boundaries and goroutine trees.

**Theoretical Foundation:**
- Implements request-scoped data propagation
- Enables graceful cancellation of long-running operations
- Provides deadline management for timeout handling
- Thread-safe and can be passed between goroutines

**Key Components:**
- Value storage: key-value pairs for request data
- Cancellation: Done() channel closes when cancelled
- Deadline: timeout management with ContextWithTimeout()
- Cause: reason for cancellation (Go 1.21+)

**Context Types:**
- Background: empty context, never cancelled
- TODO: placeholder for contexts not yet determined
- WithCancel: adds cancellation capability
- WithDeadline/WithTimeout: adds time-based cancellation
- WithValue: adds key-value data

**Code Example:**
```go
ctx, cancel := context.WithTimeout(context.Background(), time.Second)
defer cancel()

select {
case <-ctx.Done():
    fmt.Println("Cancelled:", ctx.Err())
case result := <-doWork():
    fmt.Println("Result:", result)
}
```

---

## 10. Pipeline

Pipelines represent a series of stages connected by channels, where each stage performs work on data flowing through the pipeline.

**Theoretical Foundation:**
- Based on Unix pipeline philosophy (command | command)
- Each stage is a goroutine processing data from input channel to output channel
- Enables parallel processing of data streams
- Natural fit for stream processing and data transformation

**Pipeline Patterns:**
- Linear pipelines: A → B → C
- Fan-out: one stage distributes to multiple workers
- Fan-in: multiple workers merge into one stage
- Diamond pattern: branching and merging

**Benefits:**
- Clean separation of concerns
- Natural parallelism
- Backpressure through channel buffering
- Easy testing of individual stages

**Code Example:**
```go
func stage1(out chan<- int) {
    defer close(out)
    for i := 0; i < 5; i++ {
        out <- i
    }
}

func stage2(in <-chan int, out chan<- int) {
    defer close(out)
    for v := range in {
        out <- v * 2
    }
}

ch1 := make(chan int)
ch2 := make(chan int)
go stage1(ch1)
go stage2(ch1, ch2)
for result := range ch2 {
    fmt.Println(result)
}
```

---

## 11. Fan-out / Fan-in

Fan-out distributes work from one channel to multiple worker goroutines, while fan-in merges results from multiple channels into one.

**Theoretical Foundation:**
- Fan-out: one-to-many distribution pattern
- Fan-in: many-to-one aggregation pattern
- Enables parallel processing of independent work items
- Balances load across available workers

**Fan-out Patterns:**
- Round-robin distribution
- Load-based distribution
- Hash-based partitioning
- Work-stealing

**Fan-in Patterns:**
- Ordered merging (preserving input order)
- Unordered merging (best performance)
- Priority-based merging
- Result aggregation

**Code Example:**
```go
func fanOut(in <-chan int, out chan<- int) {
    for v := range in {
        out <- v * v
    }
}

func fanIn(ch1, ch2 <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for {
            select {
            case v := <-ch1:
                out <- v
            case v := <-ch2:
                out <- v
            }
        }
    }()
    return out
}
```

---

## 12. Buffered Work Queue

Buffered channels used as work queues allow producers to submit work without blocking, decoupling production and consumption rates.

**Theoretical Foundation:**
- Implements producer-consumer pattern
- Buffer absorbs bursts in production/consumption
- Prevents fast producers from overwhelming slow consumers
- Enables asynchronous processing

**Queue Semantics:**
- Size determines burst tolerance
- Full queue blocks producers (backpressure)
- Empty queue blocks consumers
- Zero-size buffer = synchronous communication

**Applications:**
- Job queues
- Task schedulers
- Event processing systems
- Load balancing

**Code Example:**
```go
workQueue := make(chan Work, 100) // Buffered queue

// Producer
go func() {
    for job := range jobs {
        workQueue <- job
    }
    close(workQueue)
}()

// Consumer
for job := range workQueue {
    process(job)
}
```

---

## 13. Deadlock

Deadlock occurs when goroutines are blocked forever waiting for each other to release resources, creating a circular dependency.

**Theoretical Foundation:**
- Defined by the "four necessary conditions" (Coffman et al.)
- Circular wait: goroutines wait for resources held by others in a cycle
- Mutual exclusion: resources can't be shared
- Hold and wait: goroutines hold resources while waiting for others
- No preemption: resources can't be forcibly taken

**Types:**
- Resource deadlock: waiting for locks/channels
- Communication deadlock: waiting for messages that never come
- Livelock: goroutines active but making no progress

**Prevention:**
- Acquire locks in consistent order
- Use timeouts and context cancellation
- Avoid holding multiple locks simultaneously
- Design for failure and recovery

**Code Example (Deadlock):**
```go
// This would deadlock - don't run!
/*
var mu1, mu2 sync.Mutex

go func() {
    mu1.Lock()
    time.Sleep(time.Millisecond) // Context switch
    mu2.Lock() // Deadlock!
}()

go func() {
    mu2.Lock()
    time.Sleep(time.Millisecond) // Context switch  
    mu1.Lock() // Deadlock!
}()
*/
```

---

## 14. Race Condition

Race conditions occur when the behavior of a program depends on the relative timing of operations in concurrent goroutines.

**Theoretical Foundation:**
- Non-deterministic behavior due to timing dependencies
- Multiple goroutines accessing shared data simultaneously
- Result depends on interleaving of operations
- Often subtle and hard to reproduce

**Types:**
- Read-modify-write races
- Check-then-act races
- Data races (undefined behavior in Go)
- Atomicity violations

**Detection:**
- Go race detector (`go run -race`)
- Static analysis tools
- Careful code review
- Stress testing

**Code Example (Race Condition):**
```go
counter := 0

go func() { counter++ }() // Race condition!
go func() { counter++ }() // Multiple goroutines access counter

time.Sleep(time.Millisecond)
// counter might be 1 or 2 - unpredictable!
```

---

## 15. Memory Visibility / Happens-Before

The Go memory model defines when writes by one goroutine become visible to reads by another, ensuring predictable concurrent behavior.

**Theoretical Foundation:**
- Defines the happens-before relationship between operations
- Ensures memory consistency across goroutines
- Compiler and hardware reordering is constrained
- Provides guarantees for synchronization primitives

**Happens-Before Rules:**
- Within a goroutine: program order
- Channel communication: send happens before receive
- Lock/unlock: unlock happens before subsequent lock
- Once: initialization happens before all reads

**Memory Barriers:**
- Synchronization operations act as memory barriers
- Prevent reordering of operations across the barrier
- Ensure visibility of writes before the barrier

**Code Example:**
```go
var ready bool
var data int

go func() {
    data = 42    // Write
    ready = true // Write (happens after data write)
}()

for !ready {    // Read ready
    runtime.Gosched()
}
fmt.Println(data) // Guaranteed to see 42
```

---

## 16. Thread Pool / Worker Pool

Worker pools limit the number of concurrent goroutines performing similar tasks, controlling resource usage and preventing overload.

**Theoretical Foundation:**
- Limits concurrency to prevent resource exhaustion
- Reuses goroutines for multiple tasks
- Provides natural backpressure
- Balances throughput vs resource usage

**Components:**
- Task queue: holds pending work
- Worker goroutines: fixed number processing tasks
- Dispatcher: assigns tasks to available workers
- Result collection: gathers completed work

**Benefits:**
- Controlled resource usage
- Predictable performance
- Natural load limiting
- Efficient goroutine reuse

**Code Example:**
```go
jobs := make(chan int, 100)
results := make(chan int, 100)

// Start workers
for w := 1; w <= 3; w++ {
    go worker(w, jobs, results)
}

// Send jobs
for j := 1; j <= 9; j++ {
    jobs <- j
}
close(jobs)

// Collect results
for r := 1; r <= 9; r++ {
    <-results
}
```

---

## 17. Owner & Ownership Transfer

Ownership patterns ensure that only one goroutine is responsible for a resource at any time, preventing concurrent access issues.

**Theoretical Foundation:**
- Single responsibility principle for concurrent resources
- Clear ownership transfer protocols
- Prevents data races through ownership discipline
- Makes concurrent code more predictable

**Transfer Patterns:**
- Channel-based transfer: send/receive ownership
- Function return: callee transfers to caller
- Mutex-based: lock/unlock transfers responsibility
- Context-based: scoped ownership

**Benefits:**
- Eliminates data races
- Clear responsibility boundaries
- Easier reasoning about concurrent code
- Automatic cleanup through ownership

**Code Example:**
```go
func producer() <-chan int {
    ch := make(chan int)
    go func() {
        defer close(ch) // Producer closes channel
        ch <- 42
    }()
    return ch // Transfer ownership to consumer
}

func consumer(ch <-chan int) {
    value := <-ch // Consumer owns the channel
    fmt.Println(value)
}
```

---

## 18. Backpressure

Backpressure prevents fast producers from overwhelming slow consumers by applying resistance when the system can't keep up.

**Theoretical Foundation:**
- Natural consequence of bounded buffers
- Prevents resource exhaustion
- Enables graceful degradation under load
- Maintains system stability

**Mechanisms:**
- Blocking channels (natural backpressure)
- Buffered channels with limits
- Semaphore-based rate limiting
- Explicit flow control protocols

**Applications:**
- Network services
- Queue-based systems
- Stream processing pipelines
- Resource-constrained environments

**Code Example:**
```go
bufferedCh := make(chan int, 10) // Limited buffer

go func() {
    for i := 0; i < 20; i++ {
        bufferedCh <- i // Blocks when buffer full (backpressure)
    }
    close(bufferedCh)
}()

for value := range bufferedCh {
    fmt.Println(value)
    time.Sleep(time.Millisecond) // Slow consumer
}
```

---

## 19. Starvation

Starvation occurs when a goroutine is unable to make progress because other goroutines monopolize shared resources.

**Theoretical Foundation:**
- Priority inversion or unfair scheduling
- Goroutines blocked indefinitely despite being runnable
- Can cause deadlock-like symptoms
- Often caused by unfair resource allocation

**Causes:**
- Unfair mutex contention
- Channel selection bias
- Priority scheduling issues
- Resource monopolization

**Prevention:**
- Fair scheduling algorithms
- Time-based fairness
- Resource quotas and limits
- Proper priority management

**Code Example (Starvation):**
```go
ch := make(chan int)

go func() {
    for {
        select {
        case ch <- 1: // Always ready
        default:      // Never taken
        }
    }
}()

// This goroutine starves
go func() {
    <-ch // Never gets CPU time!
}()
```

---

## 20. Block vs Non-blocking

Blocking operations suspend goroutine execution until a condition is met, while non-blocking operations return immediately.

**Theoretical Foundation:**
- Blocking: synchronous, suspends execution
- Non-blocking: asynchronous, returns immediately
- Trade-off between simplicity and efficiency
- Choice depends on concurrency requirements

**Blocking Operations:**
- Channel send/receive (unbuffered)
- Mutex.Lock()
- select without default
- I/O operations

**Non-blocking Operations:**
- Buffered channel operations (when buffer available)
- select with default
- TryLock() methods
- Asynchronous I/O

**Code Example:**
```go
// Blocking
unbufferedCh := make(chan int)
unbufferedCh <- 42 // Blocks until received

// Non-blocking
bufferedCh := make(chan int, 1)
select {
case bufferedCh <- 42: // Non-blocking send
    fmt.Println("Sent")
default:
    fmt.Println("Would block")
}
```

---

## 21. Cancellation

Cancellation allows goroutines to be gracefully stopped, releasing resources and preventing unnecessary work.

**Theoretical Foundation:**
- Cooperative cancellation pattern
- Uses context.Done() channel
- Enables clean shutdown of goroutine trees
- Prevents resource leaks from abandoned goroutines

**Mechanisms:**
- Context.WithCancel()
- Context.WithTimeout()
- Context.WithDeadline()
- Manual cancellation channels

**Best Practices:**
- Check cancellation regularly
- Clean up resources on cancellation
- Propagate cancellation to child goroutines
- Handle cancellation gracefully

**Code Example:**
```go
ctx, cancel := context.WithCancel(context.Background())

go func() {
    select {
    case <-ctx.Done():
        fmt.Println("Cancelled")
        return
    case <-time.After(time.Second):
        fmt.Println("Work done")
    }
}()

cancel() // Cancel the operation
```

---

## 22. close(ch)

Closing a channel signals that no more values will be sent, allowing receivers to detect completion and clean up.

**Theoretical Foundation:**
- Signals end of data stream
- Allows receivers to terminate cleanly
- Prevents deadlock in fan-in patterns
- Enables deterministic shutdown

**Semantics:**
- Send to closed channel panics
- Receive from closed channel returns zero value + false
- Multiple closes panic
- Close on nil channel panics

**Patterns:**
- Producer closes channel after sending all data
- Range over channel automatically detects close
- Select with close detection
- Graceful shutdown signaling

**Code Example:**
```go
ch := make(chan int)

go func() {
    for i := 0; i < 3; i++ {
        ch <- i
    }
    close(ch) // Signal completion
}()

for value := range ch { // Automatically stops when closed
    fmt.Println(value)
}
```

---

## 23. Fan-out Concurrency Limit

Fan-out with concurrency limits controls the number of goroutines spawned for parallel processing, preventing resource exhaustion.

**Theoretical Foundation:**
- Balances parallelism with resource constraints
- Prevents goroutine explosion
- Maintains system stability under varying loads
- Enables adaptive concurrency

**Mechanisms:**
- Semaphore-based limiting
- Worker pool patterns
- Channel-based throttling
- Dynamic limit adjustment

**Benefits:**
- Predictable resource usage
- Improved stability
- Better performance under load
- Configurable concurrency levels

**Code Example:**
```go
semaphore := make(chan struct{}, 3) // Limit to 3 concurrent
work := make(chan int, 10)

for i := 0; i < 10; i++ {
    go func(id int) {
        semaphore <- struct{}{}        // Acquire
        defer func() { <-semaphore }() // Release
        
        work <- process(id)
    }(i)
}
```

---

## 24. CPU-bound vs IO-bound

CPU-bound tasks are limited by processor speed, while IO-bound tasks spend most time waiting for input/output operations.

**Theoretical Foundation:**
- CPU-bound: computation-intensive, limited by CPU cores
- IO-bound: waiting for external resources (disk, network, etc.)
- Different optimization strategies for each type
- Affects goroutine scheduling and resource allocation

**CPU-bound Characteristics:**
- High CPU utilization
- Minimal I/O waits
- Scales with CPU cores
- Benefits from fewer, longer-running goroutines

**IO-bound Characteristics:**
- Low CPU utilization
- Frequent blocking on I/O
- Scales well with many goroutines
- Benefits from multiplexing (epoll, kqueue)

**Code Example:**
```go
// CPU-bound (few goroutines)
func cpuIntensive() {
    for i := 0; i < 1000000; i++ {
        _ = i * i // Computation
    }
}
runtime.GOMAXPROCS(1) // Single CPU

// IO-bound (many goroutines)  
func ioIntensive() {
    resp, _ := http.Get("http://example.com")
    defer resp.Body.Close()
}
```

---

## 25. Scheduler (G-M-P Model)

Go's scheduler implements an M:N threading model where G goroutines are multiplexed onto M OS threads, coordinated by P processors.

**Theoretical Foundation:**
- G: Goroutines (user-space threads)
- M: Machine threads (OS threads)
- P: Processors (scheduling contexts)
- Work-stealing scheduler for load balancing

**Components:**
- G queue: runnable goroutines per P
- Global G queue: overflow goroutines
- M pool: available OS threads
- P pool: scheduling contexts (GOMAXPROCS)

**Scheduling:**
- Cooperative: goroutines yield at synchronization points
- Work-stealing: idle P steals work from busy P
- Preemption: prevents goroutines from hogging CPU
- Network poller: handles I/O readiness

**Code Example:**
```go
// GOMAXPROCS sets number of P's
runtime.GOMAXPROCS(4) // 4 processors

// Goroutines (G's) scheduled on processors (P's)
// OS threads (M's) execute the work
for i := 0; i < 100; i++ {
    go func(id int) {
        fmt.Println("Goroutine", id)
    }(i)
}
```

---

## 26. Race-Free Patterns

Race-free patterns provide structured approaches to concurrent programming that eliminate data races by design.

**Theoretical Foundation:**
- Design patterns that prevent concurrent access issues
- Use of CSP principles and message passing
- Ownership and immutability patterns
- Structured concurrency approaches

**Key Patterns:**
- Actor model: encapsulated state with message passing
- Pipeline pattern: streaming data through stages
- Worker pool: controlled concurrency
- Immutable data structures: share without synchronization

**Benefits:**
- Eliminates data races by construction
- Easier to reason about
- More maintainable concurrent code
- Leverages Go's strengths (channels, goroutines)

**Code Example:**
```go
// Race-free: communicate via channels
func counter(in <-chan bool, out chan<- int) {
    count := 0
    for range in {
        count++
    }
    out <- count // Send result via channel
}

done := make(chan bool, 10)
result := make(chan int)

go counter(done, result)

// Send signals
for i := 0; i < 10; i++ {
    done <- true
}
close(done)

fmt.Println("Count:", <-result)
```