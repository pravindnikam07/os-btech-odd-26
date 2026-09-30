**Navigation:** [Unit index](00_Index.md) · [Previous: 3.3 Peterson’s solution](03_Petersons_Solution.md) · [Next: 3.5 Classical problems](05_Classical_Problems.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 3 — Process Synchronization and Deadlocks

## Topic 3.4 — Semaphores

### Learning Outcomes

- Define a semaphore and the P and V operations.
- Differentiate a counting semaphore and a binary semaphore.
- Implement wait and signal so that a waiting process sleeps instead of spinning.
- Use a semaphore to enforce mutual exclusion, and avoid the deadlock caused by taking two semaphores in opposite orders.

### Prerequisites

Critical section and busy waiting (Topics 3.1 and 3.2). Process states: a waiting process can be blocked and later made ready.

### Introduction

A semaphore is an integer on which only two operations are allowed: wait and signal. Dijkstra called them P and V. The integer is not read or written by ordinary assignment. That restriction is what makes the semaphore usable as a lock and as a counter of free resources. A binary semaphore has the values 0 and 1 and serves as a lock. A counting semaphore ranges over a larger set of integers and serves as a pool of identical resources.

### Definition

**Definition:**  
A semaphore is a shared integer variable accessed only through the atomic operations wait (P) and signal (V).

**Definition:**  
A binary semaphore takes only the values 0 and 1. A counting semaphore takes non-negative values in the busy-wait definition, and may be negative in the blocking definition, where the magnitude records the number of waiting processes.

### The operations

The textbook’s first definition busy-waits:

```text
wait(S):                     // also called P(S) or down(S)
    while (S <= 0)
        ;                    // do nothing
    S = S - 1;

signal(S):                   // also called V(S) or up(S)
    S = S + 1;
```

`S = S - 1` must not start until the test has found S positive, and no other wait or signal may touch S in between. The test and the decrement are one critical section. Signal’s increment is also atomic.

P comes from the Dutch *proberen*, to test. V comes from *verhogen*, to increment. Examination answers may use any of the pairs wait/signal, P/V, or down/up, provided the pair is used consistently.

### Blocking implementation

Busy waiting spends a core. The operating-system form puts the process on a queue:

```text
wait(S):
    S = S - 1;
    if (S < 0)
        block this process on S's queue;

signal(S):
    S = S + 1;
    if (S <= 0)
        wake one process from S's queue;
```

```text
typedef struct {
    int value;
    process_queue queue;
} semaphore;
```

If three processes wait on a semaphore whose value started at 0, the value becomes −3. The absolute value is the length of the queue. A signal increases the value and wakes exactly one process. Waking a process moves it to the ready queue. It does not immediately run unless the scheduler selects it.

| | Busy-wait semaphore | Blocking semaphore |
| --- | --- | --- |
| Waiting process | Spins | Sleeps on a queue |
| Value while processes wait | Stays 0 until a signal | Becomes negative |
| Meaning of a negative value | Not used | Number of processes on the queue |
| Suitable when | The wait is shorter than a context switch | The wait may be long |
| Still required | Atomic update of the integer | Atomic update of the integer and the queue |

The spin is removed from the application. A very short spin may remain inside the kernel while it updates the semaphore structure. That inner spin is the spinlock of Topic 3.7.

### Mutual exclusion

```text
semaphore mutex = 1;

repeat
    wait(mutex);
    critical section
    signal(mutex);
    remainder section
until false
```

The first process finds 1, decrements it to 0, and enters. The next process finds 0 and waits. The exit signal makes the resource available to one waiter. For a binary semaphore used this way, every wait that enters must have a matching signal, and the process that entered is the one that signals.

### Counting a pool of resources

`S = n` means n identical units are free, for example n buffers or n printers of the same type.

```text
wait(S);          // take one unit; wait if none is free
use the resource
signal(S);        // return one unit
```

This is not mutual exclusion among different kinds of work. It is a count. Mutual exclusion of the list that records which unit was taken is a separate binary semaphore if that list is shared.

### Two semaphores and deadlock

Process A and process B both need semaphore X and semaphore Y.

```text
A                          B
wait(X)                    wait(Y)
wait(Y)                    wait(X)
```

A holds X and waits for Y. B holds Y and waits for X. Neither signals, so neither wait ends. This is a deadlock. The usual prevention inside semaphore code is to take the locks in the same order in every process:

```text
A                          B
wait(X)                    wait(X)
wait(Y)                    wait(Y)
```

The producer-consumer program in the next topic is the standard place where the order of two waits is part of the correctness, not a matter of taste.

### Advantages and limitations

| Advantages | Limitations |
| --- | --- |
| One mechanism expresses both a lock and a resource count | The programmer must pair waits and signals; a missing signal leaves processes blocked |
| The blocking form does not waste a core during a long wait | The wrong order of two waits deadlocks |
| Binary and counting forms use the same two operations | A semaphore has no record of which process waited; any process may signal, so a programming error is easy |
| P and V are the operations assumed by the classical problems | The integer must never be touched except by wait and signal |

### Common mistakes

- Writing `S = S - 1` in the application instead of wait. The decrement is legal only inside the atomic operation.
- Using a counting semaphore whose initial value is 1 and calling every such semaphore a binary semaphore without also restricting later values. A binary semaphore is not allowed to go above 1.
- Signalling twice on a binary mutex “to be safe.” The extra signal can let two processes into the critical section.
- Reversing wait and signal. Signal at the start and wait at the end does not protect the section.
- Forgetting that a blocking wait may leave S negative. In that implementation, S is not the number of free units once processes are asleep; |S| waiters exist when S is negative.

### Important exam points

- wait is P; signal is V.
- Binary semaphore: 0 or 1, used as mutex, initial value 1 for an unlocked lock.
- Counting semaphore: initial value equal to the number of free resources.
- Blocking form: `S--`, sleep if S < 0; `S++`, wake one process if S ≤ 0.
- Same lock order on every path. Opposite orders deadlock.
- Do not access the integer except through P and V.

### University exam questions

#### 2-mark questions

1. Define a semaphore.
2. What are P and V?
3. Differentiate a binary semaphore and a counting semaphore.
4. Write the wait and signal used for mutual exclusion.

#### 4/5-mark questions

1. Explain the blocking implementation of wait and signal.
2. Show how a binary semaphore provides mutual exclusion.
3. Show a deadlock caused by two semaphores taken in opposite orders.

#### 8/10-mark questions

1. Explain counting and binary semaphores, the busy-wait operations, and the blocking operations. State what a negative value means.
2. Explain mutual exclusion with a semaphore and the deadlock that appears when two processes acquire two semaphores in different orders.

### Practice problems

#### Easy

1. Which operation decrements a semaphore?
2. A mutex semaphore should be initialised to what value if the critical section is initially free?
3. Five identical buffers are free. What is the initial value of a counting semaphore that represents them?
4. Name the Dutch letters for wait and signal.
5. In the blocking implementation, three processes are asleep and no signal has woken them. What is S if it started at 0?

#### Medium

1. Rewrite the busy-wait wait so that a reader can see where the atomic region must be.
2. Explain why signal wakes only one process.
3. Two processes execute wait(mutex) and neither executes signal. What happens to the second process?
4. Draw the opposite-order deadlock with semaphores X and Y.
5. Why is a busy-wait semaphore a poor lock for a critical section that performs disk I/O?

#### Hard

1. S starts at 2. Give one sequence of three waits and one signal, and state the value after each operation in the blocking implementation. Let the first two waits find S positive.
2. Show that two extra signal operations on a binary mutex can destroy mutual exclusion if the implementation allows the value to become 2.
3. A process waits on full and then on mutex. Another waits on mutex and then on full. Describe the deadlock. The producer-consumer topic fixes the order; here, identify only the circular wait.
4. Compare the meaning of S = 0 in the busy-wait definition and S = −4 in the blocking definition.
5. Why must the queue update and the integer update be one critical section inside the kernel?

**Answers**

- Easy 5: S = −3.
- Hard 1, blocking form, S starting at 2: wait → 1, no sleep; wait → 0, no sleep; wait → −1, the third process sleeps; signal → 0, and that process is woken.
- If a binary mutex can be signalled up to 2, two later waits can both pass. Mutual exclusion is lost. A true binary semaphore must not be raised above 1.

### MCQs

**Q1. The P operation:**

A. Waits until it may decrement the semaphore  
B. Always increments the semaphore and never blocks  
C. Deletes the semaphore  
D. Swaps two semaphores  

**Answer:** A  

**Explanation:** P is wait. V is the increment.

**Q2. A binary semaphore used as a lock is initialised to:**

A. 1, if the critical section is free  
B. The number of processes in the system  
C. −1  
D. The size of the process control block  

**Answer:** A  

**Explanation:** 1 means one entry permit is available. The first wait consumes it.

**Q3. A counting semaphore initialised to 4 can represent:**

A. Four available units of a resource  
B. A binary lock that must stay at 4  
C. Four critical sections that may all run together on the same data without exclusion  
D. Peterson’s turn variable  

**Answer:** A  

**Explanation:** Each successful wait consumes one unit. Each signal returns one unit.

**Q4. In the blocking wait, the process sleeps when:**

A. S is less than 0 after the decrement  
B. S is greater than 10  
C. The remainder section starts  
D. Signal is executed by the same process before wait  

**Answer:** A  

**Explanation:** The decrement happens first. A negative result means the permit was not available.

**Q5. In that implementation, S = −2 means:**

A. Two processes are waiting on the semaphore  
B. Two units are free  
C. The semaphore is a binary semaphore with value 1  
D. Mutual exclusion has failed  

**Answer:** A  

**Explanation:** The magnitude of a negative value is the number of blocked processes.

**Q6. Signal on a blocking semaphore with S < 0:**

A. Increments S and wakes one waiting process  
B. Wakes every waiting process  
C. Sets S to 1 always  
D. Leaves S unchanged  

**Answer:** A  

**Explanation:** One signal releases one waiter. S moves toward zero.

**Q7. Deadlock with two semaphores occurs if:**

A. Each process holds one and waits for the other  
B. Both processes signal before they wait  
C. Both processes use only one semaphore  
D. The initial value is 1 and only one process runs  

**Answer:** A  

**Explanation:** Opposite acquisition orders create a circular wait.

**Q8. Application code may change the semaphore integer:**

A. Only by wait and signal  
B. By any assignment, because the integer is an ordinary counter  
C. By writing the process control block  
D. Only by test-and-set on a different variable, leaving S unchanged  

**Answer:** A  

**Explanation:** Direct assignment races with wait and signal and breaks the meaning of the value.

### Quick revision

#### Operations

```text
P / wait / down  :  take a permit, or sleep
V / signal / up  :  return a permit, or wake one waiter
```

#### Kinds

- Binary: 0 or 1. Mutex initialised to 1.
- Counting: initialised to the number of free units.

#### Blocking form

```text
wait:   S = S - 1;  if (S < 0) sleep
signal: S = S + 1;  if (S <= 0) wake one
```

A negative S counts the waiters. Take multiple semaphores in one fixed order.
