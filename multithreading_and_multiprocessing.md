# Multithreading and Multiprocessing

# Introduction

In modern software development, performance and efficiency are essential.

Imagine you're using a video streaming platform like **Netflix**. When you hit play on a video, two important tasks happen:

- **Loading the video data**
- **Buffering the content**

If each task takes 5 seconds, you'll be waiting for a total of 10 seconds before you can watch your video.

But what if these tasks could happen at the same time?

Instead of waiting 10 seconds, you could reduce the total time to just 5 seconds, drastically improving your user experience.

---

## Everyday Life Analogy

Think of a restaurant kitchen where multiple chefs are working on different tasks:

- One chopping vegetables
- Another cooking
- Another plating the food

Instead of waiting for one chef to finish their task before the next can begin, all tasks happen in parallel.

This parallel work speeds up the process and leads to better overall efficiency.

---

## Why is This Important?

This principle of performing **multiple tasks at the same time**, without unnecessary delays, is crucial in today’s applications.

Whether it’s:

- A web server handling hundreds of requests
- A mobile app providing real-time updates

Understanding how to execute tasks efficiently is the key to building fast, responsive systems.

But how do we make this possible in software development?

That’s where concepts like:

- **Multithreading**
- **Concurrency**

come in — and learning how to leverage them is essential for any developer.

---

# Program, Process and Thread

In software development, we often come across the terms:

- **Program**
- **Process**
- **Thread**

While they may seem similar at first glance, they represent different concepts that are crucial to understanding how computers execute tasks.

To make it easier to grasp, let’s first start with a real-world analogy.

---

## Real-World Analogy

Imagine you’re running a bakery.

Here’s how we can relate the terms **program**, **process**, and **thread**:

### Program

Think of a recipe book.

A recipe book is a collection of instructions, but it doesn’t do anything on its own. It's just a set of written plans for making different types of cakes, bread, etc.

The recipe book is like a **program**: a static set of instructions to be followed.

### Process

Now, imagine you decide to bake a cake.

You pick a recipe from the book and follow the instructions.

The baking process is the **process**: it’s the execution of the recipe (the program) in action.

A process is a running instance of a program, actively working in memory.

### Thread

Inside your bakery, there may be multiple bakers working at the same time.

- One might be mixing ingredients
- Another might be preheating the oven
- Another might be icing the cake

Each baker represents a **thread**: a smaller task within the overall process.

Multiple threads can run concurrently, each handling part of the job.

---

## Program

A program is a collection of instructions written in a programming language that is intended to perform a specific task or solve a particular problem.

It’s like the recipe book mentioned earlier — static, not doing anything until it is run.

### Example

If you download **chrome.exe** from the internet, **chrome.exe** is a **program**.

It’s just a file sitting on your computer.

It contains instructions on how to launch and interact with Google Chrome, but it’s not doing anything until you run it.

---

## Process

A **process** is an instance of a program that is being executed.

When you launch a program (like opening **chrome.exe**), it gets loaded into memory and starts running.

This running instance is called a **process**.

A process includes:

- The program code
- Current activity
- Memory
- CPU usage
- Input/output resources

### Example

Continuing with the **chrome.exe** example:

Once you double-click on the Chrome icon to launch the browser, a **process** is created.

The program code in **chrome.exe** is now running in memory as a **process**, using system resources like memory and CPU time.

### Keypoints

- Each process has its own address space.
- Runs independently from other processes.
- Can execute without interfering unless allowed.
- Managed by the operating system.

---

## Thread

A **thread** is the **smallest unit of execution within a process**.

A process can contain multiple threads, which share the same resources but run independently.

Each thread can perform a separate task within the same process.

Threads allow for parallelism, where multiple tasks are executed simultaneously.

### Example

Within the **chrome.exe** process, there might be several threads running concurrently.

For instance:

- One thread renders the UI
- Another manages network requests
- Another handles user inputs

These threads all operate within the same process but perform different tasks simultaneously.

### Keypoints

- Threads are referred to as "lightweight" processes.
- They share resources like memory and CPU time.
- Threads within the same process share memory.

---

## Why Understanding These Concepts Matters

### Program

Understanding the program is essential for writing efficient code that can be turned into a process.

### Process

Processes allow us to execute programs, but they come with limitations like memory and resource allocation.

### Thread

Threads are at the heart of performance optimization.

By breaking a process into multiple threads, we can perform tasks concurrently, speeding up execution and improving user experience.

---

# Cores in CPU

A **core** in a CPU (Central Processing Unit) is a physical processing unit capable of executing instructions.

Modern CPUs often have **multiple cores**, allowing them to handle several tasks simultaneously.

