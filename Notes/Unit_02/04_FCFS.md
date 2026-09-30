**Navigation:** [Unit index](00_Index.md) · [Previous: 2.3 Scheduling criteria](03_Scheduling_Criteria.md) · [Next: 2.5 SJF and SRTF](05_SJF_and_SRTF.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 2 — Processes, Threads and CPU Scheduling

## Topic 2.4 — First-Come, First-Served Scheduling

### Learning Outcomes

- Apply FCFS using arrival order.
- Draw a Gantt chart and compute turnaround time, waiting time, and response time.
- Explain the convoy effect.

### Prerequisites

Scheduling criteria and the formulas TAT = CT − AT, WT = TAT − BT, and RT = first start − AT (Topic 2.3).

### Introduction

First-Come, First-Served (FCFS) is the simplest CPU scheduling algorithm. The ready queue is a FIFO queue. The process that has been waiting longest is served next, and it runs until it blocks or finishes. FCFS is non-preemptive.

### Definition

**Definition:**  
FCFS schedules processes in the order of their arrival. A running process is not preempted by a later arrival.

### How it works

```text
Processes arrive
        ↓
Join the tail of the ready queue
        ↓
CPU becomes free
        ↓
Process at the head runs for its whole CPU burst
        ↓
Process blocks or completes
        ↓
Next process at the head runs
```

1. If the CPU is idle and the queue is not empty, dispatch the process at the head.
2. If several processes are present at time 0, the stated order (P1, P2, P3, …) is the arrival order.
3. A process that arrives while the CPU is busy waits at the tail. It does not interrupt the running process.
4. When the running process finishes, the process that arrived earliest among those waiting is selected.
5. If two processes arrive at the same time, the problem’s given order is used.

```text
Algorithm: FCFS

Step 1: Sort processes by arrival time. Keep the given order when arrival times are equal.
Step 2: Set the clock to 0.
Step 3: Take the next process in that order.
Step 4: If the clock is earlier than its arrival time, move the clock forward to the arrival time.
Step 5: Record the start time. Run the process for its full burst.
Step 6: Completion time = start time + burst time. Advance the clock to the completion time.
Step 7: Repeat from Step 3 until every process has run.
Step 8: TAT = CT − AT, WT = TAT − BT, RT = start − AT.
```

### Example — processes that arrive at different times

**Problem:**  
Schedule the following processes with FCFS.

| Process | Arrival time | Burst time |
| --- | ---: | ---: |
| P1 | 0 | 5 |
| P2 | 1 | 3 |
| P3 | 2 | 8 |
| P4 | 3 | 6 |

**Approach:**  
P1 arrives first and runs to completion. The others then run in arrival order. None of them preempts P1.

**Gantt chart:**

```text
| P1 | P2 |    P3    |    P4    |
0    5    8          16         22
```

**Calculation:**

| Process | AT | BT | Start | CT | TAT = CT − AT | WT = TAT − BT | RT = start − AT |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| P1 | 0 | 5 | 0 | 5 | 5 | 0 | 0 |
| P2 | 1 | 3 | 5 | 8 | 7 | 4 | 4 |
| P3 | 2 | 8 | 8 | 16 | 14 | 6 | 6 |
| P4 | 3 | 6 | 16 | 22 | 19 | 13 | 13 |

**Averages:**

- Average turnaround time = (5 + 7 + 14 + 19) / 4 = 45 / 4 = **11.25**
- Average waiting time = (0 + 4 + 6 + 13) / 4 = 23 / 4 = **5.75**
- Average response time = **5.75**

Response time equals waiting time because FCFS is non-preemptive: each process runs exactly once.

**Check for P4:**  
P4 arrives at 3 and starts at 16, so it waits 13. TAT = 22 − 3 = 19. WT = 19 − 6 = 13. The gap method and the formula agree.

### Example — convoy effect

**Problem:**  
All three processes arrive at time 0. Schedule them in the order P1, P2, P3.

| Process | Burst time |
| --- | ---: |
| P1 | 24 |
| P2 | 3 |
| P3 | 3 |

**Gantt chart:**

```text
|            P1            | P2 | P3 |
0                         24   27   30
```

| Process | CT | TAT | WT |
| --- | ---: | ---: | ---: |
| P1 | 24 | 24 | 0 |
| P2 | 27 | 27 | 24 |
| P3 | 30 | 30 | 27 |

Average waiting time = (0 + 24 + 27) / 3 = 51 / 3 = **17**

P2 and P3 are short, but they wait while the long process P1 runs. That is the **convoy effect**: short processes are stuck behind one long process, as vehicles are stuck behind a slow truck. If the short processes could run first, the average wait would fall sharply. Shortest-job scheduling, in the next topic, does that and reduces this average from 17 to 3.

### Characteristics

- The ready queue is FIFO.
- Implementation is simple: enqueue on arrival, dequeue when the CPU is free.
- No starvation in the sense of a process being overtaken forever by later processes. Every process reaches the head. A process can still wait a long time behind a long burst.
- FCFS does not use burst time, so it cannot prefer short or interactive work.
- It performs poorly when one CPU-bound process is followed by many I/O-bound or short processes.

### Advantages and limitations

| Advantages | Limitations |
| --- | --- |
| Simple to understand and code | Average waiting time is often high |
| No priority computation | Convoy effect |
| Fair in arrival order | Not suitable as the only policy for interactive time-sharing |
| Non-preemptive, so no timer is required for scheduling | A long burst delays every process behind it |

### Common mistakes

- Reordering processes by burst time. That is SJF, not FCFS.
- Preempting the running process when a new process arrives. FCFS does not preempt.
- Forgetting that a process cannot start before it arrives. If the CPU becomes free at time 4 and the next process arrives at time 6, the CPU is idle from 4 to 6.
- Using WT = CT − BT when the arrival time is not zero. Subtract the arrival time as well, through TAT.

### Important exam points

- FIFO, non-preemptive.
- Gantt chart in arrival order.
- TAT, WT, and RT formulas, with RT = WT for FCFS.
- Convoy effect, with the 24, 3, 3 example: average waiting time 17.

### University exam questions

#### 2-mark questions

1. Define FCFS scheduling.
2. Is FCFS preemptive or non-preemptive?
3. What is the convoy effect?
4. Which data structure represents the FCFS ready queue?

#### 4/5-mark questions

1. Explain FCFS with a Gantt chart and the average waiting time for a given set.
2. Explain the convoy effect with a numerical example.
3. Why is FCFS a weak policy for an interactive system?

#### 8/10-mark questions

1. Given processes with arrival and burst times, draw the FCFS Gantt chart and compute average turnaround time and average waiting time. Show every row of the table.
2. Explain FCFS. Compare its treatment of a long process followed by short processes with the idea of scheduling the short processes first.

### Practice problems

#### Easy

1. State the selection rule of FCFS.
2. Three processes arrive at time 0 in the order A, B, C. Which one runs first?
3. Does a newly arrived short process preempt a long running process under FCFS?
4. For FCFS, why is response time equal to waiting time?
5. Name the effect in which short processes wait behind one long process.

#### Medium

1. Draw the FCFS Gantt chart for (P1, AT 0, BT 4), (P2, AT 1, BT 3), (P3, AT 2, BT 1), (P4, AT 3, BT 2).
2. For that set, compute average waiting time.
3. Explain one idle-CPU situation under FCFS.
4. Why does FCFS need no estimate of the next CPU burst?
5. A long process arrives just before five short ones. Describe the waiting times of the short processes qualitatively.

#### Hard

1. Compute CT, TAT, WT, and RT for every process in the medium question, and the three averages.
2. Show that changing the order of two processes that arrived at the same time changes the average waiting time. Use bursts 10 and 1, both arrived at 0.
3. Compare FCFS average waiting time for bursts 24, 3, 3 in that order with the order 3, 3, 24. Both sets arrive at 0.
4. A process arrives at time 10 with burst 2. The CPU is busy until time 6 with another process, and no other work is waiting. When does this process start?
5. Explain why “no starvation” does not mean “short waiting time” under FCFS.

**Answers for the numerical practice**

- Medium 1 and Hard 1: Gantt P1 0–4, P2 4–7, P3 7–8, P4 8–10.  
  WT = 0, 3, 5, 5. Average WT = 13/4 = 3.25.  
  TAT = 4, 6, 6, 7. Average TAT = 5.75.  
  RT = WT for each process.
- Hard 2: order burst 10 then 1 gives WT 0 and 10, average 5. Order burst 1 then 10 gives WT 0 and 1, average 0.5.
- Hard 3: order 24, 3, 3 gives average WT 17. Order 3, 3, 24 gives WT 0, 3, and 6, average 3.
- Hard 4: the process starts at its arrival time, 10. The CPU is already idle from 6 to 10.

### MCQs

**Q1. FCFS selects the process that:**

A. Has the smallest burst time  
B. Arrived earliest among the waiting processes  
C. Has the highest priority number in every convention  
D. Has the nearest deadline  

**Answer:** B  

**Explanation:** FCFS is arrival order. Burst time, priority, and deadline are not the selection rule.

**Q2. FCFS is:**

A. Preemptive  
B. Non-preemptive  
C. A real-time deadline test  
D. A page-replacement algorithm  

**Answer:** B  

**Explanation:** The running process keeps the CPU until it blocks or completes.

**Q3. The convoy effect occurs when:**

A. A long process makes short processes wait behind it  
B. Every process has burst time 1  
C. The CPU is faster than the ready queue  
D. Two threads share a stack  

**Answer:** A  

**Explanation:** Short processes are delayed for the whole remaining burst of the long process in front.

**Q4. For processes with bursts 24, 3, and 3, arriving together and served in that order, the average waiting time is:**

A. 0  
B. 3  
C. 17  
D. 24  

**Answer:** C  

**Explanation:** Waiting times are 0, 24, and 27. The average is 51/3 = 17.

**Q5. Under FCFS, response time equals waiting time because:**

A. Each process, once started, runs its burst without preemption  
B. Arrival time is always 0  
C. Burst time is not used in TAT  
D. The Gantt chart is circular  

**Answer:** A  

**Explanation:** The process does not return to the ready queue before finishing, so the first wait is the only wait.

**Q6. If the CPU is free at time 4 and the next process arrives at time 7, FCFS:**

A. Starts that process at time 4  
B. Leaves the CPU idle until time 7  
C. Runs the process for 4 time units before it arrives  
D. Deletes the process  

**Answer:** B  

**Explanation:** A process cannot be dispatched before its arrival time.

**Q7. The FCFS ready queue is a:**

A. FIFO queue  
B. Stack, so the newest process runs first  
C. Red-black tree ordered by virtual runtime  
D. Binary heap of deadlines  

**Answer:** A  

**Explanation:** The earliest arrival is at the head and runs next.

**Q8. FCFS can cause a long wait even though it does not starve a later process forever, because:**

A. A process may stand behind a very long burst  
B. The scheduler skips every second process permanently  
C. Short processes are always deleted  
D. Arrival time is ignored  

**Answer:** A  

**Explanation:** The process will eventually run, but only after everything ahead of it has taken its full burst.

### Quick revision

#### Rule

Earliest arrival runs next. Non-preemptive. FIFO ready queue.

#### Formulas

TAT = CT − AT, WT = TAT − BT, RT = start − AT = WT.

#### Convoy example

Bursts 24, 3, 3 in that order, all at time 0. Average waiting time = 17.

#### Common set used later for comparison

P1 (0, 5), P2 (1, 3), P3 (2, 8), P4 (3, 6).  
FCFS average TAT = 11.25, average WT = 5.75, average RT = 5.75.
