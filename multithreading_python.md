# Creating and Managing Threads

**Theory**  
Discussion | C++ | Java | Python

---

# Introduction

In many real-world applications, tasks are performed sequentially, where one task must be completed before the next one begins. While this approach is simple, it may not always be the most efficient, especially when tasks take time to complete.

In this article, we will explore how multithreading can help optimize such scenarios by allowing multiple tasks to run concurrently, improving efficiency.

Let's explore a simple problem where multiple operations need to be executed sequentially, each taking a certain amount of time to complete.

---

# Problem Statement

Imagine you've placed an order, and the next steps involve sending three notifications:

- **Send an SMS:** This takes 2 seconds.
- **Send an Email:** This takes 3 seconds.
- **Send ETA (Estimated Time of Arrival):** This takes 5 seconds.

To simulate these delays in Python, we will use `time.sleep()` from the built-in `time` module.

Let's look at the code that tries to solve this problem, performing the tasks in a sequential manner (without using Multithreading).

---

## Code Simulating Delays Sequentially Without Multithreading

```python
import time

# Function to simulate sending SMS with a 2-second delay
def send_sms():
    time.sleep(2)  # Delay of 2 seconds
    print("SMS Sent!")

# Function to simulate sending Email with a 3-second delay
def send_email():
    time.sleep(3)  # Delay of 3 seconds
    print("Email Sent!")

# Function to simulate calculating the ETA with a 5-second delay
def calculate_eta():
    time.sleep(5)  # Delay of 5 seconds
    return "25 minutes"  # Returning the calculated ETA

# OrderService function that executes tasks sequentially
def order_service():
    print("Placing order...\n")

    # Send SMS and simulate the delay of 2 seconds
    send_sms()
    print("Task 1 done.\n")

    # Send Email and simulate the delay of 3 seconds
    send_email()
    print("Task 2 done.\n")

    # Calculate ETA with a delay of 5 seconds
    eta = calculate_eta()
    print(f"Order placed. Estimated Time of Arrival: {eta}")
    print("Task 3 done.\n")

# Entry point
if __name__ == "__main__":
    order_service()
```

---

# Understanding the Issue

In real-world applications, executing tasks sequentially like this can lead to significant inefficiencies, especially when tasks involve waiting periods, such as sending SMS, emails, or calculating ETAs.

In our current approach, each operation must complete before the next one begins, causing unnecessary delays and making the system slower than it needs to be.

For instance, while waiting for an SMS to be sent, the program is effectively "idle", unable to do anything else. This results in wasted time, especially in systems where responsiveness and speed are critical.

By running tasks sequentially, we also limit the system's ability to handle multiple operations concurrently, which can lead to poor user experiences, slower processing times, and bottlenecks in performance.

This is where multithreading comes into play.

By executing tasks concurrently, we can ensure that while one task is waiting (like sending an SMS), others can proceed without delay. Instead of idly waiting for each operation to finish, multithreading allows the system to be more responsive, efficiently utilizing time and resources.

In the next section, we will explore how multithreading can help overcome these issues, making the system more efficient and responsive.

---

# Implementing Multithreading by Subclassing `threading.Thread`

In Python, multithreading is provided through the built-in `threading` module.

One way to implement it is by creating a subclass of `threading.Thread` and overriding the `run()` method. Each thread will execute the `run()` method, allowing concurrent execution of tasks.

This directly mirrors the Java pattern of extending the `Thread` class.

---

## Key Concepts

- **threading.Thread:** The base class for all threads in Python's `threading` module. By subclassing it and overriding `run()`, we specify what the thread will do when it executes.

- **run() method:** Contains the code to be executed in the thread. Override this method in the subclass to define the thread's behavior.

- **start() method:** Starts the thread by calling `run()` in a new OS thread. Calling `run()` directly would execute it in the current thread without creating a new one.

- **join() method:** Blocks the calling thread until the thread on which it is called has finished execution. This ensures the main thread waits for all spawned threads to complete.

---

## Code

```python
import threading
import time

# Subclass of threading.Thread to send SMS
class SMSThread(threading.Thread):
    def run(self):
        time.sleep(2)  # 2-second delay for SMS
        print("SMS Sent using Thread subclass.")

# Subclass of threading.Thread to send Email
class EmailThread(threading.Thread):
    def run(self):
        time.sleep(3)  # 3-second delay for Email
        print("Email Sent using Thread subclass.")

# Subclass of threading.Thread to calculate ETA
class ETACalculationThread(threading.Thread):
    def run(self):
        time.sleep(5)  # 5-second delay for ETA calculation
        print("ETA Calculated using Thread subclass. Estimated Time: 25 minutes.")

if __name__ == "__main__":

    # Create thread objects for SMS, Email, and ETA Calculation
    sms_thread = SMSThread()
    email_thread = EmailThread()
    eta_thread = ETACalculationThread()

    print("Task Started.\n")

    # Start all threads
    sms_thread.start()
    print("Task 1 ongoing...")

    email_thread.start()
    print("Task 2 ongoing...")

    eta_thread.start()
    print("Task 3 ongoing...")

    # Wait for all threads to finish
    sms_thread.join()
    email_thread.join()
    eta_thread.join()

    print("All tasks completed.")
```

