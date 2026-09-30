**Navigation:** [Unit index](00_Index.md) · [Previous: 3.6 Monitors and condition variables](06_Monitors_and_Condition_Variables.md) · [Next: 3.8 Atomic operations and lock-free algorithms](08_Atomic_and_Lock_Free.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 3 — Process Synchronization and Deadlocks

## Topic 3.7 — Mutexes and Spinlocks

### Learning Outcomes

- Define a mutex and a spinlock.
- Explain ownership of a mutex.
- Choose a spinlock or a blocking mutex according to the length of the critical section.
- Differentiate a mutex from a binary semaphore.

### Prerequisites

Test-and-set, compare-and-swap, and the blocking semaphore (Topics 3.2 and 3.4).

### Introduction

The word mutex is short for mutual exclusion. In modern code it means a lock with an owner: the thread that locked it is the thread that unlocks it. A spinlock is a mutex that waits by repeating an atomic instruction instead of going to sleep. Both protect a critical section. They spend the waiting time differently, and that difference decides which one belongs in a device driver and which one belongs around a disk read.

### Definition

**Definition:**  
A mutex is a lock used for mutual exclusion. It is unlocked or locked. The thread that locks it becomes its owner and is the only thread allowed to unlock it.

**Definition:**  
A spinlock is a lock whose waiting thread repeatedly executes an atomic test, such as test-and-set or compare-and-swap, until the lock becomes free.

### How each wait works

```text
Mutex, blocking

lock:
    atomically try to take the lock
    if it is already held
        put this thread on the mutex queue and sleep
unlock:                      // only the owner may call this
    release the lock
    if a thread is waiting, wake one
```

```text
Spinlock

lock:
    while (test_and_set(&lock) == true)
        ;                    // stay on this core and try again
unlock:
    lock = false
```

The spinlock’s waiting thread stays Running. The blocking mutex’s waiting thread is Waiting. That is the operational difference.

### When each lock is the right tool

| Situation | Better lock | Reason |
| --- | --- | --- |
| Critical section of a few instructions on a multicore machine | Spinlock | Sleeping and waking would cost more than the remaining wait |
| Critical section that may read a disk or wait for a user | Blocking mutex | A spinning thread would occupy a core for a long time |
| Code that runs with interrupts off or that cannot sleep | Spinlock | There is no scheduler sleep available in that context |
| One core only, and the holder is not running | Blocking mutex | A spin on the only core cannot finish until the holder runs, and the spinner is the one occupying the core |

A spinlock on a uniprocessor is a special danger. If the holder is descheduled while the lock is held, the spinner runs and never sees the lock released, because the holder is not running. Kernel spinlocks are therefore used together with a rule that the holder is not preempted, or they are used on multiprocessors where another core can release the lock.

### Mutex and binary semaphore

| Parameter | Mutex | Binary semaphore |
| --- | --- | --- |
| Values | Locked or unlocked | 0 or 1 |
| Ownership | The locking thread unlocks | Any thread may signal |
| A signal with no previous wait | Not a legal unlock by a non-owner | Can raise the value and accidentally create an extra permit |
| Typical use | A critical section belonging to one thread at a time | A permit that may be released by a different thread, such as “buffer slot filled” |
| Waiting | Usually a sleep queue; a spinlock is the spinning form | Busy-wait or blocking, depending on the implementation |

The producer’s `signal(full)` is not a mutex unlock. The consumer, a different thread, is supposed to complete that handshake. A mutex would be the wrong tool there because the thread that “locks” would have to be the thread that “unlocks.” `mutex` in the bounded-buffer program is the one that fits a mutex. `full` and `empty` fit counting semaphores.

### Recursive locking

A thread that already owns an ordinary mutex and locks it again can wait for itself. That is a self-deadlock. Some libraries offer a recursive mutex, which counts nested locks by the same owner and releases the lock only when the count returns to zero. Unless the mutex is documented as recursive, nested locking is an error.

### Advantages and limitations

| Lock | Advantages | Limitations |
| --- | --- | --- |
| Blocking mutex | The waiter does not burn a core; ownership catches an unlock by the wrong thread | Sleep and wakeup are expensive for a few instructions |
| Spinlock | Very fast when the lock is held briefly; usable where sleeping is forbidden | Wastes a core; unsafe if the holder cannot run |
| Either | Provides mutual exclusion for a critical section | Neither provides a fair queue unless the implementation adds one; a newcomer can win the next race |

### Common mistakes

- Calling every binary semaphore a mutex. Ownership is the difference.
- Spinning around a critical section that performs I/O.
- Unlocking a mutex from a thread that did not lock it.
- Locking a non-recursive mutex twice in one thread and expecting the second lock to succeed.
- Using a spinlock on the only CPU while the holder can be preempted.

### Important exam points

- Mutex: owner locks and unlocks; others sleep or fail.
- Spinlock: busy wait on test-and-set or compare-and-swap.
- Short critical section, especially on a multicore kernel path: spin. Long or blocking section: sleep.
- `full` and `empty` are semaphores. The buffer’s exclusion lock can be a mutex.
- Nested lock of a non-recursive mutex deadlocks with itself.

### University exam questions

#### 2-mark questions

1. Define a mutex.
2. Define a spinlock.
3. Who may unlock a mutex?
4. Why is a spinlock unsuitable for a long critical section?

#### 4/5-mark questions

1. Differentiate a mutex and a spinlock.
2. Differentiate a mutex and a binary semaphore.
3. Explain when the kernel prefers a spinlock.

#### 8/10-mark questions

1. Explain mutexes and spinlocks. Compare their waiting methods and give one correct use of each.
2. Explain ownership. Show why the producer-consumer `full` semaphore is not a mutex, and why the buffer exclusion lock is.

### Practice problems

#### Easy

1. Does a spinning thread stay in the Running state?
2. Does a thread blocked on a mutex stay in the Running state?
3. Name the atomic instruction a spinlock can use.
4. Can thread B unlock a mutex owned by thread A?
5. Which of `mutex`, `full`, and `empty` in the buffer solution has an owner?

#### Medium

1. The critical section is four assignments and the machine has four cores. Which lock is reasonable, and why?
2. The critical section reads a disk. Which lock is reasonable, and why?
3. Explain self-deadlock on a non-recursive mutex.
4. Why can a uniprocessor spinlock fail if the holder is preempted?
5. A binary semaphore is signalled by a thread that never waited. How can that break exclusion? Why does mutex ownership forbid the same call?

#### Hard

1. A spinlock is held for 2 microseconds and a context switch costs 10 microseconds. Explain the cost of blocking instead of spinning.
2. The same lock is then held while a 5-millisecond disk read runs. Reconsider the choice.
3. Describe a recursive mutex count going from 0 to 2 and back to 0 as one thread nests two calls.
4. Why do several cores spinning on one lock hurt threads that are not even waiting for that lock?
5. Compare fairness of a simple spinlock with the bounded-waiting test-and-set of Topic 3.2.

**Answers**

- The buffer exclusion lock is the mutex. `full` and `empty` are released by the other thread.
- A 2-microsecond hold is shorter than a 10-microsecond switch, so spinning costs less than sleeping.
- A 5-millisecond hold is hundreds of times a switch. The waiter should sleep.
- Nested calls: unlocked, lock → count 1, lock → count 2, unlock → count 1, unlock → count 0 and the lock becomes free.
- Spinning cores still need the memory system and the scheduler. Other threads on those cores do not run.

### MCQs

**Q1. A mutex records:**

A. Which thread owns the lock  
B. A count of buffer slots that any thread may signal  
C. The period of a real-time task  
D. Peterson’s turn for an arbitrary number of processes  

**Answer:** A  

**Explanation:** Ownership is what distinguishes a mutex from a general semaphore.

**Q2. A spinlock waits by:**

A. Repeating an atomic test until the lock is free  
B. Moving the thread to the waiting state immediately  
C. Signalling a condition variable with nobody waiting  
D. Disabling every disk  

**Answer:** A  

**Explanation:** The thread remains scheduled and executes test-and-set or compare-and-swap.

**Q3. The thread allowed to unlock a mutex is:**

A. The thread that locked it  
B. Any thread in the process  
C. The consumer, if the producer locked it, by the definition of a mutex  
D. The scheduler, instead of the owner  

**Answer:** A  

**Explanation:** Unlock by a non-owner is an error. A semaphore does not have this rule.

**Q4. A blocking mutex is preferred when the critical section:**

A. May wait for I/O  
B. Contains two machine instructions on a multiprocessor  
C. Runs with sleep forbidden  
D. Is the inner update of the spinlock word itself  

**Answer:** A  

**Explanation:** A long wait should release the core. A few instructions are the spinlock case.

**Q5. A spinlock on one CPU can hang if:**

A. The holder is preempted and the spinner occupies the only CPU  
B. The lock is left free  
C. Compare-and-swap is atomic  
D. The owner unlocks before anyone waits  

**Answer:** A  

**Explanation:** The spinner never gives the holder a chance to run and release the lock.

**Q6. `signal(full)` in the producer is:**

A. A semaphore operation that another thread will consume  
B. A mutex unlock by the producer, who is the owner of `full`  
C. A condition signal that is stored in `count`  
D. Test-and-set  

**Answer:** A  

**Explanation:** The consumer, not the producer, completes the handshake by waiting on `full`. That is semaphore behaviour, not mutex ownership.

**Q7. Locking a non-recursive mutex twice in the same thread:**

A. Can make the thread wait for a lock it already holds  
B. Always increases a legal recursion count  
C. Unlocks the mutex  
D. Converts it into a counting semaphore of value n  

**Answer:** A  

**Explanation:** The second lock sees the mutex held and waits. The only thread that could unlock it is the thread that is waiting.

**Q8. Compared with a blocking mutex, a spinlock:**

A. Avoids sleep and wakeup when the hold time is very short  
B. Uses less CPU during a long wait  
C. May be unlocked by any thread  
D. Provides the readers-writers counter by itself  

**Answer:** A  

**Explanation:** The gain is the avoided context switch. The loss is the CPU time spent in the loop.

### Quick revision

#### Mutex

Locked or unlocked. Only the owner unlocks. A normal mutex sleeps if the lock is held. Nested locking deadlocks unless the mutex is recursive.

#### Spinlock

Busy wait on an atomic instruction. Right for a few instructions, especially on a multicore machine. Wrong across I/O, and dangerous on a single CPU if the holder can be preempted.

#### Semaphore distinction

A mutex has an owner. A binary semaphore may be signalled by a different thread. Use the semaphore for `full` and `empty`. Use the mutex for the buffer structure.
