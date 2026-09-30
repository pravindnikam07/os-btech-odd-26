**Navigation:** [Unit index](00_Index.md) · [Previous: 2.8 Multilevel queue and multilevel feedback queue](08_MLQ_and_MLFQ.md) · [Next: 2.10 Real-time scheduling: RM and EDF](10_Real_Time_RM_and_EDF.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 2 — Processes, Threads and CPU Scheduling

## Topic 2.9 — Multicore Scheduling

### Learning Outcomes

- Explain how scheduling changes when the computer has more than one core.
- Distinguish soft processor affinity and hard processor affinity.
- Explain push and pull load balancing.
- Explain the tension between a balanced load and cache affinity.

### Prerequisites

Ready queue, context switch, and kernel-level threads that can run in parallel (Topics 2.1 and 2.2).

### Introduction

A core is a processor that can execute instructions. A multicore chip has several cores. At any moment each core runs at most one thread, so several threads of the same or of different processes can run together. The scheduler’s new problems are which core runs a thread, and what to do when one core has a long ready queue while another core is idle.

### Definition

**Definition:**  
Multicore scheduling assigns ready threads to the cores of a multiprocessor so that the cores stay busy and a thread can reuse data left in a core’s cache.

**Definition:**  
Processor affinity is the preference, or the restriction, that a thread run on a particular core.

**Definition:**  
Load balancing moves work from a busy core to a less busy core.

### Two ways to organise the queues

| Organisation | How it works | Consequence |
| --- | --- | --- |
| Common ready queue | One queue for all cores. A core that becomes free takes the next thread | Automatic sharing of work. Several cores may contend for the same queue lock. Affinity is weaker |
| Per-core ready queue | Each core has its own queue | A thread can stay on the same core. A core can be idle while another core’s queue is long, unless the system balances the queues |

Modern general-purpose systems use per-core run queues and a separate balancing step. Linux CFS, described in Topic 2.11, is organised this way: each core has a run queue.

```text
Common queue                         Per-core queues

+----+----+----+                     Core 0: P1 -> P2
| P1 | P2 | P3 |                     Core 1: P3
+----+----+----+                     Core 2: (idle)
   |     |     |
 Core0 Core1 Core2                   Balancing may move P2 onto Core 2
```

### Processor affinity

When a thread runs on Core 0, the core’s cache fills with that thread’s data and instructions. If the next dispatch of the same thread is also on Core 0, many of those references hit in the cache. If the thread is moved to Core 1, it starts with a cold cache and must fetch the data again. Keeping the thread on Core 0 is processor affinity.

| Kind | Meaning | Who decides |
| --- | --- | --- |
| Soft affinity | The scheduler tries to keep a thread on the same core, but it may move the thread to balance the load | The operating system |
| Hard affinity | The thread is allowed to run only on a stated set of cores | The programmer or administrator, through a system call such as Linux `sched_setaffinity` |

Soft affinity is the normal policy. Hard affinity is used when a thread must stay near a particular device, or when an experiment must pin a measurement thread to one core.

```text
Thread T last ran on Core 0

Soft affinity:
  Prefer Core 0
  Move to Core 1 only if balancing requires it

Hard affinity, mask = {Core 0}:
  Core 0 is allowed
  Core 1 is forbidden, even if Core 1 is idle
```

### Load balancing

**Definition:**  
Load balancing keeps the amount of ready work from becoming concentrated on one core.

| Method | Who acts | What happens |
| --- | --- | --- |
| Push migration | A busy core, or a periodic balancer | A thread is moved from an overloaded core to a less loaded core |
| Pull migration | An idle or lightly loaded core | That core takes a thread from a busier core’s queue |

```text
Push
Core 0 queue: T1 T2 T3 T4          Core 1 queue: T5
                 |
                 +---- move T4 ---> Core 1

Pull
Core 0 queue: T1 T2 T3             Core 1 queue: empty
                 |
                 +---- Core 1 takes T3
```

Balancing is checked periodically and also when a core becomes idle. Moving a thread has a cost: the context is transferred, and the destination core does not have the thread’s cache contents. A system that migrates on every small difference in queue length will damage affinity and spend time on migration. A system that never migrates will leave a core idle. The scheduler therefore balances when the imbalance is significant, and otherwise leaves a thread on its current core.

### How a scheduling decision fits together

```text
A core needs work
        ↓
Soft affinity: is the previously run thread ready and is this core allowed?
        ↓
Yes, and this core is not required to steal work  →  run it here
        ↓
No ready thread on this core
        ↓
Pull a thread from a busier core, respecting hard affinity
        ↓
Dispatcher on this core loads the thread
```

1. The core looks at its own run queue first.
2. Hard affinity removes any thread that is not allowed on this core.
3. If the local queue has a ready thread, the core’s scheduler, for example CFS on Linux, chooses among those threads.
4. If the local queue is empty, pull migration takes a thread whose affinity mask includes this core.
5. Push migration may still run in the background when one queue grows much longer than the others.

### Advantages and limitations

| Advantages | Limitations |
| --- | --- |
| Several threads run at the same time | A thread cannot use two cores for a single instruction stream |
| Soft affinity reuses cache contents | Migration makes the cache cold |
| An idle core can pull work and raise utilization | Queue locks and migration add overhead |
| Hard affinity supports placement next to a device or a pinned experiment | A hard mask that excludes every busy-but-allowed arrangement can leave a thread waiting while other cores are idle |

### Applications

- A web server runs several request threads on several cores.
- A parallel program pins each worker to a core so that the workers do not migrate during a measurement.
- The operating system moves a ready thread onto an idle core rather than leaving that core stopped.
- Interrupt handling can be steered toward a chosen core, and the thread that processes the result can be given affinity for the same core.

### Common mistakes

- Saying a process with one thread runs on every core at once. One thread runs on one core at a time. Parallel use of cores needs several runnable threads.
- Treating load balancing and affinity as the same goal. Balancing moves threads. Affinity prefers not to move them.
- Describing hard affinity as a hint. Hard affinity is a restriction. Soft affinity is the hint.
- Forgetting that pull migration happens when a core is idle, and push migration is initiated from the overloaded side or by a periodic balancer.

### Important exam points

- Per-core queues versus one common queue.
- Soft affinity: prefer the same core for the cache. Hard affinity: a permitted set of cores.
- Push migration and pull migration.
- The reason not to migrate on every tiny imbalance: cache contents are lost.
- One thread occupies one core.

### University exam questions

#### 2-mark questions

1. What is processor affinity?
2. Differentiate soft and hard affinity.
3. What is push migration?
4. What is pull migration?

#### 4/5-mark questions

1. Explain load balancing on a multicore system.
2. Explain why the scheduler prefers to run a thread on the same core.
3. Compare a common ready queue with per-core ready queues.

#### 8/10-mark questions

1. Explain multicore scheduling with processor affinity and load balancing. Show push and pull migration with a diagram.
2. Explain the conflict between cache affinity and load balancing, and how soft affinity differs from hard affinity.

### Practice problems

#### Easy

1. Can one thread execute on two cores at the same instant?
2. Which affinity is only a preference?
3. Which migration is started by an idle core?
4. What hardware structure becomes less useful if a thread changes core?
5. Name one Linux call associated with hard affinity.

#### Medium

1. Core 0 has four ready threads and Core 1 has none. Describe a pull and a push that repair this.
2. A thread’s hard affinity mask is {Core 0}. Core 0 is busy and Core 1 is idle. May the scheduler run the thread on Core 1?
3. Why does a per-core queue need an extra balancing mechanism?
4. Explain soft affinity in terms of cache hits.
5. A process has four kernel threads and the chip has two cores. What is the maximum number of those threads that can run at the same time?

#### Hard

1. Explain a situation in which migrating a thread reduces CPU idle time but increases the thread’s memory-access time.
2. Why can a common ready queue weaken affinity?
3. Hard affinity is set to {Core 2} and Core 2 fails. What scheduling problem appears?
4. Compare the cost of a context switch that stays on the same core with one that moves the thread to another core.
5. Why do general-purpose systems avoid migrating a thread for a queue-length difference of one thread?

### MCQs

**Q1. Soft processor affinity means the scheduler:**

A. Tries to run a thread on the same core, but may move it  
B. Never allows the thread to run on that core  
C. Duplicates the thread on every core at the same time  
D. Removes the cache  

**Answer:** A  

**Explanation:** Soft affinity is a preference used to keep cache contents useful. Load balancing can override it.

**Q2. Hard processor affinity means:**

A. The thread may run only on cores in a specified set  
B. The thread must change core on every quantum  
C. Every core must run the thread together  
D. The ready queue is stored in the cache only  

**Answer:** A  

**Explanation:** The affinity mask is a restriction, not a hint.

**Q3. Pull migration is performed by:**

A. An idle or lightly loaded core, taking work from a busier core  
B. A thread that refuses to run  
C. The disk, moving processes into a file  
D. A core that pushes work away even when it is the idle one, by definition of pull  

**Answer:** A  

**Explanation:** The underloaded core pulls a ready thread onto itself.

**Q4. Push migration:**

A. Moves a thread from an overloaded core toward a less loaded core  
B. Copies the thread’s stack to every core  
C. Increases the burst time  
D. Disables the cache permanently  

**Answer:** A  

**Explanation:** The busy side, or a balancer watching it, pushes excess work outward.

**Q5. A thread is usually kept on the same core because:**

A. That core’s cache may already hold its data and instructions  
B. Other cores cannot execute instructions  
C. The PCB forbids a core number  
D. Load balancing is illegal  

**Answer:** A  

**Explanation:** Reusing the cache avoids fetching the working set again.

**Q6. One thread at one instant runs on:**

A. One core  
B. All cores, each executing the same instruction stream independently by the definition of a thread  
C. No core if it is ready  
D. The ready queue itself  

**Answer:** A  

**Explanation:** Parallel use of several cores requires several runnable threads.

**Q7. Per-core run queues need load balancing because:**

A. One core can have many ready threads while another core is idle  
B. A common queue cannot be drawn  
C. Affinity forbids every core  
D. The quantum must be zero  

**Answer:** A  

**Explanation:** Separate queues do not share work unless threads are moved.

**Q8. Frequent migration hurts performance because:**

A. The thread loses the benefit of the cache on its previous core  
B. The thread’s arrival time changes  
C. FCFS becomes SJF  
D. Hard affinity is created automatically  

**Answer:** A  

**Explanation:** The destination core must reload the thread’s working set into its cache.

### Quick revision

#### Queues

One common queue shares work but contends and weakens affinity. Per-core queues preserve affinity and need balancing.

#### Affinity

- Soft: prefer the last core.
- Hard: run only on the allowed set.

#### Balancing

- Push: overloaded side moves a thread out.
- Pull: idle side takes a thread in.
- Migrate for a real imbalance, not for a trivial one, because the cache becomes cold.