Each core can independently execute a thread, meaning more cores lead to the ability to run more threads concurrently.

---

## Real-life Analogy

CPU cores are like **workers in an office**.

- Each worker can independently complete tasks.
- More workers mean more tasks can be handled simultaneously.
- A single-core CPU is like one worker doing everything alone.
- A multi-core CPU is like several workers handling tasks together.

---

## Significance of Understanding Cores in CPU

### Performance

More cores mean more tasks can be processed simultaneously.

### Parallelism

Cores allow parallel execution of threads.

### Energy Efficiency

Modern CPUs divide work efficiently to consume less power.

### Scaling Applications

Understanding CPU cores helps optimize multi-threaded software.

---

## Hyperthreading

**Hyperthreading** is a technology developed by Intel that allows a single physical core to act as two logical cores.

It enables **one core to run two threads** simultaneously.

### Real-life Analogy

If a worker could perform two tasks at once, they would complete more work without needing additional workers.

---

### Intelligent Time Slicing

- Each logical core takes turns executing tasks.
- Time slicing divides execution time between multiple threads.
- If one thread is waiting, another can continue execution.

---

### Resource Sharing

Both threads running on a single physical core share:

- Cache
- Execution units
- Memory bandwidth

---

### Benefits of Hyperthreading

- Better resource utilization
- Enhanced multitasking

---

# Context Switching

**Context Switching** is the process of storing and restoring the state of a thread or process so that it can be resumed later.

This allows the CPU to switch between different tasks or threads.

---

## Real-life Analogy

Think of context switching like a chef in a kitchen working on multiple dishes.

- The chef pauses one task
- Saves its current state
- Works on another task
- Returns later without losing progress

---

## How Does Context Switching Happen?

### Interrupt

A running thread is interrupted.

### Save State

The current thread’s state is saved.

### Load State

The next thread’s state is loaded.

### Switch Execution

The CPU starts executing the new thread.

---

## Importance of Context Switching

### Multitasking

Allows multiple tasks to run concurrently on a single core.

### Resource Management

Prevents one task from hogging the CPU.

### Efficiency

Keeps the CPU busy.

---

## Thread Scheduler

The thread scheduler:

- Manages context switching
- Decides which thread runs next
- Uses scheduling algorithms

Examples include:

- Round-robin scheduling
- Priority-based scheduling

---

### Performance Considerations

#### Task Scheduler Overhead

Saving and loading thread states takes time.

#### Decreased Performance

Too many threads increase context switching overhead.

---

# Multithreading

**Multithreading** is a programming technique that allows a CPU to execute multiple threads concurrently.

Instead of executing one task after another, multithreading allows the CPU to switch between tasks quickly.

Threads share the same memory space, allowing communication and coordination.

---

## Significance of Multithreading

### Better Performance

Tasks run concurrently.

### Non-blocking Nature

Threads can continue while others wait for I/O.

### Resource Sharing

Threads share memory and data efficiently.

### Scalability in Backend Services

Helps servers handle multiple requests simultaneously.

---

## Additional Benefits of Multithreading

### Responsiveness in UI Applications

Background threads keep the UI responsive.

### Efficient CPU Utilization

Multi-core processors can execute multiple threads efficiently.

### Real-Time Processing

Useful in gaming and streaming applications.

---

# Concurrency vs. Parallelism

Learners often confuse **Concurrency** and **Parallelism**.

However, they represent different concepts.

| Concurrency | Parallelism |
|---|---|
| Managing multiple tasks over time | Simultaneous execution of tasks |
| Can run on a single core | Requires multiple cores/processors |
| Uses context switching | Runs tasks at the same time |
| Focuses on managing tasks | Focuses on reducing execution time |

---

# Process vs. Thread

Processes and threads are different concepts.

| Process | Thread |
|---|---|
| Independent program in execution | Lightweight unit within a process |
| Has isolated memory | Shares memory with other threads |
| Communication is complex | Communication is easier |
| Heavyweight | Lightweight |
| Crashes usually isolated | One thread crash may affect others |

### Examples

- **Process:** PostgreSQL database instance
- **Thread:** Chrome browser tabs

---

# Shared Memory vs Isolated Memory

Understanding memory management is essential.

| Shared Memory | Isolated Memory |
|---|---|
| Accessible by multiple threads/processes | Dedicated to one process |
| Enables fast communication | More secure and isolated |
| Risk of race conditions | Safer but slower communication |
| Used in multi-threaded applications | Used in isolated processes |

### Examples

- Shared Memory: Video processing application
- Isolated Memory: Word processor and browser

---

# When to Use Thread and Process

