**Navigation:** [Unit index](00_Index.md) · [Previous: 2.10 Real-time scheduling: RM and EDF](10_Real_Time_RM_and_EDF.md) · [Next: Unit summary](12_Unit_Summary.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 2 — Processes, Threads and CPU Scheduling

## Topic 2.11 — Linux CFS

### Learning Outcomes

- Explain the goal of the Completely Fair Scheduler.
- Compute a time share from task weights.
- Explain virtual runtime and why the task with the smallest virtual runtime runs next.
- Describe the red-black tree used as the run queue.

### Prerequisites

Preemptive scheduling, multicore per-core run queues (Topic 2.9), and the idea of a priority or nice value (Topic 2.7).

### Introduction

Since Linux 2.6.23, ordinary processes have been scheduled by the Completely Fair Scheduler (CFS). CFS does not give every process a fixed Round Robin quantum chosen in advance. It tries to give each runnable process a share of the CPU in proportion to a weight. The weight comes from the nice value. Over an interval called the target latency, every runnable process should receive its share, as if the processor were divided among them at the same time. A single core cannot do that literally, so CFS runs one process at a time and keeps an account, the virtual runtime, of how much each process has received.

CFS schedules normal policy tasks (`SCHED_NORMAL`, also called `SCHED_OTHER`). Real-time policies are handled separately from this fair account.

### Definition

**Definition:**  
CFS is the Linux scheduler for ordinary processes. It orders runnable processes by virtual runtime and runs the process that has so far received the least weighted CPU time.

**Definition:**  
Virtual runtime (vruntime) is the CPU time a task has received, scaled by its weight. A higher-weight task’s virtual runtime grows more slowly for the same wall-clock run, so it is allowed to run longer before another task catches up.

### Key terminology

| Term | Meaning |
| --- | --- |
| Nice value | A user-set priority adjustment. A higher nice value means a lower weight and a smaller CPU share |
| Weight | The scheduler’s numeric importance of a task. Nice 0 has weight 1024 |
| vruntime | Weighted CPU time already consumed |
| min_vruntime | The smallest virtual runtime on the run queue, used as a moving base |
| Target latency | The period over which runnable tasks should each receive their fair share |
| Red-black tree | The balanced tree used as the CFS run queue, ordered by vruntime |
| Leftmost node | The node with the smallest vruntime; the next task to run |

### Weight and share

For a set of runnable tasks, the fair wall-clock time of task i during one target latency L is

```text
time_i = L × (weight_i / Σ weight)
```

Nice 0 corresponds to weight 1024. A larger nice value reduces the weight, so that task receives less CPU. A negative nice value increases the weight.

**Example.**  
Target latency L = 12 ms. Task A has weight 1024. Task B has weight 512. The total weight is 1536.

```text
time_A = 12 × (1024 / 1536) = 12 × (2/3) = 8 ms
time_B = 12 × (512 / 1536)  = 12 × (1/3) = 4 ms
```

A receives twice the processor time of B because its weight is twice B’s weight. The two slices add to the target latency.

### Virtual runtime

While a task runs for an actual time Δ, CFS advances its virtual runtime by

```text
vruntime  =  vruntime  +  Δ × (1024 / weight)
```

The constant 1024 is the weight of nice 0. A nice-0 task’s virtual runtime grows one-to-one with wall-clock time. A lighter task’s virtual runtime grows faster, so it reaches a large vruntime after less real CPU time and then yields to others.

Using the same A and B:

```text
A runs 8 ms:  vruntime_A increases by  8 × (1024 / 1024) = 8
B runs 4 ms:  vruntime_B increases by  4 × (1024 / 512)  = 8
```

Both accounts increase by 8. The scheduler has been unfair in wall-clock time, on purpose, and fair in virtual time. After these slices, neither task is behind the other.

```text
Algorithm: CFS selection

Step 1: Keep a red-black tree of runnable tasks, ordered by vruntime.
Step 2: Select the leftmost task, which has the smallest vruntime.
Step 3: Run it for a slice based on its weight and the target latency,
        but not below a minimum granularity if many tasks are runnable.
Step 4: Add Δ × (1024 / weight) to its vruntime.
Step 5: Reinsert it into the tree at its new vruntime.
Step 6: Again select the leftmost task.
```

If only one task is runnable, it runs without being sliced against another task. Its vruntime still advances. When a second task wakes, the comparison uses the stored accounts.

A task that has been sleeping would otherwise have a very small vruntime and could occupy the CPU for a long time to catch up. CFS places a waking task near the current minimum virtual runtime, so the task runs soon and is not given an unbounded debt to collect. The exact adjustment is a kernel parameter; the examination point is the reason for the adjustment.

### The red-black tree

```text
                 vruntime 20
                 /         \
        vruntime 10       vruntime 30
        /        \
 vruntime 4    vruntime 12
     ^
 leftmost: run this task next
```

- Insertion and removal of a task are O(log n) for n runnable tasks.
- The leftmost node is cached, so choosing the next task does not require a walk of the whole tree.
- There is one such run queue per CPU. Which CPU a task uses is the affinity and load-balancing decision from Topic 2.9. CFS then orders the tasks on that CPU.

This replaces the older Linux O(1) scheduler, which used fixed priority arrays. CFS does not keep a separate queue for each nice value. The tree is ordered by the accounting value, vruntime.

### A short timeline

A and B both start with vruntime 0. A has weight 1024 and B has weight 512. The leftmost node can be either task; suppose A is started.

| Wall time | What happens | vruntime A | vruntime B |
| ---: | --- | ---: | ---: |
| 0 | A is selected | 0 | 0 |
| 8 | A has run its 8 ms share and is reinserted | 8 | 0 |
| 8 | B is now leftmost | 8 | 0 |
| 12 | B has run its 4 ms share | 8 | 8 |
| 12 | Virtual runtimes match; the next choice may be A again | 8 | 8 |

The ideal continuous shares would have been 2/3 and 1/3 throughout the 12 ms. CFS approximates that ideal by alternating slices.

### Comparison with algorithms already studied

| Parameter | Round Robin | CFS |
| --- | --- | --- |
| Slice | A fixed quantum for every process | A share proportional to weight |
| Who runs next | The process at the head of a FIFO queue | The runnable task with the smallest vruntime |
| Priority | Not used, unless RR is only one queue of a larger scheduler | Nice value, through the weight |
| Queue structure | FIFO | Red-black tree |
| Fairness | Equal turns | Equal progress in virtual time |

CFS is not Rate Monotonic or EDF. It does not assign priority by period or by deadline. Real-time Linux tasks use real-time policies; CFS is the scheduler for ordinary time-shared work.

### Advantages and limitations

| Advantages | Limitations |
| --- | --- |
| A task that has received less than its share is preferred next | The ideal parallel CPU is only approximated by slices |
| Nice values change the share in a defined ratio | A very small slice, when many tasks are runnable, increases switching; the minimum granularity limits this |
| The next task is found in logarithmic time | Sleeping tasks need a vruntime adjustment so they do not later monopolise the CPU |
| Per-CPU trees fit multicore machines | CFS does not provide the deadline guarantee of RM or EDF |

### Common mistakes

- Describing CFS as Round Robin with one global quantum. The slice depends on weights.
- Selecting the largest vruntime. The smallest vruntime runs, because that task is behind.
- Adding the raw Δ to every task’s vruntime. The increment is Δ × (1024 / weight).
- Claiming one CFS tree for the whole machine. Each CPU has a run queue. Load balancing moves tasks between CPUs.
- Using CFS as the answer to a hard real-time deadline question. Deadline guarantees come from a real-time policy, not from CFS.

### Important exam points

- Goal: fair share by weight, recorded as vruntime.
- time_i = L × weight_i / total weight.
- vruntime increases by Δ × 1024 / weight.
- Next task: leftmost node of the red-black tree, the smallest vruntime.
- Nice 0 has weight 1024. A larger nice value means less CPU.
- Worked pair: weights 1024 and 512, latency 12 ms, slices 8 ms and 4 ms, both vruntimes increase by 8.

### University exam questions

#### 2-mark questions

1. What does CFS try to equalise among runnable tasks?
2. Which task does CFS run next?
3. Write the virtual-runtime update.
4. What is the weight of nice 0?

#### 4/5-mark questions

1. Explain virtual runtime with the weights 1024 and 512.
2. Explain the CFS red-black tree.
3. Differentiate Round Robin and CFS.

#### 8/10-mark questions

1. Explain the Linux Completely Fair Scheduler: target latency, weight, virtual runtime, and the run queue. Work the 8 ms and 4 ms example.
2. Explain how CFS approximates an ideal fair processor, and how a per-CPU tree is related to multicore scheduling.

### Practice problems

#### Easy

1. Does the task with the largest vruntime run next?
2. Two tasks have equal weight. How is a 20 ms target latency divided?
3. What data structure holds the CFS run queue?
4. A task runs for 6 ms at weight 1024. By how much does vruntime grow?
5. Does a higher nice value increase or decrease the CPU share?

#### Medium

1. Weights are 1024 and 512 and the target latency is 18 ms. Compute each slice.
2. For those slices, compute the vruntime increase of each task.
3. Why does the lighter task’s vruntime grow faster per millisecond?
4. Why is a waking task not always allowed to keep a very old, very small vruntime?
5. Why is the next task the leftmost node?

#### Hard

1. Three nice-0 tasks are runnable and the target latency is 18 ms. What slice does each receive?
2. Explain O(log n) for insertion into the CFS tree, and why picking the minimum need not scan all n tasks.
3. A task of weight 2048 and a task of weight 1024 share a 12 ms latency. Compute the slices and show that the vruntime increases match.
4. Why can two CPUs not share one vruntime ordering without a further decision?
5. Contrast CFS fairness with the EDF rule in one paragraph.

**Answers**

- Equal weights split 20 ms into 10 ms and 10 ms.
- A 6 ms run at weight 1024 increases vruntime by 6.
- A higher nice value decreases the share.
- For latency 18 ms and weights 1024 and 512: slices are 12 ms and 6 ms. Vruntime increases are 12 × 1024/1024 = 12 and 6 × 1024/512 = 12.
- Three equal tasks over 18 ms receive 6 ms each.
- Weights 2048 and 1024, latency 12 ms: total weight 3072. The heavier task gets 12 × 2048/3072 = 8 ms. The lighter gets 4 ms. Vruntime increases: 8 × 1024/2048 = 4, and 4 × 1024/1024 = 4.
- Each CPU has its own run queue. Choosing the CPU is load balancing and affinity; CFS then orders that CPU’s tasks.

### MCQs

**Q1. CFS selects the runnable task with:**

A. The smallest virtual runtime  
B. The largest virtual runtime  
C. The earliest arrival time only, as in FCFS  
D. The shortest period, as in Rate Monotonic scheduling  

**Answer:** A  

**Explanation:** A small vruntime means the task is behind its fair share, so it runs next.

**Q2. The virtual-runtime increase for a run of length Δ is:**

A. Δ × (1024 / weight)  
B. Δ × weight, with no division  
C. The target latency only  
D. The number of CPUs  

**Answer:** A  

**Explanation:** Division by the weight makes a heavier task’s account grow more slowly.

**Q3. Nice 0 has weight:**

A. 1024  
B. 1  
C. 0  
D. The length of the ready queue  

**Answer:** A  

**Explanation:** The kernel uses 1024 as the weight of a normal nice-0 task, and the vruntime formula scales against that weight.

**Q4. In a 12 ms target latency, weights 1024 and 512 produce slices of:**

A. 8 ms and 4 ms  
B. 6 ms and 6 ms  
C. 4 ms and 8 ms  
D. 12 ms and 12 ms  

**Answer:** A  

**Explanation:** The shares are 2/3 and 1/3 of 12 ms.

**Q5. After those slices, both virtual runtimes increase by:**

A. 8  
B. 4  
C. 12  
D. 1024  

**Answer:** A  

**Explanation:** 8 × 1024/1024 = 8 and 4 × 1024/512 = 8.

**Q6. The CFS run queue is a:**

A. Red-black tree ordered by vruntime  
B. Single FIFO queue with a fixed quantum for every nice value  
C. Stack  
D. List of absolute deadlines  

**Answer:** A  

**Explanation:** The balanced tree keeps tasks in vruntime order. The leftmost node is the next task.

**Q7. A larger nice value:**

A. Reduces the task’s weight and its CPU share  
B. Increases the share above nice 0  
C. Converts the task into an EDF task  
D. Pins the task to every CPU  

**Answer:** A  

**Explanation:** Nice is a courtesy: a higher nice value yields CPU time to other tasks.

**Q8. CFS differs from EDF because CFS:**

A. Fairly shares the CPU by weight and does not schedule by absolute deadline  
B. Always runs the shortest period  
C. Is non-preemptive FCFS  
D. Uses one queue for the whole computer and ignores per-CPU queues  

**Answer:** A  

**Explanation:** EDF is a real-time deadline rule. CFS is the fair scheduler for ordinary Linux tasks. Each CPU has its own CFS run queue.

### Quick revision

#### Idea

Give each runnable task CPU time in proportion to its weight, and record the result as virtual runtime.

#### Formulas

```text
time_i   =  L × weight_i / Σ weight
vruntime =  vruntime + Δ × (1024 / weight)
```

Nice 0 has weight 1024.

#### Example

Weights 1024 and 512, L = 12 ms. Slices 8 ms and 4 ms. Both virtual runtimes increase by 8.

#### Next task

The leftmost node of the per-CPU red-black tree: the smallest vruntime. Tree operations are O(log n).
