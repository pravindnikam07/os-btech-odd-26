**Navigation:** [Unit index](00_Index.md) · [Previous: 3.4 Semaphores](04_Semaphores.md) · [Next: 3.6 Monitors and condition variables](06_Monitors_and_Condition_Variables.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 3 — Process Synchronization and Deadlocks

## Topic 3.5 — Classical Synchronization Problems

### Learning Outcomes

- Implement the bounded-buffer problem with semaphores.
- Explain why the order of wait(mutex) and wait(empty) matters.
- Write the readers-writers solution and state which group can starve.
- Explain the dining philosophers problem and one deadlock-free way to pick up the chopsticks.

### Prerequisites

Binary and counting semaphores, and the blocking behaviour of wait (Topic 3.4).

### Introduction

Three problems are used to test whether a synchronization idea is actually usable. The producer and the consumer share a buffer. Readers and writers share a table. Philosophers share chopsticks. Each problem has a tempting solution that deadlocks or that lets one group wait forever. The solutions below are the standard semaphore solutions.

### Definition

**Definition:**  
The bounded-buffer (producer-consumer) problem is the problem of moving items through a buffer of n slots, with producers that must wait when the buffer is full and consumers that must wait when it is empty, without corrupting the buffer index.

**Definition:**  
The readers-writers problem is the problem of allowing several readers of a shared object at the same time, while giving a writer exclusive access.

**Definition:**  
The dining philosophers problem is the problem of allocating two shared resources to each of several processes so that no process waits forever in a circle and neighbouring processes do not use the same resource together.

### Producer-consumer

Shared data:

```text
semaphore mutex = 1;     // protects the buffer indices
semaphore empty = n;     // number of free slots
semaphore full  = 0;     // number of filled slots
```

```text
Producer                         Consumer

repeat                           repeat
    produce an item                  wait(full);
    wait(empty);                     wait(mutex);
    wait(mutex);                     remove an item
    insert the item                  signal(mutex);
    signal(mutex);                   signal(empty);
    signal(full);                    consume the item
until false                      until false
```

`empty` stops the producer when every slot is full. `full` stops the consumer when every slot is empty. `mutex` stops them from changing `in` and `out`, or the slot contents, at the same time. The counting semaphores do not replace the mutex. Two producers can both pass `wait(empty)` when two slots are free, and they then need mutual exclusion to update the shared index.

**The order is part of the solution.**  
The producer waits for a free slot before it takes the mutex. Consider the reversed producer:

```text
wait(mutex);
wait(empty);
```

The buffer is full, so this producer holds `mutex` and sleeps on `empty`. The consumer needs `mutex` before it can remove an item and signal `empty`. The consumer sleeps on `mutex`. The producer sleeps on `empty`. Neither can signal the semaphore the other needs. That is a deadlock. The published order, resource semaphore first and mutex second, is the one that avoids it. The consumer similarly waits on `full` before `mutex`.

```text
n = 2. Start: empty = 2, full = 0, mutex = 1.

Producer inserts once:
    empty 2 → 1, mutex taken and released, full 0 → 1

Consumer removes once:
    full 1 → 0, mutex taken and released, empty 1 → 2

Two further inserts with no consumer:
    empty becomes 0 and full becomes 2
    the next producer waits on empty
```

### Readers-writers

The first readers-writers solution gives readers priority. A reader may enter if no writer is writing, even if a writer is waiting. Several readers may be inside together. A writer is alone.

```text
semaphore mutex = 1;     // protects readcount
semaphore wrt   = 1;     // exclusive access for a writer, or for the first/last reader
int readcount = 0;
```

```text
Writer                           Reader

wait(wrt);                       wait(mutex);
write                            readcount = readcount + 1;
signal(wrt);                     if (readcount == 1)
                                     wait(wrt);      // first reader locks out writers
                                 signal(mutex);

                                 read

                                 wait(mutex);
                                 readcount = readcount - 1;
                                 if (readcount == 0)
                                     signal(wrt);    // last reader lets writers in
                                 signal(mutex);
```

`readcount` is shared by readers, so it has its own mutex. Only the first reader waits on `wrt`. Later readers see a non-zero count and enter. The last reader signals `wrt`. A writer always waits on `wrt`, so it cannot overlap another writer or any reader.

Writers can starve. While one reader keeps arriving before the previous readers have all left, `readcount` never stays at 0 and `wrt` stays held by the reader side. A waiting writer never starts. A second version of the problem reverses the preference and can starve readers. The examination solution above is the readers-preference version. Naming the starvation is part of a complete answer.

### Dining philosophers

Five philosophers sit at a round table. There are five chopsticks, one between each pair. A philosopher thinks, then picks up the two chopsticks beside the plate, eats, and puts them down. Neighbours cannot eat at the same time because they share a chopstick.

```text
                 P0
             C0      C1
          P4            P1
             C4      C2
                 P2
                 C3
                 P3
```

Each chopstick is a semaphore initialised to 1.

```text
Philosopher i, the unsafe version

wait(chopstick[i]);                 // left
wait(chopstick[(i + 1) mod 5]);     // right
eat
signal(chopstick[i]);
signal(chopstick[(i + 1) mod 5]);
think
```

If every philosopher picks up the left chopstick before anyone picks up the right, each holds one and waits for the other. The waits form a circle. This is the textbook deadlock.

**A correct asymmetry.**  
Odd-numbered philosophers pick up the left chopstick first. Even-numbered philosophers pick up the right chopstick first.

```text
if i is even:
    first = (i + 1) mod 5
    second = i
else:
    first = i
    second = (i + 1) mod 5

wait(chopstick[first])
wait(chopstick[second])
eat
signal(chopstick[second])
signal(chopstick[first])
```

Two neighbours no longer take their shared chopstick in the same role at the same moment. The circular wait is broken. Another correct restriction is to allow at most four philosophers to sit down, so at least one chopstick remains free and the circle cannot close. A third is to pick up both chopsticks inside a mutex, so the two waits are not separated by another philosopher’s wait. The asymmetry is the solution to write out in full.

### Comparison

| Problem | Shared object | Semaphores | Failure the solution prevents | Remaining weakness |
| --- | --- | --- | --- | --- |
| Producer-consumer | Buffer of n slots | mutex = 1, empty = n, full = 0 | Overlap of buffer updates, and waiting on a full or empty buffer | Deadlock if mutex is taken before the slot semaphore |
| Readers-writers | A table or file | mutex = 1, wrt = 1, plus readcount | A writer overlapping a reader, or two writers overlapping | Waiting writers can starve |
| Dining philosophers | Five chopsticks | one binary semaphore per chopstick | Two neighbours eating with one chopstick; deadlock, if the pick-up order is asymmetric | A philosopher can still starve if the neighbours are always faster, unless a further fairness rule is added |

### Common mistakes

- Initialising `full` to n and `empty` to 0. At the start the buffer is empty, so `full` is 0 and `empty` is n.
- Treating `empty` as a mutex. It counts slots. The mutex is separate.
- Letting every reader wait on `wrt`. Only the first reader does. If every reader waited, readers could not share the table.
- Forgetting the last reader’s signal. Writers then wait forever after the first reading period.
- Giving every philosopher the same left-then-right order and calling it deadlock-free.

### Important exam points

- Producer: `wait(empty)`, `wait(mutex)`, insert, `signal(mutex)`, `signal(full)`.
- Consumer: `wait(full)`, `wait(mutex)`, remove, `signal(mutex)`, `signal(empty)`.
- Reversing mutex and empty deadlocks when the buffer is full.
- First reader locks `wrt`; last reader unlocks it; writers can starve.
- Five chopsticks, left-then-right deadlock, and one stated cure: different order for even and odd, or at most four seated.

### University exam questions

#### 2-mark questions

1. What are the initial values of mutex, empty, and full for a buffer of n slots?
2. Why can writers starve in the first readers-writers solution?
3. How many chopsticks does a philosopher need?
4. State one deadlock-free chopstick policy.

#### 4/5-mark questions

1. Write the producer and consumer using semaphores.
2. Show the deadlock if the producer takes mutex before empty.
3. Write the reader and writer code and explain readcount.

#### 8/10-mark questions

1. Explain the bounded-buffer problem and its semaphore solution, including the reason for the order of the waits.
2. Explain the dining philosophers problem, the deadlock in the naive solution, and one correct solution.
3. Explain the readers-writers problem. Write the semaphore solution and discuss starvation.

### Practice problems

#### Easy

1. A buffer has 8 slots and is empty. Give empty and full.
2. Which semaphore does the first reader acquire that later readers, during the same reading period, do not acquire?
3. How many philosophers, each holding one chopstick, produce the circular deadlock?
4. Which signal does the producer execute last?
5. Does the consumer signal empty or full after a removal?

#### Medium

1. The buffer is full and the producer has correctly waited on empty without holding mutex. What does the consumer do that wakes the producer?
2. readcount goes from 0 to 1 to 2 to 1 to 0. How many times is wait(wrt) executed by readers in that period, and how many times is signal(wrt)?
3. Explain the even/odd chopstick order for philosopher 0 and philosopher 1.
4. Why is mutex still required when empty and full already exist?
5. Describe writer starvation with a steady stream of readers.

#### Hard

1. Start with n = 1, empty = 1, full = 0. Interleave a producer and a consumer so that the consumer waits, then the producer runs, then the consumer continues. Give the semaphore values after each wait and signal.
2. Show the four waits of the reversed producer and a consumer when the buffer is full, and name the semaphore each is blocked on.
3. A solution lets at most four philosophers call pickup. Why can the five-chopstick circle not close?
4. Two writers and no readers are active. Can both be inside the write section of the solution above?
5. Why does protecting readcount with wrt, instead of with a separate mutex, reduce concurrent reading?

**Answers**

- Empty buffer of 8: empty = 8, full = 0, mutex = 1.
- Readers call wait(wrt) once in that period, when the count becomes 1, and signal(wrt) once, when the count returns to 0.
- For n = 1, one correct interleaving: consumer wait(full) blocks, full is 0. Producer wait(empty) makes empty 0, wait/signal mutex, signal(full) makes full 1 and wakes the consumer. Consumer then takes mutex, removes, signals mutex, signals empty. Empty returns to 1 and full returns to 0.
- Reversed case: producer holds mutex and waits on empty. Consumer waits on mutex. Producer is blocked on empty; consumer is blocked on mutex.
- Two writers cannot both pass wait(wrt). The second waits until the first signals.
- If readers used wrt for the whole read, the second reader would wait for wrt and readers would no longer overlap. The separate mutex is held only while the counter changes.

### MCQs

**Q1. For an empty buffer of n slots, the initial values are:**

A. mutex = 1, empty = n, full = 0  
B. mutex = 0, empty = 0, full = n  
C. mutex = n, empty = 1, full = 1  
D. mutex = 1, empty = 0, full = n  

**Answer:** A  

**Explanation:** All slots are free, no slot is full, and the buffer structure is unlocked.

**Q2. The producer must call wait(empty) before wait(mutex) because:**

A. Holding the mutex while waiting for a slot can deadlock the consumer  
B. empty is a binary semaphore and mutex is not  
C. full must stay at n  
D. the consumer never uses mutex  

**Answer:** A  

**Explanation:** The consumer needs the mutex in order to free a slot. It must not be stuck behind a producer that already holds that mutex.

**Q3. In the readers-preference solution, wrt is acquired by a reader when:**

A. That reader is the first of the current group  
B. Every reader acquires it before reading  
C. readcount is already greater than 1  
D. A writer is inside the critical section and the reader should join the writer  

**Answer:** A  

**Explanation:** The first reader locks writers out. Further readers only update the counter under mutex.

**Q4. Writers can starve in that solution because:**

A. New readers can keep readcount above 0  
B. The writer never calls wait  
C. mutex is initialised to 0  
D. Chopsticks are required for reading  

**Answer:** A  

**Explanation:** The writer needs readcount to fall to 0 so that wrt is signalled. A continuous flow of readers prevents that.

**Q5. The naive dining solution deadlocks when:**

A. Each philosopher holds the left chopstick and waits for the right  
B. Only one philosopher sits down  
C. All chopsticks are signalled before any wait  
D. Philosophers only think  

**Answer:** A  

**Explanation:** The five waits form a cycle. No philosopher can get a second chopstick.

**Q6. An asymmetric cure is:**

A. Even philosophers pick up the right chopstick first and odd philosophers pick up the left first  
B. Every philosopher picks up the left chopstick first  
C. All five pick up both chopsticks without any semaphore  
D. Philosophers share one chopstick among all five plates  

**Answer:** A  

**Explanation:** Neighbours no longer acquire the shared chopstick in the same order, so the circle is broken.

**Q7. full is signalled by:**

A. The producer, after inserting an item  
B. The consumer, after inserting an item  
C. The first reader  
D. A philosopher who starts thinking  

**Answer:** A  

**Explanation:** The producer has created one full slot. The consumer waits on that semaphore and later signals empty.

**Q8. readcount is protected by mutex because:**

A. Several readers update the same counter  
B. The writer uses readcount as a chopstick  
C. mutex replaces wrt for the writer  
D. The counter is private to one reader  

**Answer:** A  

**Explanation:** The increment and the test against 1 must themselves be a critical section. That section is short; the long shared access is protected by wrt.

### Quick revision

#### Producer-consumer

```text
mutex = 1, empty = n, full = 0
producer: wait(empty), wait(mutex), insert, signal(mutex), signal(full)
consumer: wait(full),  wait(mutex), remove, signal(mutex), signal(empty)
```

Mutex before empty, when the buffer is full, deadlocks.

#### Readers-writers

First reader wait(wrt). Last reader signal(wrt). Writers can starve.

#### Dining philosophers

Left-then-right for all five deadlocks. Break the circle by an even/odd order, or by seating at most four.