Choosing between a thread and process depends on the application.

---

## When to Use a Thread

- Tasks need to share data
- Low overhead is important
- Tasks are tightly coupled
- High performance is required
- Responsiveness is important

---

## When to Use a Process

- Isolation is required
- One crash should not affect others
- Security boundaries are needed
- Different technology stacks are involved
- Resource limits are necessary

---

# Fault Tolerance

**Fault tolerance** refers to a system’s ability to continue functioning even when failures occur.

---

## Real-Life Analogy

An airplane has multiple engines and backup systems.

If one fails, others continue working.

---

## Keypoints

### Redundancy

Backup systems take over during failures.

### Error Detection and Correction

Systems detect and correct errors automatically.

### Graceful Degradation

Performance may reduce, but the system continues functioning.

### Automatic Recovery

Failed components restart or workloads are redirected automatically.

---

# Isolation

**Isolation** refers to separating tasks or environments so they do not interfere with each other.

---

## Real-life Analogy

Separate hotel rooms isolate guests from one another.

---

## Keypoints

### Memory Separation

Processes operate in distinct memory spaces.

### Failure Containment

One failure does not affect others.

### Security Boundaries

Prevents unauthorized access.

### Predictable Behavior

Ensures stable and predictable execution.

---


# Multithreading vs Multiprocessing in Python

In Python, the primary difference is that **multiprocessing bypasses the Global Interpreter Lock (GIL)** to achieve true parallel execution on multiple CPU cores, while **multithreading is limited to concurrent execution within a single core** due to that same lock.

---

# Core Comparison

| Feature | Multithreading (`threading`) | Multiprocessing (`multiprocessing`) |
|---|---|---|
| Execution | Concurrent (appears simultaneous) | Parallel (actually simultaneous) |
| Memory | Shared among all threads | Separate for each process |
| GIL Impact | Limited by the GIL | Bypasses the GIL |
| Overhead | Lightweight; fast to start | Heavyweight; slower to start |
| Best For | I/O-bound tasks (e.g., networking) | CPU-bound tasks (e.g., data crunching) |

---

# When to Use Which

## Use Multithreading when:

Your program spends most of its time waiting for external events. Because threads share the same memory space, they are efficient for tasks like:

- Downloading multiple files or making API requests
- Reading/writing to a database or disk
- Building responsive GUI applications

---

## Use Multiprocessing when:

Your program needs to perform heavy calculations. Since each process has its own Python interpreter and memory, it can fully utilize multiple CPU cores for:

- Heavy mathematical computations or data processing
- Image or video processing
- Any task where the CPU is the bottleneck

---

# Key Technical Concepts

## Global Interpreter Lock (GIL)

A mutex that protects access to Python objects, preventing multiple threads from executing Python bytecodes at once.

---

## IPC (Inter-Process Communication)

Because processes do not share memory, they must use specialized tools like Queues or Pipes to exchange data.

---

## Future Trends

Starting with Python 3.13/3.14, an experimental "free-threaded" build is being introduced to eventually remove the GIL, which may allow multithreading to achieve true parallelism in the future.

---

# References

## Articles

### Multithreading VS Multiprocessing in Python | by Amine Baatout

**Published:** 5 Dec 2018

> Conclusion:
>
> - There can only be one thread running at any given time in a Python process.
> - Multiprocessing is parallelism.

---

## Discussions

### Threading or multiprocessing - Python Discussions

**Published:** 19 May 2021

> As long as you do not need IPC (inter-process communication), multiprocessing is as easy as multithreading.

---

## Stack Overflow

### Multiprocessing vs Threading Python [duplicate]

**Published:** 15 Jun 2010

> The `threading` module uses threads, while the `multiprocessing` module uses processes.
>
> Threads run in the same memory space, whereas processes have separate memory spaces.

---

# Note

AI can make mistakes, so double-check responses.


# How Multiprocessing Achieves True Parallelism in Python

## The Problem with Threads in Python

Python’s standard implementation (**CPython**) uses something called the **Global Interpreter Lock (GIL)**.

The GIL allows:

- Only **one thread** to execute Python bytecode at a time
- Even if multiple threads exist, they take turns using the CPU

So in multithreading:

```python
# Multiple threads
Thread A ---> waits
Thread B ---> runs
Thread C ---> waits
```


# How Multiprocessing Achieves True Parallelism in Python

## The Problem with Threads in Python

Python’s standard implementation (**CPython**) uses something called the **Global Interpreter Lock (GIL)**.

The GIL allows:

- Only **one thread** to execute Python bytecode at a time
- Even if multiple threads exist, they take turns using the CPU

