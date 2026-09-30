**Navigation:** [Unit index](00_Index.md) · [Previous: 3.5 Classical problems](05_Classical_Problems.md) · [Next: 3.7 Mutexes and spinlocks](07_Mutexes_and_Spinlocks.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 3 — Process Synchronization and Deadlocks

## Topic 3.6 — Monitors and Condition Variables

### Learning Outcomes

- Explain a monitor and why its procedures run under mutual exclusion.
- Use wait and signal on a condition variable.
- Distinguish Hoare semantics from Mesa semantics.
- Write the dining-philosophers monitor.

### Prerequisites

Critical section, semaphores, and the dining philosophers problem (Topics 3.1, 3.4, and 3.5).

### Introduction

Semaphores are powerful and easy to misuse. One forgotten signal, or two waits in the wrong order, is enough to deadlock a correct-looking program. A monitor packages the shared data and the procedures that touch it. Only one process executes inside the monitor at a time. Condition variables then let a process wait for a state, such as “a slot is free,” without coding that mutual exclusion by hand.

### Definition

**Definition:**  
A monitor is a program module containing shared data, the procedures that operate on that data, and an initialization block. Only one process executes inside the monitor at any instant.

**Definition:**  
A condition variable is a queue of processes waiting for a condition inside a monitor. It supports wait, which blocks the caller and lets another process enter the monitor, and signal, which wakes one waiting process.

A condition variable is not a semaphore. It has no integer value. Wait always blocks the caller. Signal does nothing if nobody is waiting. A semaphore signal is remembered as an increased count. A condition signal is not remembered.

### The shape of a monitor

```text
monitor Example
    shared variables

    procedure P1(...)
        ...
    procedure P2(...)
        ...

    initialization
        set the shared variables
end monitor
```

```text
Process A is inside the monitor
        |
        |  another call arrives
        v
Process B waits at the monitor entry
        |
        |  A leaves, or A waits on a condition
        v
One waiting process is allowed in
```

The mutual exclusion is automatic. The programmer does not write wait(mutex) at the start of every procedure. The programmer does still wait and signal when the shared state is not the state the procedure needs.

### Wait and signal

```text
condition x;

x.wait();      // block on x and leave the monitor
x.signal();    // if a process is waiting on x, wake one; otherwise do nothing
```

A process that calls wait is inside the monitor. Wait releases the monitor while the process sleeps. Otherwise no other process could enter and make the condition true.

| | Hoare signal | Mesa signal |
| --- | --- | --- |
| Who runs after signal | The woken process runs immediately. The signaler waits until that process leaves the monitor or waits again | The signaler continues. The woken process re-enters later |
| Condition when the waiter runs | Still the condition the signaler established | May already be false, because another process may have entered |
| Test in the waiter | `if` can be correct | `while` is required |
| Where it appears | Many textbook monitors | Java `wait` and `notify` |

The pattern that is correct for Mesa semantics, and harmless for Hoare semantics, is:

```text
while (the needed condition is false)
    condition.wait();
```

### Producer-consumer as a monitor

```text
monitor BoundedBuffer
    item buffer[n]
    int count = 0
    condition notfull, notempty

    procedure insert(item x)
        while (count == n)
            notfull.wait()
        add x to the buffer
        count = count + 1
        notempty.signal()

    procedure remove() returns item
        while (count == 0)
            notempty.wait()
        remove x from the buffer
        count = count - 1
        notfull.signal()
        return x
end monitor
```

There is no mutex semaphore in the source. The monitor supplies the exclusion. `notfull` and `notempty` are conditions, not counters. `count` is an ordinary integer because only one process is inside the monitor while it changes.

Under Mesa semantics, `if (count == n)` is not enough. A woken producer can find the buffer full again if another producer took the last free slot before this one re-entered. The `while` loop checks again.

### Dining philosophers as a monitor

A philosopher is allowed to eat only when neither neighbour is eating. The test runs inside the monitor, so two philosophers cannot both pass it and then share a chopstick.

```text
monitor DiningPhilosophers
    enum { THINKING, HUNGRY, EATING } state[5]
    condition self[5]

    procedure pickup(int i)
        state[i] = HUNGRY
        test(i)
        while (state[i] != EATING)
            self[i].wait()

    procedure putdown(int i)
        state[i] = THINKING
        test((i + 4) mod 5)
        test((i + 1) mod 5)

    procedure test(int i)
        if (state[(i + 4) mod 5] != EATING
            and state[i] == HUNGRY
            and state[(i + 1) mod 5] != EATING)
            state[i] = EATING
            self[i].signal()
end monitor
```

Neighbours of philosopher i are (i + 4) mod 5 and (i + 1) mod 5.

`pickup` marks the philosopher hungry and calls `test`. If both neighbours are not eating, the state becomes EATING and the following while does not wait. If a neighbour is eating, the philosopher waits on `self[i]`. `putdown` asks both neighbours to try again. A neighbour becomes eating only when it is hungry and its other neighbour is also free.

This removes the circular deadlock of the left-then-right chopstick program. The philosopher never holds one chopstick while waiting for the other. The test grants both sides or neither. It does not by itself guarantee fairness: the two neighbours can alternate so that a philosopher remains hungry at every test.

The while loop matches Mesa semantics. Under Hoare semantics, an `if` is enough, because a philosopher signalled by `test` runs immediately, before a neighbour can slip in. Writing `while` is the version that stays correct for both.

### Comparison with semaphores

| Parameter | Semaphore | Monitor |
| --- | --- | --- |
| Mutual exclusion | Written with wait and signal | Provided for every procedure |
| Waiting for a state | Often a counting semaphore | A condition variable; the state sits in ordinary variables |
| Signal when nobody is waiting | The count increases and the signal is remembered | Nothing is remembered |
| Typical error | Wrong order, or a missing signal | Using `if` under Mesa semantics |
| Shared data | Wherever the variables are visible | Inside the monitor |

### Advantages and limitations

| Advantages | Limitations |
| --- | --- |
| Mutual exclusion is not a manual wait(mutex) | The conditions must still be written correctly |
| Shared data and its operations stay in one module | Hoare and Mesa signals are not interchangeable |
| A waiting process releases the monitor, so another process can change the state | Underneath, the implementation still uses a lock and queues |
| The philosopher test grants both sides or neither | The basic monitor does not guarantee that a particular hungry philosopher eventually eats |

### Common mistakes

- Treating a condition signal as a stored count. If nobody is waiting, the signal disappears.
- Calling wait without releasing the monitor. Then the process that could make the condition true cannot enter.
- Using `if` in a Java-style monitor. The recheck belongs in a `while`.
- Describing the dining monitor as “pick up the left chopstick, then the right.” The eating state is granted only when both neighbours are free.

### Important exam points

- One process inside the monitor at a time.
- `wait` blocks and leaves the monitor. `signal` wakes one waiter, or does nothing.
- Hoare: the waiter runs at once. Mesa: the signaler continues, so use `while`.
- Buffer monitor: `count`, `notfull`, `notempty`.
- Dining monitor: `state[5]`, `self[5]`, and `test` of both neighbours from `putdown`.

### University exam questions

#### 2-mark questions

1. Define a monitor.
2. Define a condition variable.
3. What does a condition signal do when no process is waiting?
4. Why does Mesa-style code use `while`?

#### 4/5-mark questions

1. Explain wait and signal on a condition variable.
2. Write the bounded-buffer monitor.
3. Differentiate Hoare and Mesa semantics.

#### 8/10-mark questions

1. Explain monitors and condition variables. Write a producer-consumer monitor and justify the loops.
2. Write the dining-philosophers monitor and explain why it does not deadlock.

### Practice problems

#### Easy

1. How many processes may run inside one monitor at the same time?
2. Does a condition signal increment a stored integer?
3. What does wait do to the monitor lock?
4. Name the two signal semantics.
5. Which three states does the dining `test` look at?

#### Medium

1. Explain why wait must release the monitor.
2. Why can a woken Mesa producer observe a full buffer even though a signal means “a slot was free”?
3. What does putdown(i) ask the two neighbours to do?
4. Compare a semaphore signal on an idle semaphore with a condition signal on an empty queue.
5. Why is there no mutex semaphore written in the buffer monitor?

#### Hard

1. Philosopher 1 is eating. Philosophers 0 and 2 call pickup. What are their states?
2. Philosopher 1 then calls putdown, and philosophers 3 and 4 are thinking. Can 0 and 2 both become eating?
3. Describe a starvation pattern this monitor allows.
4. Why may `count` be an ordinary integer inside the buffer monitor?
5. A Hoare signal is executed before the shared state is updated. What does the woken process see?

**Answers**

- While 1 is eating, `test(0)` and `test(2)` fail because 1 is a neighbour of both. Philosophers 0 and 2 stay HUNGRY and wait.
- `putdown(1)` calls `test(0)` and `test(2)`. The other neighbour of 0 is 4, and the other neighbour of 2 is 3. Both are thinking, so both tests succeed. Philosophers 0 and 2 do not share a chopstick.
- Neighbours of a hungry philosopher can take turns eating, so every test of the hungry philosopher finds one neighbour eating. There is no circular wait, but that philosopher need not eat.
- Only one process is inside the monitor, so two updates of `count` cannot interleave.
- The woken process runs before the signaler continues, so it still sees the old state. Update the state, then signal.

### MCQs

**Q1. A monitor guarantees that:**

A. Only one process executes inside it at a time  
B. Every condition signal is stored as an integer  
C. Deadlock is impossible in the whole operating system  
D. Philosophers pick up the left chopstick first  

**Answer:** A  

**Explanation:** Mutual exclusion of the monitor procedures is part of the definition.

**Q2. A condition variable:**

A. Has wait and signal, and no remembered count  
B. Is a counting semaphore initialised to n  
C. Allows several processes to execute the monitor together  
D. Replaces the process table  

**Answer:** A  

**Explanation:** Signal wakes a waiter if one exists. It does not increment a value for a future wait.

**Q3. wait on a condition:**

A. Blocks the caller and releases the monitor  
B. Spins while holding the monitor forever  
C. Deletes the shared data  
D. Is identical to semaphore signal  

**Answer:** A  

**Explanation:** The monitor must be released so another process can establish the condition.

**Q4. Under Mesa semantics the waiter must:**

A. Recheck the condition in a while loop  
B. Assume the condition is true and never look at it  
C. Call signal instead of wait  
D. Disable interrupts on every core  

**Answer:** A  

**Explanation:** The signaler continues, and other processes may enter before the waiter runs.

**Q5. Under Hoare semantics, after a successful signal:**

A. The woken process runs before the signaler continues in the monitor  
B. The signal is ignored  
C. Both processes execute the monitor together  
D. The semaphore value becomes n  

**Answer:** A  

**Explanation:** The waiter runs immediately, which is why an `if` can be correct in that convention.

**Q6. In the buffer monitor, a producer waits on notfull when:**

A. count equals n  
B. count equals 0  
C. a reader is active  
D. mutex is a condition  

**Answer:** A  

**Explanation:** n means the buffer is full. An empty buffer has count 0, and that is when the consumer waits.

**Q7. In the dining monitor, test(i) sets state[i] to EATING only when:**

A. i is hungry and neither neighbour is eating  
B. i is thinking  
C. both neighbours are eating  
D. i already holds one chopstick  

**Answer:** A  

**Explanation:** Both neighbouring states are examined before the eating state is granted.

**Q8. putdown(i) calls test on:**

A. Both neighbours of i  
B. Only philosopher 0  
C. Every philosopher, including those who are thinking and were not waiting  
D. The same philosopher i, who has just been set to THINKING and so will not start eating again inside that call  

**Answer:** A  

**Explanation:** The two neighbours are (i + 4) mod 5 and (i + 1) mod 5. Philosopher i is THINKING, so test(i) would not make i eat; the code tests the neighbours instead.

### Quick revision

#### Monitor

Shared data plus procedures. One process inside at a time.

#### Condition

`wait` always sleeps and leaves the monitor. `signal` wakes one process, or nobody. It does not store a count.

#### Semantics

Hoare: waiter runs at once. Mesa: signaler continues, so write `while`.

#### Dining test

Hungry, and both neighbours not eating, becomes eating. putdown retests both neighbours.
