A thread is the smallest unit of execution within a program, representing a sequence of instructions that can run concurrently with other threads.

# 4-Threads

## Table of Contents
- [1) Thread Life Cycle](#1-thread-life-cycle)
- [2) Creating and Starting Threads (Two Ways)](#2-creating-and-starting-threads-two-ways)
- [3) `start()` vs `run()` — Classic Trap](#3-start-vs-run--classic-trap)
- [4) Deadlock and Prevention via Global Lock Ordering](#4-deadlock-and-prevention-via-global-lock-ordering)
- [5) Starvation vs Deadlock](#5-starvation-vs-deadlock)
- [6) `volatile`: Visibility, Ordering, and Limits](#6-volatile-visibility-ordering-and-limits)
- [7) `sleep()`, `wait()`, `notify()`, `notifyAll()`](#7-sleep-wait-notify-notifyall)
- [8) Precision Corrections / Exam-Safe Refinements](#8-precision-corrections--exam-safe-refinements)
- [9) High-Value Oral One-Liners to Memorize](#9-high-value-oral-one-liners-to-memorize)
- [10) Possible Professor Questions (with Model Answers)](#10-possible-professor-questions-with-model-answers)

---

## 1) Thread Life Cycle

A thread moves through distinct states. Conceptually, this is often explained as:

`NEW → RUNNABLE → RUNNING → (BLOCKED / WAITING / TIMED_WAITING) → TERMINATED`

### Conceptual meaning of each stage
- **NEW**: Thread object created, but `start()` not called yet.
- **RUNNABLE**: `start()` called; eligible to run, waiting for CPU scheduling.
- **RUNNING** (conceptual): scheduler is currently executing thread code.
- **BLOCKED / WAITING / TIMED_WAITING**:
  - **BLOCKED**: waiting to acquire a monitor lock (e.g., entering a `synchronized` block).
  - **WAITING**: waiting indefinitely for another thread action (e.g., `wait()`, `join()` without timeout).
  - **TIMED_WAITING**: waiting up to a specified time (`sleep(1000)`, `wait(timeout)`, `join(timeout)`).
- **TERMINATED**: execution finished; cannot restart same thread instance.

### Critical Java precision
In official Java `Thread.State`, there is **no separate `RUNNING` enum value**.  
Official states are exactly:

- `NEW`
- `RUNNABLE`
- `BLOCKED`
- `WAITING`
- `TIMED_WAITING`
- `TERMINATED`

So in Java’s model, a thread that is “actually executing” is still represented as **RUNNABLE**.

---

## 2) Creating and Starting Threads (Two Ways)

### Way 1 — Extend `Thread`

```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Running in a separate thread!");
    }
}

MyThread t = new MyThread(); // NEW
t.start();                   // RUNNABLE (then scheduled to execute run())
```

- Class **is a Thread**.
- Direct but less flexible due to Java single inheritance.

### Way 2 — Implement `Runnable` (Usually Preferred)

```java
class MyTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Running via Runnable!");
    }
}

MyTask task = new MyTask();  // just a task object
Thread t = new Thread(task); // actual thread wrapper
t.start();                   // starts concurrent execution
```

- Class **is not a Thread**, it is a task.
- Better separation of concerns.
- Preserves your one `extends` slot for another base class.

### Why `Runnable` is generally preferred
Java allows only **single class inheritance**.  
If your class already extends another class, it cannot also extend `Thread`.  
So `Runnable` is usually the default design choice.

---

## 3) `start()` vs `run()` — Classic Trap

```java
MyThread t = new MyThread();
t.run();   // WRONG for concurrency: normal method call on current thread
t.start(); // RIGHT: creates a new thread; JVM later invokes run()
```

- `run()` directly = no new thread.
- `start()` = real concurrent execution path.

---

## 4) Deadlock and Prevention via Global Lock Ordering

### Why deadlock happens
Deadlock typically appears when threads lock shared resources in opposite order:

- Thread A: locks X, then waits for Y
- Thread B: locks Y, then waits for X

This creates circular wait → permanent standstill.

A key insight: once a thread is blocked on a lock, it cannot “negotiate” or “swap” locks. It is paused while still holding what it already acquired.

### Practical prevention strategy: consistent global lock order
Force all threads to acquire multiple locks in one agreed order (e.g., lower account ID first).

```java
class BankAccount {
    private final int id; // unique ID
    private double balance;

    void transfer(BankAccount to, double amount) {
        BankAccount first  = (this.id < to.id) ? this : to;
        BankAccount second = (this.id < to.id) ? to : this;

        synchronized (first) {
            synchronized (second) {
                this.balance -= amount;
                to.balance += amount;
            }
        }
    }
}
```

Because every thread uses the same ordering rule, circular wait cannot form.

### Edge-case precision
If IDs are not truly unique, ordering can become ambiguous.  
Production-safe designs either:
- enforce strict ID uniqueness, or
- add a deterministic tie-breaker lock/policy.

### Exam principle
> One common way to prevent deadlock is to enforce a consistent, global lock ordering — ensuring all threads always acquire multiple locks in the same fixed order, which eliminates the circular wait[...]

---

## 5) Starvation vs Deadlock

They are different concurrency failures.

### Deadlock
- Structural circular dependency.
- Threads are permanently stuck.
- No scheduling luck can fix it.

### Starvation
- Fairness/scheduling problem.
- A thread is not structurally blocked forever, but repeatedly denied resource access.
- Can continue indefinitely in practice if other threads keep winning.

Example scenario: lower-priority thread repeatedly preempted by constant higher-priority arrivals.

### Strong contrast line
> Deadlock is a permanent standstill caused by circular resource dependencies — mathematically guaranteed to never resolve on its own. Starvation is a fairness problem where a thread is repeate[...]

---

## 6) `volatile`: Visibility, Ordering, and Limits

### Core problem it addresses
Without synchronization/volatile, one thread may update a variable while another thread keeps observing a stale value (e.g., due to caching/reordering effects under the Java Memory Model).

### Example without `volatile`

```java
class Flag {
    private boolean running = true; // not volatile

    void stop() {
        running = false;
    }

    void doWork() {
        while (running) {
            // work
        }
        System.out.println("Stopped!");
    }
}
```

Risk: worker thread may keep seeing `running == true` longer than expected.

### Fix with `volatile`

```java
class Flag {
    private volatile boolean running = true;

    void stop() {
        running = false;
    }

    void doWork() {
        while (running) {
            // work
        }
        System.out.println("Stopped!");
    }
}
```

### What `volatile` guarantees
- **Visibility**: writes by one thread become visible to others reading that same variable.
- **Ordering (happens-before)**: a write to a volatile variable happens-before subsequent reads of that variable by other threads.

### Important precision
A simplified teaching phrase is “read/write goes to main memory,” but exam-grade phrasing is:
- implementations may still use caches,
- yet JVM/CPU memory semantics enforce visibility and ordering guarantees for volatile accesses.

### What `volatile` does NOT guarantee
- No mutual exclusion.
- No atomicity for compound actions.

```java
volatile int x = 0;
x++; // still non-atomic (read-modify-write)
```

So `volatile` is excellent for flags, but not a replacement for `synchronized`/locks in compound critical sections.

---

## 7) `sleep()`, `wait()`, `notify()`, `notifyAll()`

These are core coordination primitives and very common oral exam questions.

### `Thread.sleep(...)`
- `sleep(ms)` pauses the **current thread** for at most that duration.
- Puts thread into **TIMED_WAITING**.
- On timeout, thread returns to runnable.
- **Does not release locks** currently held by that thread.

```java
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt(); // preserve interrupt status
}
```

### `wait()`
- Method of **`Object`**, not `Thread`.
- Must be called while owning the object's monitor (`synchronized(lock)`).
- Causes current thread to enter:
  - **WAITING** (`wait()`), or
  - **TIMED_WAITING** (`wait(timeout)`).
- **Releases that monitor lock** while waiting.

```java
synchronized (lock) {
    while (!condition) {
        lock.wait(); // releases lock and waits
    }
    // resumes only after re-acquiring lock
}
```

Why `while` instead of `if`:
- protects against spurious wakeups,
- re-checks condition correctness after wake-up.

### `notify()` and `notifyAll()`
- Also methods of **`Object`**.
- Must be invoked while holding the same monitor.
- `notify()` wakes one waiting thread (choice is scheduler-dependent).
- `notifyAll()` wakes all waiting threads; they compete to re-acquire the monitor.

```java
synchronized (lock) {
    condition = true;
    lock.notifyAll();
}
```

### State mapping summary
- `sleep(1000)` → `TIMED_WAITING`
- `wait()` → `WAITING`
- `wait(1000)` / `join(1000)` → `TIMED_WAITING`
- waiting to enter `synchronized` → `BLOCKED`

### `sleep()` vs `wait()` (must-know contrast)
- `sleep()`:
  - Thread method
  - pure timed pause
  - **does not** release monitor lock
- `wait()`:
  - Object method
  - coordination primitive
  - must hold monitor to call
  - **does** release monitor while waiting
  - resumes after notification + monitor reacquisition

### Mini coordination example
```java
class Signal {
    private boolean ready = false;

    public synchronized void produce() {
        ready = true;
        notifyAll();
    }

    public synchronized void consume() throws InterruptedException {
        while (!ready) {
            wait();
        }
        ready = false;
    }
}
```

### Oral-ready precision line
> `wait()`/`notify()` are monitor-based coordination methods on `Object`; `wait()` releases the monitor and suspends until notification, while `sleep()` is a timed pause on `Thread` that does not rele[...] 

---

## 8) Precision Corrections / Exam-Safe Refinements

1. **State model precision**  
   Conceptual “running” is okay for intuition, but official Java enum has no `RUNNING`; execution is represented by `RUNNABLE`.

2. **Termination wording**  
   Prefer: a thread terminates when `run()` exits normally or by uncaught exception.  
   Avoid relying on deprecated `Thread.stop()` language.

3. **Volatile wording precision**  
   Use visibility + happens-before terminology, not absolute “never cached.”

4. **Atomicity reminder**  
   `volatile` does not make compound operations atomic (`x++` still unsafe without additional synchronization/atomics).

5. **Deadlock example robustness**  
   Ensure unique ordering key (ID uniqueness) or add deterministic tie-breaker.

---

## 9) High-Value Oral One-Liners to Memorize

- “Prefer `Runnable` because it separates task from thread and preserves single inheritance flexibility.”
- “`start()` creates a new execution thread that invokes `run()`; direct `run()` is just a normal call on the current thread.”
- “Use global lock ordering so all threads acquire shared locks in one fixed order, eliminating circular wait.”
- “Deadlock is permanent circular waiting; starvation is indefinite postponement caused by unfair scheduling.”
- “`volatile` guarantees visibility and ordering (happens-before) for a variable across threads, but not mutual exclusion or compound-operation atomicity.”
- “`sleep()` does not release locks; `wait()` releases the monitor and requires synchronized context.”

---

## 10) Possible Professor Questions (with Model Answers)

### Q1) What are Java’s official thread states?
**A:** `NEW`, `RUNNABLE`, `BLOCKED`, `WAITING`, `TIMED_WAITING`, `TERMINATED`.

### Q2) Is there a `RUNNING` state in `Thread.State`?
**A:** Not as an enum constant. In Java’s official model, actively executing threads are still represented as `RUNNABLE`.

### Q3) Why is implementing `Runnable` often preferred to extending `Thread`?
**A:** Because Java has single inheritance. `Runnable` avoids consuming the only `extends` slot and separates task logic from thread execution mechanism.

### Q4) What is the difference between `start()` and `run()`?
**A:** `start()` creates a new thread and schedules `run()` there; calling `run()` directly executes on the current thread with no concurrency.

### Q5) Define deadlock.
**A:** Deadlock is a permanent blocking situation where threads wait in a cycle for resources held by each other (circular wait), so none can proceed.

### Q6) How does lock ordering prevent deadlock?
**A:** If every thread acquires multiple locks in a single global order, circular wait cannot occur, so deadlock is prevented.

### Q7) What is starvation, and how is it different from deadlock?
**A:** Starvation is indefinite postponement due to unfair scheduling/resource allocation; unlike deadlock, progress is not structurally impossible.

### Q8) What does `volatile` guarantee?
**A:** Visibility of latest writes across threads and ordering (happens-before) between volatile writes and subsequent volatile reads of the same variable.

### Q9) Does `volatile` make `x++` thread-safe?
**A:** No. `x++` is a non-atomic read-modify-write sequence. Use synchronization or atomic classes.

### Q10) Difference between `sleep()` and `wait()`?
**A:** `sleep()` is `Thread`-based timed pause and does not release locks. `wait()` is `Object`-based coordination, must be called under synchronized monitor ownership, and releases the monitor w[...] 

### Q11) Why use `while` around `wait()`?
**A:** To handle spurious wakeups and to re-check the guard condition after waking before proceeding.

### Q12) Can a terminated thread be restarted?
**A:** No. Once terminated, that `Thread` instance cannot be started again; create a new thread object.