---

## Explanation

- The `SMSThread`, `EmailThread`, and `ETACalculationThread` classes subclass `threading.Thread` and override the `run()` method to simulate sending SMS, Email, and calculating ETA with delays.

- The `start()` method initiates the execution of each thread in a new OS thread.

- `join()` ensures the main thread waits for all tasks to finish before continuing.

The above code demonstrates three concurrent tasks: sending an SMS, sending an email, and calculating the ETA, each using a separate thread by subclassing `threading.Thread`.

---

# Demerit: No Return Value from the `run()` Method

A limitation of subclassing `threading.Thread` in Python is that the `run()` method cannot directly return a value to the caller.

This means that if you need to retrieve a result — like an ETA string — you cannot directly get it from the thread's `run()` method.

A workaround is to store the result in an instance variable and read it after `join()`, but this adds boilerplate.

This is a key drawback for tasks that require returning values.

> **Note:** The idiomatic Python way to get a result from a thread is discussed later in the article using `concurrent.futures`.

---

# Implementing Multithreading Using `threading.Thread` with a Target Function

In Python, another common way to implement multithreading is by passing a target function directly to `threading.Thread` via the `target` keyword argument.

This avoids subclassing entirely and mirrors Java's Runnable interface pattern — the task logic is separated from the thread management.

---

## Key Concepts

- **target Parameter:** The `threading.Thread` constructor accepts a `target` keyword argument — any callable (function, method, or lambda). The thread will call this callable when it starts, analogous to implementing `Runnable.run()` in Java.

- **args and kwargs:** Additional arguments to pass to the target function can be provided as a tuple via `args` or as a dict via `kwargs`.

- **start() Method:** Starts the thread's execution, invoking the target function in a new OS thread.

- **join() Method:** Ensures the main thread waits for all spawned threads to complete before continuing.

---

## Code

```python
import threading
import time

# Target function for sending SMS (no return value required)
def send_sms():
    time.sleep(2)  # 2-second delay for SMS
    print("SMS Sent using target function.")

# Target function for sending Email (no return value required)
def send_email():
    time.sleep(3)  # 3-second delay for Email
    print("Email Sent using target function.")

# Target function for calculating ETA (no return value required)
def calculate_eta():
    time.sleep(5)  # 5-second delay for ETA calculation
    print("ETA Calculated using target function. Estimated Time: 25 minutes.")

if __name__ == "__main__":

    # Create Thread objects and pass target functions to them
    sms_thread = threading.Thread(target=send_sms)
    email_thread = threading.Thread(target=send_email)
    eta_thread = threading.Thread(target=calculate_eta)

    print("Task Started.\n")

    # Start all threads
    sms_thread.start()
    print("Task 1 ongoing...")

    email_thread.start()
    print("Task 2 ongoing...")

    eta_thread.start()
    print("Task 3 ongoing...")

    # Wait for all threads to finish
    sms_thread.join()
    email_thread.join()
    eta_thread.join()

    print("All tasks completed.")
```

## Explanation
- Target Functions: Each task (SMS, Email, ETA calculation) is a plain Python function. These are passed as the target argument to threading.Thread.
- Thread Creation: Instead of subclassing threading.Thread, we create instances directly and pass the target callable. The thread calls the target when start() is invoked.
- Thread Execution: join() ensures the main thread waits for all tasks to finish before printing "All tasks completed."

The above code demonstrates three concurrent tasks: sending an SMS, sending an email, and calculating the ETA, each using a separate thread by passing target functions to threading.Thread.

## Advantages of Using Target Functions
- Separation of Concerns: The task logic (the function) is cleanly separated from the thread management (threading.Thread). The same function can be reused across multiple threads.
- Flexibility: Since threading.Thread accepts any callable as target, this approach works with functions, lambdas, bound methods, and any other callable — offering more flexibility than subclassing.

## Demerit: No Return Value from the Target Function

- A limitation of using plain target functions with threading.Thread is that the return value of the target function is discarded.

- If you need to retrieve a result — like an ETA string — you cannot directly get it from the thread.

