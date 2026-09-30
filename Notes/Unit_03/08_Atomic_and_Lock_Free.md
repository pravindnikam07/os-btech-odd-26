**Navigation:** [Unit index](00_Index.md) · [Previous: 3.7 Mutexes and spinlocks](07_Mutexes_and_Spinlocks.md) · [Next: 3.9 Deadlock and the four necessary conditions](09_Deadlock_and_Necessary_Conditions.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 3 — Process Synchronization and Deadlocks

## Topic 3.8 — Atomic Operations and Lock-Free Algorithms

### Learning Outcomes

- Define an atomic operation.
- Build a shared counter with compare-and-swap.
- Explain a lock-free stack push and pop.
- State the ABA problem.

### Prerequisites

Compare-and-swap (Topic 3.2) and the cost of a mutex (Topic 3.7).

### Introduction

A lock keeps other threads out of a critical section. It also makes those threads wait, and a thread that is preempted while holding the lock delays every waiter. Some shared structures can be updated by a single atomic instruction. If the instruction succeeds, the update is visible as a whole. If it fails, another thread changed the same word, and this thread retries. No thread holds a lock. An algorithm is lock-free when some thread completes an operation in a finite number of steps even if other threads are delayed.

### Definition

**Definition:**  
An atomic operation appears instantaneous to other threads. They see the state before the operation or the state after it, never a mixture.

**Definition:**  
A lock-free algorithm does not use a mutual-exclusion lock. If several threads execute it, at least one of them completes its operation after a finite number of its own steps, regardless of how slowly the others run.

Lock-free is stronger than “usually no lock.” A spinlock is not lock-free: if the owner stops running, every spinner stops making progress. In a lock-free structure, a delayed thread cannot freeze the others. They may fail a compare-and-swap and retry, and one of the retries succeeds.

Wait-free is a stronger promise again: every thread finishes in a finite number of its own steps, not merely some thread. The stack below is lock-free. It is not claimed to be wait-free, because one unlucky thread can fail its compare-and-swap repeatedly while others succeed.

### An atomic counter

The lost update of Topic 3.1 was a load, an add, and a store. Compare-and-swap folds the check and the store together.

```text
add_one(int *count):
    repeat
        old = *count
        new = old + 1
    until compare_and_swap(count, old, new) == old
```

If `count` is still `old`, this thread’s new value is stored and the returned old value matches. If another thread stored a different value, the returned value is not `old`, the loop reads again, and the addition is based on the new base. No increment is lost. There is no mutex.

```text
count is 5.
Thread A reads old = 5, new = 6.
Thread B reads old = 5, new = 6.
A’s compare-and-swap succeeds. count is 6.
B’s compare-and-swap fails because count is no longer 5.
B reads old = 6, new = 7, and succeeds.
count is 7.
```

### A lock-free stack

The stack is a linked list. `head` points to the first node.

```text
push(node):
    repeat
        old = head
        node.next = old
    until compare_and_swap(&head, old, node) == old

pop():
    repeat
        old = head
        if old == null
            return null          // stack was empty
        next = old.next
    until compare_and_swap(&head, old, next) == old
    return old
```

Push publishes `node` as the new head only if `head` is still the pointer this thread stored in `node.next`. Otherwise someone else pushed or popped, and the thread samples `head` again.

Pop swings `head` to the second node only if `head` is still the node it read. Two pops cannot both claim the same node: only one compare-and-swap finds the expected pointer.

```text
head → N1 → N2 → null

Thread A reads old = N1, next = N2.
Thread B pushes N3. head → N3 → N1 → N2.
A’s compare-and-swap expects N1 and fails.
A retries, reads N3, and pops N3 only if head is still N3.
```

### The ABA problem

Compare-and-swap asks “is the word still the bit pattern I saw?” It does not ask “did the word stay unchanged the whole time?”

```text
head is N1.
Thread A reads old = N1 and next = N2, then pauses.
Thread B pops N1 and later pushes N1 again.
head is N1 once more, but the node after N1 may now be different.
Thread A’s compare-and-swap sees N1, succeeds, and writes the stale next pointer.
```

The bit pattern came back. The stack’s meaning did not. This is the ABA problem: the value went from A to B and back to A, and the waiting compare-and-swap cannot tell.

Practical implementations add a version count beside the pointer, so the word changes even if the same node is reused, or they do not reuse a node while a popped pointer might still be compared. The examination point is the definition of ABA and the reason a bare pointer comparison is not a full proof.

### Comparison

| Parameter | Mutex around the stack | Lock-free stack |
| --- | --- | --- |
| Waiting | Threads sleep or spin while the lock is held | A conflicting thread retries |
| If the owner is preempted | Every other thread waits | Other threads can still push and pop |
| Progress | Not lock-free | Some thread completes an operation |
| Correctness hazard | Forgetting unlock, or deadlock with a second lock | ABA, and freeing a node another thread still reads |
| Hardware used | A lock, itself built on an atomic instruction | Compare-and-swap on the head |

### Advantages and limitations

| Advantages | Limitations |
| --- | --- |
| No lock is held across the operation | The retry loop can starve one thread; that is why it is not automatically wait-free |
| A stalled thread does not block the others | ABA can make a successful compare-and-swap wrong |
| A counter or a stack needs no mutex variable | Reclaiming memory safely is harder than the push and pop code |
| The hardware primitive is the same compare-and-swap already used for locks | Not every critical section is one word; a large update may still need a lock |

### Common mistakes

- Calling a spinlock lock-free. The spinners make no progress while the holder is stopped.
- Forgetting the retry. One compare-and-swap without a loop loses the update when the expected value has changed.
- Testing only the final bit pattern and ignoring ABA.
- Freeing a popped node immediately. Another thread may still be reading `old.next` from that node. The notes stop at that warning; safe reclamation is a further design problem.

### Important exam points

- Atomic: others see before or after, not a mixture.
- Lock-free: some thread finishes even if others stall. A lock does not give this.
- Counter: `do { old = *count; } while (CAS(count, old, old + 1) != old)`.
- Stack push and pop swing `head` with compare-and-swap.
- ABA: the value returns to A after a real change, and CAS succeeds on a stale expectation.

### University exam questions

#### 2-mark questions

1. Define an atomic operation.
2. Define lock-free.
3. What is the ABA problem?
4. Why is a spinlock not lock-free?

#### 4/5-mark questions

1. Implement an atomic increment with compare-and-swap.
2. Explain lock-free stack push.
3. Explain ABA with a pointer that is popped and pushed again.

#### 8/10-mark questions

1. Explain atomic operations and lock-free algorithms. Write the lock-free stack and compare it with a stack protected by a mutex.
2. Explain compare-and-swap retry loops and the ABA problem. State what guarantee lock-free gives and what guarantee it does not give.

### Practice problems

#### Easy

1. Does an atomic increment need a mutex in the loop above?
2. What does compare-and-swap return when the expected value does not match?
3. In the counter trace, which thread retries?
4. A lock holder is preempted. Do threads blocked on that mutex keep completing stack operations?
5. Expand CAS as used in these algorithms.

#### Medium

1. `count` is 10. Two threads execute the atomic increment. What is the only correct final value, and why is 11 impossible?
2. Draw a stack N1 → N2. Show the pointers after a successful push of N3.
3. Two threads pop together. Why can only one compare-and-swap succeed?
4. State the ABA sequence in three steps.
5. Distinguish lock-free from wait-free in one sentence each.

#### Hard

1. Thread A has read head = N1 and next = N2. Before A’s compare-and-swap, thread B pops N1 and pushes N4, then pushes N1 so that N1’s successor is N4. What does A’s successful compare-and-swap do to N4?
2. Why does adding a version number to the pointer detect that sequence?
3. Explain a thread that fails its pop compare-and-swap forever while other threads succeed. Which definition does this violate, lock-free or wait-free?
4. Why is “free the node immediately after pop” unsafe even when the compare-and-swap succeeded?
5. A shared update changes two pointers that must be seen together. Why is one compare-and-swap on only one of them not enough?

**Answers**

- The final value is 12. One compare-and-swap stores 11. The other fails, reads 11, and stores 12. A final value of 11 would be a lost update.
- After push of N3, if it wins, head is N3 and N3.next is the old head N1.
- Only one pop finds head still equal to the pointer it sampled.
- ABA steps: read A; another thread changes A to B and back to A; the first compare-and-swap succeeds.
- In question 1 of the hard set, A writes next = N2 into head and the node N4, which was after N1, is no longer on the stack. The version number would have changed when N1 was reused, so A’s expected word would not match.
- Endless failure of one thread violates wait-free. Lock-free still holds if other pops complete.
- The slow thread may still load `next` from the node. Freeing it can reuse that memory for a different object.
- The two pointers would need one atomic update of a single descriptor word, or a lock around both stores. A successful swap of only one pointer publishes a mixed state.

### MCQs

**Q1. An atomic operation:**

A. Is observed entirely before or entirely after, not halfway  
B. Always takes a mutex  
C. Can be interrupted in the middle and still look correct to other cores  
D. Requires five philosophers  

**Answer:** A  

**Explanation:** Other threads do not see a partially updated word.

**Q2. A lock-free algorithm guarantees that:**

A. Some thread completes an operation even if another thread is stalled  
B. Every thread finishes in a fixed number of steps  
C. No compare-and-swap is used  
D. The ABA problem cannot be described  

**Answer:** A  

**Explanation:** The guarantee is system-wide progress, not progress of each individual thread. That stronger claim is wait-free.

**Q3. The atomic increment retries when:**

A. compare-and-swap finds that `count` is no longer the sampled value  
B. The addition is larger than 1  
C. The mutex is free  
D. The old value equals the new value and the store succeeded  

**Answer:** A  

**Explanation:** A mismatch means another thread stored a newer count. This thread must recompute.

**Q4. Lock-free push installs the new node when:**

A. `head` still equals the pointer stored in the new node’s next field  
B. A mutex is held  
C. `head` has changed to a different pointer  
D. The stack is empty only  

**Answer:** A  

**Explanation:** That is the expected value of the compare-and-swap.

**Q5. ABA means:**

A. A value changes and then returns to the original bit pattern, hiding the change from compare-and-swap  
B. Three philosophers eat together  
C. A mutex is recursive  
D. The stack is empty  

**Answer:** A  

**Explanation:** Compare-and-swap sees A again and treats the location as unchanged.

**Q6. A spinlock is not lock-free because:**

A. If the holder stops, the spinners do not complete their critical sections  
B. It uses compare-and-swap  
C. It has no loop  
D. Unlock is atomic  

**Answer:** A  

**Explanation:** Progress of the waiters depends on the holder running. Lock-free algorithms do not have that dependency.

**Q7. Two concurrent pops of a lock-free stack:**

A. Cannot both swing `head` away from the same node  
B. Always corrupt `head`  
C. Require a monitor  
D. Both return the same node after two successful compare-and-swaps on that node  

**Answer:** A  

**Explanation:** The second compare-and-swap no longer sees the original head, so it retries and pops whatever the head is now.

**Q8. Wait-free differs from lock-free because wait-free requires:**

A. Every thread to finish in a finite number of its own steps  
B. Only one thread in the whole system to finish  
C. A mutex  
D. The ABA pattern  

**Answer:** A  

**Explanation:** Lock-free allows one thread to retry indefinitely as long as others succeed. Wait-free does not.

### Quick revision

#### Atomic

One word moves from the old state to the new state with no visible middle.

#### Lock-free counter

```text
repeat
    old = *count
until compare_and_swap(count, old, old + 1) == old
```

#### Lock-free stack

Push and pop change `head` with compare-and-swap. Failure means retry. A stalled thread does not hold a lock.

#### ABA

The bits return to A after a real change. A version count makes the word different even if the pointer is reused.

#### Not the same

A spinlock is not lock-free. Lock-free is not automatically wait-free.
