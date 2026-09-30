**Navigation:** [Unit index](00_Index.md) · [Previous: 3.1 Critical section and race condition](01_Critical_Section_and_Race_Condition.md) · [Next: 3.3 Peterson’s solution](03_Petersons_Solution.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 3 — Process Synchronization and Deadlocks

## Topic 3.2 — Test-and-Set and Compare-and-Swap

### Learning Outcomes

- Explain why the lock operations must be atomic.
- Write the test-and-set instruction and the lock built from it.
- Write compare-and-swap and the lock built from it.
- State that the simple busy-wait form does not provide bounded waiting, and how the waiting-array form does.

### Prerequisites

Critical section, mutual exclusion, and bounded waiting (Topic 3.1).

### Introduction

A lock variable seems easy: if `lock` is false the process sets it to true and enters. Two processes can both read false before either writes true, and both enter. The read and the write have to be one hardware step. Test-and-set and compare-and-swap are those steps. They are the usual foundation of spinlocks.

### Definition

**Definition:**  
Test-and-set is an atomic instruction that returns the old value of a memory word and sets that word to true.

**Definition:**  
Compare-and-swap is an atomic instruction that sets a memory word to a new value only if the word still contains an expected value, and returns the word’s old value.

Atomic means the processor performs the read and the update as one indivisible memory operation. No other core observes the word between the read and the write.

### Test-and-set

```text
boolean test_and_set(boolean *target) {
    boolean rv = *target;
    *target = true;
    return rv;
}
```

If the lock was free, the call returns false and leaves the lock true. The caller may enter. If the lock was already true, the call returns true and the lock stays true. The caller must wait.

```text
boolean lock = false;

repeat
    while (test_and_set(&lock))
        ;                      // busy wait
    critical section
    lock = false;
    remainder section
until false
```

```text
Algorithm: acquire with test-and-set

Step 1: Atomically read lock and set it to true.
Step 2: If the old value was false, this process now owns the lock. Enter.
Step 3: If the old value was true, another process owns it. Repeat Step 1.
Step 4: After the critical section, store false into lock.
```

**Mutual exclusion.**  
Suppose two processes try to enter. The hardware allows only one test-and-set to see the old value false. The other sees true and waits. The winner stores false only after leaving.

**Progress.**  
When the lock is false, a process that executes test-and-set obtains it. The decision is not postponed.

**Bounded waiting.**  
The simple loop does not provide it. A process can leave, run its remainder, and acquire the lock again before a process that has been spinning is scheduled. There is no queue and no turn number.

### Bounded waiting with test-and-set

The following form, for n processes, gives each waiting process a boolean and hands the lock to the next waiter in order of process index.

```text
boolean lock = false;
boolean waiting[n] = { false, ..., false };

repeat
    waiting[i] = true;
    key = true;
    while (waiting[i] and key)
        key = test_and_set(&lock);
    waiting[i] = false;

    critical section

    j = (i + 1) mod n;
    while (j != i and waiting[j] == false)
        j = (j + 1) mod n;
    if (j == i)
        lock = false;          // nobody else is waiting
    else
        waiting[j] = false;    // let process j enter
    remainder section
until false
```

After process i leaves, it scans the other processes once, in index order, and releases the first one that is waiting. Each of the other n − 1 processes can enter at most once before i’s next turn comes around. That is the bound.

### Compare-and-swap

```text
int compare_and_swap(int *value, int expected, int new_value) {
    int temp = *value;
    if (*value == expected)
        *value = new_value;
    return temp;
}
```

The whole function is one atomic instruction. A lock value of 0 means free and 1 means held.

```text
int lock = 0;

repeat
    while (compare_and_swap(&lock, 0, 1) != 0)
        ;                      // busy wait: old value was not 0
    critical section
    lock = 0;
    remainder section
until false
```

If the lock is 0, compare-and-swap stores 1 and returns 0. The while test fails and the process enters. If the lock is 1, the word is left unchanged and the returned value is 1, so the process spins.

Compare-and-swap is more general than test-and-set. It can update a pointer or a counter only when the value is still the one the process last observed. That is the operation used by lock-free structures in Topic 3.8. Test-and-set only forces a boolean to true.

### Comparison

| Parameter | Test-and-set | Compare-and-swap |
| --- | --- | --- |
| What it does | Returns the old boolean and sets it to true | Writes a new value only if the old value equals the expected value |
| Typical lock | `while (test_and_set(&lock));` then `lock = false` | `while (compare_and_swap(&lock, 0, 1) != 0);` then `lock = 0` |
| Mutual exclusion | Yes, because the instruction is atomic | Yes, for the same reason |
| Bounded waiting in the simple loop | No | No |
| Other use | Spinlock | Spinlock, and also lock-free updates of pointers and counters |

Both waste CPU while they spin. That is acceptable for a very short critical section on a multicore machine, because the alternative, going to sleep, may cost more than the wait. A long critical section should block the waiter instead. Mutexes and semaphores do that.

### Advantages and limitations

| Advantages | Limitations |
| --- | --- |
| Mutual exclusion is enforced by hardware, not by a careful interleaving of ordinary loads and stores | The simple loops do not guarantee bounded waiting |
| They work on multiprocessors | Busy waiting occupies a core |
| Compare-and-swap can build more than a boolean lock | The programmer must still put the instruction in the entry section; the hardware does not find the critical section |
| The bounded-waiting test-and-set form removes the starvation of the simple loop | That form is longer and still spins |

### Common mistakes

- Writing the lock test and the lock store as two ordinary instructions and calling them test-and-set. The atomicity is the whole point.
- Saying compare-and-swap always stores the new value. It stores only when the current value equals the expected value.
- Claiming the simple spin loop has bounded waiting. It has mutual exclusion and progress, not a bound.
- Forgetting to set the lock back to false or 0 in the exit section. The next process then spins forever.

### Important exam points

- Both instructions are atomic.
- Test-and-set returns the previous value and leaves true in memory.
- Compare-and-swap returns the previous value and writes only on a match. The lock loop continues while the returned value is not 0.
- Simple spinlocks: mutual exclusion and progress, no bounded waiting, busy wait.
- The waiting-array algorithm is the version that adds bounded waiting.

### University exam questions

#### 2-mark questions

1. Define test-and-set.
2. Define compare-and-swap.
3. Why must a lock instruction be atomic?
4. Does the simple test-and-set loop guarantee bounded waiting?

#### 4/5-mark questions

1. Write test-and-set and show how it implements mutual exclusion.
2. Write compare-and-swap and the spin loop that uses it as a lock.
3. Differentiate test-and-set and compare-and-swap.

#### 8/10-mark questions

1. Explain test-and-set and compare-and-swap. Show the entry and exit code, and discuss busy waiting and bounded waiting.
2. Explain the bounded-waiting solution based on test-and-set. State how the next waiting process is chosen.

### Practice problems

#### Easy

1. What does test-and-set return if the lock was already true?
2. What value of a compare-and-swap lock means free in the loop above?
3. Who sets the lock to false: the process entering or the process leaving?
4. Name one requirement the simple spin loop does not satisfy.
5. Why is a spin suitable only for a short critical section?

#### Medium

1. Two processes execute an ordinary “if lock is false, set it true.” Show how both enter.
2. Repeat the situation with test-and-set and show that only one enters.
3. compare-and-swap(&lock, 0, 1) is executed when lock is 1. What is returned, and does lock change?
4. The same call is executed when lock is 0. What is returned, and what is lock afterwards?
5. Explain one line of the bounded-waiting solution: the scan that starts at (i + 1) mod n.

#### Hard

1. Explain why a process in the bounded-waiting solution can leave the while loop even if test-and-set keeps returning true.
2. How many other entries can occur, at most, before process i enters again after it has set waiting[i]?
3. Why can compare-and-swap update a shared pointer while test-and-set cannot express “store this pointer only if the head is still the head I saw”?
4. A process forgets `lock = false`. Describe the progress failure.
5. Compare the CPU cost of spinning for 1 microsecond with the cost of a context switch that takes longer than the critical section.

### MCQs

**Q1. Test-and-set atomically:**

A. Returns the old value and sets the word to true  
B. Sets the word to false and returns true always  
C. Swaps two process control blocks  
D. Waits until the critical section has been entered twice  

**Answer:** A  

**Explanation:** The returned value tells the caller whether the lock was already held. The word is left true.

**Q2. In the simple test-and-set lock, a process enters when the instruction returns:**

A. False, meaning the lock was free  
B. True, meaning the lock was free  
C. The process identifier  
D. The length of the critical section  

**Answer:** A  

**Explanation:** A false old value means this call was the one that took the free lock.

**Q3. Compare-and-swap writes the new value when:**

A. The memory word equals the expected value  
B. The memory word differs from the expected value  
C. Any process is in its remainder section  
D. The lock is already 1, in the convention used for this lock  

**Answer:** A  

**Explanation:** The compare succeeds only on a match. Otherwise the word is unchanged.

**Q4. The call compare_and_swap(&lock, 0, 1) while lock is 0 returns:**

A. 0, and lock becomes 1  
B. 1, and lock stays 0  
C. 0, and lock stays 0  
D. The number of waiting processes  

**Answer:** A  

**Explanation:** The old value was 0, so the swap to 1 happens and 0 is returned. The while loop then stops.

**Q5. The simple compare-and-swap spin loop does not guarantee:**

A. Bounded waiting  
B. Mutual exclusion  
C. An atomic update  
D. That a free lock can be taken  

**Answer:** A  

**Explanation:** There is no record of who has been waiting the longest. A fast process can reacquire the lock indefinitely.

**Q6. Busy waiting means the waiting process:**

A. Repeatedly executes the lock instruction until the lock is free  
B. Is placed on a sleep queue and uses no CPU  
C. Leaves the system  
D. Enters the critical section together with the owner  

**Answer:** A  

**Explanation:** The while loop consumes CPU. A blocking lock would put the process to sleep instead.

**Q7. The waiting-array form of test-and-set adds:**

A. Bounded waiting, by passing permission to the next waiting index  
B. A second critical section that several processes may share  
C. Non-atomic test-and-set  
D. The removal of mutual exclusion  

**Answer:** A  

**Explanation:** After leaving, a process clears waiting[j] for the next waiter, or clears the lock if there is no waiter.

**Q8. Compare-and-swap is used for more than boolean locks because:**

A. It can update a word only if that word still has the value the algorithm expects  
B. It disables every core  
C. It replaces the process scheduler  
D. It makes bounded waiting automatic for every data structure  

**Answer:** A  

**Explanation:** The expected-value test is what lock-free algorithms use to detect that another process changed the word.

### Quick revision

#### Test-and-set

Return the old boolean, set the word to true. Enter if the old value was false. Exit by storing false.

#### Compare-and-swap

If `*value == expected`, store `new_value`. Always return the old value. Lock loop: `while (compare_and_swap(&lock, 0, 1) != 0)`.

#### Properties of the simple loops

Mutual exclusion: yes. Progress: yes. Bounded waiting: no. Cost: busy waiting.

#### Bounded waiting

The waiting[i] algorithm passes the turn to the next process that is waiting, in index order.
