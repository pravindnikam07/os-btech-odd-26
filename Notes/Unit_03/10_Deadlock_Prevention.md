**Navigation:** [Unit index](00_Index.md) · [Previous: 3.9 Deadlock and the four necessary conditions](09_Deadlock_and_Necessary_Conditions.md) · [Next: 3.11 Banker’s algorithm](11_Bankers_Algorithm.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 3 — Process Synchronization and Deadlocks

## Topic 3.10 — Deadlock Prevention

### Learning Outcomes

- Define deadlock prevention.
- Deny each of the four necessary conditions with a concrete policy.
- Apply a resource ordering to break circular wait.
- State the cost of each policy.

### Prerequisites

The four necessary conditions (Topic 3.9).

### Introduction

Prevention does not look at a particular request and ask whether this allocation is safe. It builds the allocation rules so that one of the four necessary conditions is impossible. If that condition cannot occur, deadlock cannot occur. The policies are simple to state and often expensive or impossible for a particular resource. Avoidance, in the next topic, keeps the conditions and refuses individual unsafe requests instead.

### Definition

**Definition:**  
Deadlock prevention is a set of constraints on resource requests that denies at least one of the four necessary conditions for deadlock.

### The four policies

| Condition denied | Policy | What the process must do |
| --- | --- | --- |
| Mutual exclusion | Make the resource shareable | Allow concurrent use. Possible for a read-only file. Impossible for a printer or a mutex that protects an update |
| Hold and wait | Do not hold anything while waiting | Request every resource before execution starts, or release all held resources before requesting more |
| No preemption | Allow preemption | If a process asks for a resource it cannot get, it releases the resources it holds. Those resources are restarted later. Suitable for a register state that can be saved. Unsuitable for a printer halfway through a page, and unsuitable for many locks |
| Circular wait | Impose a total order on resource types | Every process requests resources in increasing order of that numbering, and releases them in the opposite direction |

```text
Prevention
    |
    +-- deny mutual exclusion      only if sharing is correct
    +-- deny hold and wait         all at once, or release-then-request
    +-- deny no preemption         take resources back
    +-- deny circular wait         one global order of resource types
```

### Hold and wait in practice

**All resources at the start.**  
A process declares every tape, disk block, and lock it will need and receives them before it runs. It never waits while holding. The condition is denied. The cost is that resources sit idle during the parts of the program that do not use them, and a process may not know its maximum need.

**Release, then request.**  
A process that holds a printer and now needs a tape must release the printer before waiting for the tape. It may have to redo the print. It does not hold and wait. The cost is repeated work and the possibility that, after the release, another process takes the printer for a long time.

### Resource ordering

Number the resource types. Example: mutex X is type 1, mutex Y is type 2, the tape drive is type 3. Every process may request a resource only if its type number is greater than the type numbers of resources it already holds.

```text
Legal:    request X (1), then Y (2), then tape (3)
Illegal:  hold Y (2), then request X (1)
```

Suppose there were a cycle. Each process in the cycle would be waiting for a resource whose number is strictly greater than the numbers it holds. Following the cycle would produce a strictly increasing sequence of numbers that returns to the start. An integer sequence cannot strictly increase and return. Therefore no cycle exists.

This is the same idea as the even/odd chopstick order. The chopsticks receive an order, and not every philosopher requests them in the same direction. A total order is the general rule; the philosopher asymmetry is one way to satisfy it at a round table.

Both locks in the two-thread example must be taken in the order X then Y, in every thread. The deadlock in Topic 3.9 was exactly a violation of that order.

### What prevention is not

| Approach | When the decision is made | Does it allow all four conditions? |
| --- | --- | --- |
| Prevention | The rules are fixed in advance | No. One condition is structurally impossible |
| Avoidance | At each request, using maximum claims | The conditions may hold, but an unsafe allocation is refused |
| Detection and recovery | After the fact | The conditions may hold and deadlock may exist; the system then breaks it |

Calling the Banker’s algorithm prevention is a common error. The Banker’s algorithm allows hold and wait, mutual exclusion, no preemption, and even a pattern that could become a circle. It refuses the request that would leave no safe sequence. That is avoidance.

### Advantages and limitations

| Policy | Advantage | Limitation |
| --- | --- | --- |
| Deny mutual exclusion | Deadlock on that resource disappears | Many resources are useless if shared |
| Request everything first | Simple to explain | Low utilization; maximum need must be known |
| Release before a new request | A process never holds and waits | Work may be repeated; starvation is still possible |
| Preempt resources | Breaks the “only the owner releases” rule | Cannot rewind a printer or an arbitrary lock |
| Total order | A clear proof that cycles are impossible | The order is global; a library that locks in another order breaks it |

### Common mistakes

- Denying two conditions in an explanation and not saying which policy denies which condition.
- Claiming a total order denies hold and wait. It denies circular wait. A process may still hold resource 1 while it waits for resource 2.
- Using different orders in different modules. The proof needs one order for every request in the system.
- Describing prevention as “run the safety algorithm.” That sentence is avoidance.

### Important exam points

- Prevention = make one necessary condition impossible.
- Four policies in a table, each named against its condition.
- Ordering proof: a cycle would need a strictly increasing loop of resource numbers.
- Banker’s algorithm is not prevention.

### University exam questions

#### 2-mark questions

1. Define deadlock prevention.
2. Which condition does a total resource order deny?
3. State one way to deny hold and wait.
4. Why can mutual exclusion not be denied for a printer?

#### 4/5-mark questions

1. Explain prevention by denying hold and wait.
2. Explain prevention by a resource ordering. Sketch the cycle argument.
3. Differentiate prevention and avoidance.

#### 8/10-mark questions

1. Explain deadlock prevention by showing one policy for each necessary condition, with the disadvantage of each policy.
2. Prove that a strictly increasing order of resource requests prevents circular wait. Illustrate it with two mutexes.

### Practice problems

#### Easy

1. Which condition is denied if resources are allocated all at once?
2. Which condition is denied if resource types have numbers and requests must increase?
3. Can a mutex that protects a shared counter be made shareable?
4. Is preemption a natural policy for a printer in the middle of a page?
5. Does prevention run after the deadlock has been observed?

#### Medium

1. A process holds resource 5 and requests resource 3. Which ordering rule does this break?
2. Rewrite the two-thread deadlock so both threads take X before Y. Which condition disappears?
3. A process releases the printer before waiting for the tape, then acquires the printer again. Which condition was denied during the wait?
4. Give one utilization cost of allocating every resource at the start.
5. Explain why two modules must share one numbering, not one numbering each.

#### Hard

1. Write the contradiction: a cycle and a strictly increasing resource number cannot both be true.
2. Philosophers number chopsticks 0 to 4. Philosopher 4 needs chopsticks 4 and 0. Why is “always pick up the higher number second” impossible for that philosopher, and what extra rule is used?
3. Compare “release all, then request both” with resource ordering for two mutexes. Which conditions do they deny?
4. A saved CPU-register context can be preempted. A half-printed page cannot. Explain the difference using the definition of no preemption.
5. Why does prevention of circular wait still allow a process to wait?

**Answers**

- Allocating all resources before execution denies hold and wait.
- An increasing order denies circular wait.
- Holding 5 and requesting 3 decreases the number. The legal request would release 5 first, or request 3 before 5.
- After both threads take X before Y, the circular order is gone. They may still hold X and wait for Y, so hold and wait remains. Only one of them gets X; the other waits without holding Y.
- Releasing the printer before waiting denies hold and wait for the duration of that wait.
- Philosopher 4’s chopsticks are not in increasing order if 0 is numerically less than 4. The usual extra rule is: pick up the lower number first except that one designated philosopher picks up the highest first, or allow at most four to sit. The total-order proof applies when every process can follow the order; the wrap-around of a ring needs that extra break.
- “Release all, then request both” denies hold and wait. Ordering denies circular wait.
- Waiting itself remains legal. The order restricts what may be waited for while something is held. It does not require the process to receive every resource immediately.

### MCQs

**Q1. Deadlock prevention works by:**

A. Making one of the four necessary conditions impossible  
B. Recovering after a wait-for cycle is found  
C. Granting every request and testing a safe sequence  
D. Replacing mutual exclusion with a race  

**Answer:** A  

**Explanation:** If a necessary condition cannot hold, deadlock cannot hold.

**Q2. Requesting all resources before execution denies:**

A. Hold and wait  
B. Mutual exclusion  
C. Circular wait, by numbering  
D. The existence of processes  

**Answer:** A  

**Explanation:** The process does not hold one resource while waiting for another. It waits, if at all, before it holds any of them.

**Q3. A total order of resource types denies:**

A. Circular wait  
B. Mutual exclusion  
C. The need for a CPU  
D. Starvation in every scheduler  

**Answer:** A  

**Explanation:** A cycle would require resource numbers that increase all the way around, which is impossible.

**Q4. Mutual exclusion should be denied when:**

A. The resource can be used correctly by several processes at once, such as a read-only file  
B. The resource is a printer  
C. The resource is a mutex protecting a counter  
D. The process is in a critical section for a write  

**Answer:** A  

**Explanation:** Sharing removes exclusive hold. It is not available for resources that cannot be shared correctly.

**Q5. Preemption, as a prevention policy, means:**

A. A resource can be taken from a process and given back later  
B. Processes request resources in increasing order  
C. All resources are taken at program start and never released  
D. The Banker’s algorithm rejects a request  

**Answer:** A  

**Explanation:** That policy denies “no preemption.”

**Q6. The Banker’s algorithm is:**

A. Avoidance, not prevention  
B. Prevention of mutual exclusion  
C. A page-replacement policy  
D. Peterson’s solution  

**Answer:** A  

**Explanation:** It allows the four conditions and refuses an unsafe request. It does not make a condition structurally impossible.

**Q7. Under resource ordering, a process that holds resource 2 may next request:**

A. Resource 3, if 3 is the next type it needs  
B. Resource 1, while still holding 2  
C. Resource 2 a second time as a lower number  
D. Any lower number, because the order is only a comment  

**Answer:** A  

**Explanation:** New requests must have a higher number than resources already held.

**Q8. A disadvantage of allocating every resource at the start is:**

A. Resources may sit unused for most of the program  
B. Circular wait becomes more likely  
C. Mutual exclusion is strengthened for read-only files  
D. The order of mutexes no longer matters and deadlock returns  

**Answer:** A  

**Explanation:** Utilization falls because the process reserves resources before it needs them.

### Quick revision

#### Definition

Constrain requests so that one necessary condition cannot occur.

#### Policies

| Deny | How |
| --- | --- |
| Mutual exclusion | Share the resource, if that is correct |
| Hold and wait | Take all resources first, or release all before a new request |
| No preemption | Take resources away and restore them later |
| Circular wait | One increasing order of resource types |

#### Proof sketch

A cycle plus strictly increasing resource numbers cannot close.

#### Not this topic

The Banker’s algorithm is avoidance.