This is a key drawback for tasks that require returning values.

## Fire and Forget: A Pattern in Multithreading

- Both the subclassing approach and the target function approach naturally follow the Fire and Forget pattern.

- In this pattern, tasks are initiated (fired), but the system doesn't wait for a result or confirmation of their completion.

- Instead, the tasks are executed independently in the background, and the caller doesn't need to know when or how they finish.

- This approach works well when the result of the task is not critical or when tasks do not need to communicate their results back to the main thread.

- For instance, sending SMS, emails, or calculating an ETA can be handled using these methods without waiting for confirmation of completion.

- However, while these approaches are great for many cases, they do have limitations, especially when the result of the task is needed.

This is where concurrent.futures comes into play.

In the next section, we will explore how to use it to handle tasks that require returning results.

## Implementing Multithreading Using concurrent.futures and Future

- In Python, the concurrent.futures module provides a high-level interface for asynchronously executing callables.

- The ThreadPoolExecutor class manages a pool of threads and returns Future objects that represent the result of asynchronous computations — directly analogous to Java's ExecutorService and Future.

- Unlike the previous approaches that use void-style functions, concurrent.futures allows the thread to return a value once the task completes, which can be retrieved via the Future object.

## Key Concepts
- ThreadPoolExecutor: Manages a pool of worker threads. The submit() method sends a callable to be executed by one of the threads and returns a Future object immediately.
- submit() method: Accepts a callable and optional arguments. It schedules the callable for execution and returns a Future representing the pending result.
- Future object: Represents the result of an asynchronous computation. It provides methods to check the status of the task and retrieve the result once it's available.
- Future.result(): Blocks the calling thread until the task completes and returns the result.


---

## Code

```python
import time
from concurrent.futures import ThreadPoolExecutor, Future

# Function to calculate ETA (requires a return value)
def calculate_eta() -> str:
    time.sleep(5)  # Simulate 5-second delay for ETA calculation
    print("ETA Calculated using concurrent.futures.")
    return "ETA: 25 minutes"

# Function to send SMS (no return value required)
def send_sms():
    time.sleep(2)
    print("SMS Sent using concurrent.futures.")

# Function to send Email (no return value required)
def send_email():
    time.sleep(3)
    print("Email Sent using concurrent.futures.")

if __name__ == "__main__":

    # Create a ThreadPoolExecutor with 3 worker threads
    with ThreadPoolExecutor(max_workers=3) as executor:

        # Submit all tasks
        eta_future = executor.submit(calculate_eta)
        sms_future = executor.submit(send_sms)
        email_future = executor.submit(send_email)

        print("Task Started.\n")
        print("Task 1 ongoing...")
        print("Task 2 ongoing...")
        print("Task 3 ongoing...")

        # Retrieve the result from the ETA future
        eta_result = eta_future.result()
        print(eta_result)

        # Wait for SMS and Email tasks
        sms_future.result()
        email_future.result()

    print("All tasks completed.")
```

Explanation
ThreadPoolExecutor: Manages a pool of 3 worker threads. The with statement ensures the executor shuts down gracefully when the block exits.
submit(): Schedules each function for execution and returns a Future object immediately, allowing the main thread to continue.
Result Retrieval with Future.result(): Calling eta_future.result() blocks the main thread until calculate_eta() completes and returns the value.
Using Future to Retrieve Results

When you submit a task using ThreadPoolExecutor.submit(), it returns a Future object.

The Future class provides methods to check the status of a task and retrieve its result once it completes.

Key Concepts
Future Object

Represents the result of an asynchronous computation. It can be used to:

Retrieve the result: The result() method blocks the main thread until the task completes and returns the result (or raises any exception from the task).
Check the status: done() returns True if the task has completed, and running() returns True if it is currently executing.
Cancel the task: The cancel() method attempts to cancel the task before it starts.

As shown in the code above, the result after the ETA calculation is retrieved using the Future object.

Another convenient method to wait on many futures at once is to use concurrent.futures.as_completed().

Using as_completed() to Retrieve Results

concurrent.futures.as_completed() is a utility function that yields futures as they complete, regardless of the order in which they were submitted.

This is especially useful when you have multiple tasks and want to process results as soon as each one finishes.

Key Concepts
as_completed(fs): Accepts an iterable of Future objects and returns an iterator that yields futures as they complete.
Future.result(): Called inside the loop to retrieve the result of each completed future.
Example Code: Using as_completed() for Retrieval
___

## Code

