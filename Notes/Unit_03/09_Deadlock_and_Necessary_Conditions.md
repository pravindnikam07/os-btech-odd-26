**Navigation:** [Unit index](00_Index.md) · [Previous: 3.8 Atomic operations and lock-free algorithms](08_Atomic_and_Lock_Free.md) · [Next: 3.10 Deadlock prevention](10_Deadlock_Prevention.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 3 — Process Synchronization and Deadlocks

## Topic 3.9 — Deadlock and the Four Necessary Conditions

### Learning Outcomes

- Define deadlock and a resource.
- State the four necessary conditions.
- Show a two-process deadlock with two locks.
- Distinguish deadlock from starvation.

### Prerequisites

Semaphores, mutexes, and the dining philosophers’ circular wait (Topics 3.4, 3.5, and 3.7).

### Introduction

A process that waits for a semaphore may later be woken. Deadlock is the case in which the wait cannot end. Each process holds something another process needs, and each is waiting for something another process holds. None of them can run far enough to release what it holds. The four conditions below are necessary: if even one of them is absent, this kind of deadlock cannot exist. They are not a recipe for causing deadlock. They are the checklist used by prevention.

### Definition

**Definition:**  
A set of processes is deadlocked when every process in the set is waiting for an event that can be caused only by another process in the set.

The event is usually the release of a resource: a lock, a tape drive, a memory frame, or a chopstick in the philosophers’ example. The processes do not have to be user programs. Two kernel threads can deadlock on two spinlocks.

**Definition:**  
A resource is anything a process must acquire, use, and release. It may have one instance, such as a particular mutex, or several instances, such as three identical printers.

### The four necessary conditions

All four must hold at once.

| Condition | Meaning |
| --- | --- |
| Mutual exclusion | At least one resource is held in a non-shareable mode. If another process wants it, that process waits |
| Hold and wait | A process holds at least one resource and waits for another resource that is held by a different process |
| No preemption | A resource is released only voluntarily by the process that holds it. It cannot be taken away |
| Circular wait | There is a cycle of processes: P0 waits for a resource held by P1, P1 waits for a resource held by P2, …, and Pk waits for a resource held by P0 |

If one condition is denied for the resources in question, deadlock among those waits cannot form. Prevention is the systematic denial of one condition. That is Topic 3.10. Avoidance, detection, and recovery allow the conditions and then refuse an unsafe allocation or break a deadlock after it appears.

### Two processes and two locks

```text
Thread A                      Thread B
lock(X)                       lock(Y)
lock(Y)   // waits            lock(X)   // waits
```

| Condition | How it appears |
| --- | --- |
| Mutual exclusion | X and Y are mutexes. Each is held by one thread |
| Hold and wait | A holds X and waits for Y. B holds Y and waits for X |
| No preemption | Neither mutex is taken away; each waits for the owner to unlock |
| Circular wait | A → Y held by B → X held by A |

```text
A holds X -------- waits for Y
^                         |
|                         v
+-------- B holds Y ------+
```

The same picture is the five philosophers, each holding the left chopstick and waiting for the right. The cycle has length 5 instead of 2.

### Resource allocation graph

For one instance of each resource, a deadlock graph uses two kinds of nodes and two kinds of edges.

```text
Process Pi requests Rj:     Pi  ----→  Rj
Process Pi holds Rj:        Rj  ----→  Pi
```

```text
A holds X and requests Y.  B holds Y and requests X.

X → A → Y → B → X
```

A cycle in this single-instance graph means deadlock. With several instances of a resource type, a cycle means deadlock is possible; it does not by itself prove that deadlock exists. Detection in Topic 3.12 uses the wait-for graph when each resource has one instance, and a matrix algorithm when a type has several instances.

### Deadlock and starvation

| | Deadlock | Starvation |
| --- | --- | --- |
| What the process is doing | Waiting for an event that the other waiting processes must cause | Waiting, while the resource is repeatedly given to others |
| Can the others run? | Not if they are all in the same deadlocked set | Yes. They are the processes receiving the resource |
| Example | Two threads each holding one mutex and waiting for the other | A low-priority process under priority scheduling, or a writer waiting behind a stream of readers |
| Cure discussed here | Deny a necessary condition, avoid an unsafe state, or break the deadlock | Aging, or a queue that eventually serves the waiter |

A deadlocked process is not making progress, and neither are the processes it depends on. A starving process is not making progress either, but the rest of the system can be busy. Calling starvation a deadlock loses this distinction and usually loses the mark.

### Necessary, not sufficient as a story by itself

The four conditions are necessary. Deadlock implies that all four hold. The converse needs care: all four can hold and a particular allocation can still be arranged so that processes finish. That is why a cycle in a multiple-instance graph is not an automatic declaration of deadlock, and why the Banker’s algorithm talks about a safe sequence rather than about the four conditions alone. For the two-lock example, the four conditions are present and the threads are deadlocked. The example is enough.

### Common mistakes

- Listing three conditions and omitting circular wait, or omitting hold and wait.
- Saying the conditions are sufficient in every resource model. Learn them as necessary. The single-instance cycle is the case where the picture also settles the question.
- Describing deadlock as “a process is waiting.” Waiting is normal. Deadlock is a closed set of waits.
- Mixing deadlock with the race condition of Unit 3.1. A race produces a wrong value. A deadlock produces no further value.

### Important exam points

- Definition: each process waits for an event that only another process in the set can cause.
- Four conditions, all required: mutual exclusion, hold and wait, no preemption, circular wait.
- Two-lock diagram and the five-chopstick diagram.
- Request edge Pi → Rj, assignment edge Rj → Pi.
- Deadlock versus starvation.

### University exam questions

#### 2-mark questions

1. Define deadlock.
2. Name the four necessary conditions.
3. What is hold and wait?
4. Differentiate deadlock and starvation.

#### 4/5-mark questions

1. Explain the four necessary conditions with the two-mutex example.
2. Draw a resource-allocation graph for that example and identify the cycle.
3. Explain circular wait using the dining philosophers.

#### 8/10-mark questions

1. Define deadlock. Explain the four necessary conditions and show each one in a two-process, two-resource deadlock.
2. Explain resource-allocation graphs. State what a cycle means when each resource has one instance, and how deadlock differs from starvation.

### Practice problems

#### Easy

1. Name the condition that fails if a resource can be taken away from a holder.
2. Name the condition that fails if every process requests all its resources before holding any.
3. In the two-lock example, which resource does A hold?
4. Is a waiting writer, while readers run, deadlocked?
5. How many conditions must hold for deadlock to be possible?

#### Medium

1. Draw the four-edge cycle for mutexes X and Y.
2. Map each philosopher to the chopstick he holds and the chopstick he waits for.
3. A printer can be used by only one process, a data file can be read by many, and no process waits while holding something. Which condition is absent?
4. Explain why “no preemption” is true of an ordinary mutex.
5. Give one event that could end the two-lock wait, and say why the processes themselves cannot cause it.

#### Hard

1. Three processes: A holds R1 and wants R2, B holds R2 and wants R3, C holds R3 and wants R1. List the cycle and confirm all four conditions.
2. Change the previous system so C releases R3 when it cannot get R1. Which condition is denied, and does the cycle remain?
3. A system has two identical printers. A holds one and waits for a plotter. B holds the plotter and waits for a printer. Is there a cycle of processes? Are the printers a single-instance resource?
4. Explain a starvation that satisfies none of the circular-wait condition.
5. Why does the existence of all four conditions in a large system not replace a safety algorithm when resource types have many instances?

**Answers**

- Taking a resource away denies no preemption.
- Requesting everything first, and holding nothing while waiting, denies hold and wait.
- The writer is not deadlocked if the readers can finish and release the table. The writer may starve if readers never stop arriving.
- All four conditions are necessary.
- In the three-process cycle, A → R2 → B → R3 → C → R1 → A. Each holds one resource and waits for the next. If C must release R3 before waiting for R1, C no longer holds and waits, so hold and wait is denied for that request.
- Two printers are two instances of one type. The single-instance cycle test does not apply unchanged. B can receive A’s printer only after A releases it, and A will not release it while waiting for the plotter that B holds. This particular pair is deadlocked even though a printer type has two instances, because the free instance count is zero and each process holds what the other needs.
- A low-priority process can starve with no cycle: the CPU is not held by one process that is itself waiting for the low-priority process.

### MCQs

**Q1. A set of processes is deadlocked when:**

A. Each is waiting for an event that only another process in the set can cause  
B. Each is runnable and the CPU is free  
C. A race has produced a wrong counter  
D. One process waits and the others continue to release what it needs  

**Answer:** A  

**Explanation:** The waits form a closed set. No process outside the dependency can supply the missing event.

**Q2. The four necessary conditions are:**

A. Mutual exclusion, hold and wait, no preemption, circular wait  
B. Mutual exclusion, aging, preemption, a safe sequence  
C. Progress, bounded waiting, starvation, a race  
D. Fetch, decode, execute, store  

**Answer:** A  

**Explanation:** Deadlock requires all four. Prevention works by denying one of them.

**Q3. Hold and wait means a process:**

A. Holds one resource and waits for another  
B. Holds nothing and waits for its first resource  
C. Releases every resource before waiting  
D. Preempts the holder  

**Answer:** A  

**Explanation:** The process already has something and is blocked for something more.

**Q4. Circular wait is:**

A. A cycle of processes, each waiting for a resource held by the next  
B. A round-robin quantum  
C. Aging of a priority  
D. A compare-and-swap retry  

**Answer:** A  

**Explanation:** Following the waits returns to the first process.

**Q5. In a single-instance resource-allocation graph, a cycle means:**

A. The processes on the cycle are deadlocked  
B. The system is certainly safe  
C. Starvation is impossible  
D. The graph has no assignment edges  

**Answer:** A  

**Explanation:** With one instance of each resource, a cycle is deadlock. Several instances need a further test.

**Q6. An assignment edge runs:**

A. From a resource to the process that holds it  
B. From a process to a resource it is requesting  
C. From a process to another process only, in every graph  
D. From the CPU scheduler to a semaphore  

**Answer:** A  

**Explanation:** A request edge points from the process to the resource. An assignment edge points from the resource to the holder.

**Q7. Starvation differs from deadlock because:**

A. Other processes can still obtain the resource and run  
B. The starving process is inside a circular wait  
C. Starvation requires mutual exclusion to be false  
D. A starved process has finished  

**Answer:** A  

**Explanation:** The resource is being allocated, repeatedly, to someone else. In deadlock the holders are themselves stuck.

**Q8. No preemption means:**

A. Only the holder releases the resource  
B. The operating system always takes the resource back at a timer interrupt  
C. Processes share every resource  
D. Circular wait is impossible  

**Answer:** A  

**Explanation:** The resource cannot be snatched away. The holder must release it.

### Quick revision

#### Definition

Every process in the set waits for an event that only another process in the set can cause.

#### Four necessary conditions

Mutual exclusion. Hold and wait. No preemption. Circular wait.

#### Graph

Request: process → resource. Assignment: resource → process. One instance per resource: a cycle is deadlock.

#### Not starvation

Starvation: others keep running and taking the resource. Deadlock: the set is stuck together.