So in multithreading:

```python
# Multiple threads
Thread A ---> waits
Thread B ---> runs
Thread C ---> waits
```

Threads appear concurrent, but they are **not truly running Python code simultaneously** on multiple CPU cores.

---

# How Multiprocessing Solves This

`multiprocessing` creates **separate processes**, not threads.

Each process has:

- Its own Python interpreter
- Its own memory space
- Its own GIL

Because each process has its **own independent GIL**, the operating system can schedule them on different CPU cores at the same time.

---

# Example

Suppose your CPU has 4 cores.

Using multiprocessing:

```python
from multiprocessing import Process

def task():
    while True:
        pass

for _ in range(4):
    p = Process(target=task)
    p.start()
```

The OS may run:

| Process | CPU Core |
|---|---|
| Process 1 | Core 1 |
| Process 2 | Core 2 |
| Process 3 | Core 3 |
| Process 4 | Core 4 |

Now all four processes execute simultaneously.

This is **true parallelism**.

---

# Why It Works

The operating system treats each process independently.

Unlike threads:

- Processes do not share the same interpreter
- Processes do not compete for a single GIL
- Each process can fully utilize a CPU core

So CPU-intensive work becomes much faster.

---

# Visual Comparison

## Multithreading

```text
One Process
 ├── Thread 1
 ├── Thread 2
 └── Thread 3

Single GIL
     ↓
Only one thread executes Python code at a time
```

---

## Multiprocessing

```text
Process 1 → GIL 1 → Core 1
Process 2 → GIL 2 → Core 2
Process 3 → GIL 3 → Core 3
Process 4 → GIL 4 → Core 4
```

All processes can run simultaneously.

---

# Real-World Analogy

## Multithreading

Imagine:

- One kitchen
- Multiple chefs
- But only ONE chef can use the stove at a time

That is the GIL limitation.

---

## Multiprocessing

Now imagine:

- Multiple kitchens
- Each chef has their own stove

Everyone can cook simultaneously.

That is multiprocessing.

---

# Trade-Offs

Multiprocessing gives true parallelism, but has costs:

| Advantage | Drawback |
|---|---|
| Uses multiple CPU cores | Higher memory usage |
| Faster for CPU-heavy tasks | Process creation is slower |
| Avoids GIL limitations | Communication between processes is harder |

---

# Best Use Cases

Use multiprocessing for:

- Machine learning
- Data analysis
- Image/video processing
- Scientific computing
- Large mathematical calculations

---

# Summary

Multiprocessing achieves true parallelism because:

1. It creates separate processes
2. Each process has its own Python interpreter
3. Each process has its own GIL
4. The operating system runs processes on different CPU cores simultaneously

This allows Python programs to fully utilize multicore CPUs.

# Are Multiprocessing and Multithreading the Same on a Single-Core CPU?

Not exactly — but they become much more similar.

On a single-core CPU:

- Neither can achieve true parallelism
- Both rely on context switching
- Only one thread/process runs at a time

So both provide **concurrency**, not parallelism.

---

# But They Are Still Different

## Multithreading

Threads:

- Exist inside the same process
- Share memory
- Share one Python interpreter
- Share one GIL

```text
One Process
 ├── Thread A
 ├── Thread B
 └── Thread C
```

Only one thread executes Python bytecode at a time because of the GIL.

---

## Multiprocessing

Processes:

- Are completely separate
- Have separate memory
- Have separate Python interpreters
- Have separate GILs

```text
Process A
Process B
Process C
```

Even on a single-core CPU, the OS switches between processes.

---

# Main Difference on Single-Core

| Feature | Multithreading | Multiprocessing |
|---|---|---|
| Memory | Shared | Separate |
| GIL | One shared GIL | Separate GIL per process |
| Communication | Easy | Harder (IPC needed) |
| Overhead | Low | High |
| Execution on Single Core | Concurrent | Concurrent |

---

# Important Insight

Even though multiprocessing cannot achieve parallelism on a single-core CPU:

- It still bypasses the GIL limitation internally
- But there is no second core available to run another process simultaneously

So the benefit of multiprocessing is mostly lost on single-core systems.

---

# Analogy

## Single-Core CPU

Imagine:

- One worker
- Multiple jobs

Whether:

- The worker switches between threads
or
- The worker switches between processes

only one job is done at a time.

The difference is mainly in how memory and resources are organized.

---

# Final Answer

On a single-core CPU:

- Both multithreading and multiprocessing become forms of concurrency
- Neither achieves true parallelism
- But they are still architecturally different

The biggest differences remain:

- Memory sharing
- Process isolation
- GIL behavior
- Overhead