```python
import time
from concurrent.futures import ThreadPoolExecutor, as_completed

# Function to calculate ETA
def calculate_eta() -> str:
    time.sleep(5)
    print("ETA calculated.")
    return "ETA: 25 minutes"

# Function to send SMS
def send_sms() -> str:
    time.sleep(2)
    print("SMS Sent using Runnable.")
    return "SMS done"

# Function to send Email
def send_email() -> str:
    time.sleep(3)
    print("Email Sent using Runnable.")
    return "Email done"

if __name__ == "__main__":

    print("Task Started.\n")

    with ThreadPoolExecutor(max_workers=3) as executor:

        futures = {
            executor.submit(calculate_eta): "ETA",
            executor.submit(send_sms): "SMS",
            executor.submit(send_email): "Email",
        }

        # Process results as each task completes
        for future in as_completed(futures):
            task_name = futures[future]
            result = future.result()

            print(f"{task_name} finished with result: {result}")

    print("All tasks completed.")
```
Other Ways to Implement Multithreading in Python

In addition to the methods we've discussed so far, there are other convenient ways to implement multithreading in Python.

These approaches offer different levels of simplicity and flexibility, depending on the task at hand.

1. Directly Passing a Lambda as the Target

In Python, you can pass a lambda function as the target argument to threading.Thread.

This allows you to define short, inline tasks without needing to write a named function.

This is a concise and efficient way to handle simple tasks.

Example
___

## Code

```python
import threading
import time

if __name__ == "__main__":

    # Define the thread target inline using a lambda expression
    t = threading.Thread(
        target=lambda: (
            time.sleep(2),
            print("Task completed using lambda.")
        )
    )

    t.start()
    t.join()
```
Explanation
A lambda expression is passed as the target to threading.Thread.
The thread begins executing the lambda immediately when start() is called.

This approach is particularly useful for short-lived tasks where you don't need a named, reusable function.

2. Passing Arguments to Target Functions

With threading.Thread, you can pass arguments to the target function using the args (tuple) and kwargs (dict) constructor parameters.

This allows you to parameterize thread tasks, making them more flexible and reusable.

Example
___

## Code

```python
import threading
import time

# Function that accepts arguments
def send_notification(notification_type: str, delay_seconds: int):
    time.sleep(delay_seconds)
    print(f"{notification_type} sent after {delay_seconds} second(s).")

if __name__ == "__main__":

    # Pass arguments via the 'args' parameter
    sms_thread = threading.Thread(
        target=send_notification,
        args=("SMS", 2)
    )

    email_thread = threading.Thread(
        target=send_notification,
        args=("Email", 3)
    )

    sms_thread.start()
    email_thread.start()

    sms_thread.join()
    email_thread.join()
```
Explanation
The send_notification function takes a notification_type and a delay_seconds parameter.
These are forwarded by the threading.Thread constructor via args=(...).
The same function is reused for both SMS and Email threads with different arguments.
Advantages of Lambda Expressions and Argument Passing
Conciseness: Lambdas eliminate the need for separate function definitions for simple tasks.
Flexibility: Argument passing via args and kwargs makes functions reusable across multiple threads with different parameters.
Readability: Both approaches express the task logic close to where the thread is created, improving code locality.

Lambda expressions and argument passing are especially helpful when you need to pass short, parameterized tasks to a thread without writing boilerplate code.

Thread Lifecycle Flow

The lifecycle of a Python thread follows a specific flow as shown in the diagram.

States in Thread Lifecycle

A Python thread undergoes various states during its execution.

These states represent different phases a thread can be in from its creation to termination.

Below are the key states in the lifecycle:

New (Initialized): The threading.Thread object has been created but start() has not yet been called.
Runnable: start() has been called. The thread is alive and waiting for the OS scheduler to assign it CPU time.
Running: The thread is actively executing its target function or run() method.
Blocked / Waiting: The thread is waiting for a lock (threading.Lock), a condition (threading.Condition), an event (threading.Event), or for time.sleep() to expire.
Timed Waiting: The thread is blocked for a specific amount of time due to a call to time.sleep() or Event.wait(timeout).
Terminated (Dead): The target function has returned, or an unhandled exception has occurred. The thread is no longer alive. is_alive() returns False.
State Transitions
New → Runnable: When start() is called.
Runnable → Running: When the OS scheduler assigns CPU time to the thread.
Running → Blocked/Waiting: When the thread acquires a lock, waits on a condition, or calls time.sleep().
Blocked/Waiting → Runnable: When the lock is released, the condition is notified, or the sleep expires.
Running → Waiting: When the thread calls join() on another thread or waits on an event.
Waiting → Runnable: When the waited-upon thread finishes or the event is set.
Timed Waiting → Runnable: When the waiting time expires.
Running → Terminated: When the target function returns or an unhandled exception occurs.