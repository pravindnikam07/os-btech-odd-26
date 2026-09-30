**Navigation:** [Unit index](00_Index.md) · [Previous: 2.5 SJF and SRTF](05_SJF_and_SRTF.md) · [Next: 2.7 Priority scheduling and aging](07_Priority_and_Aging.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 2 — Processes, Threads and CPU Scheduling

## Topic 2.6 — Round Robin Scheduling

### Learning Outcomes

- Apply Round Robin for a given time quantum.
- Place a preempted process at the tail and place new arrivals at the tail.
- Show that a smaller quantum improves response time and increases context switches.
- Compare Round Robin with FCFS on waiting time and response time.

### Prerequisites

Ready queue, preemption, and the scheduling formulas (Topic 2.3). FCFS (Topic 2.4), because Round Robin with a very large quantum behaves like FCFS.

### Introduction

Round Robin (RR) is the scheduling algorithm used to give each interactive process a short turn on the CPU. The ready queue is FIFO, as in FCFS, but a process may run only for a time quantum. If it is still running when the quantum ends, it is preempted and placed at the tail of the queue. The next process at the head then runs. A timer interrupt enforces the quantum.

### Definition

**Definition:**  
Round Robin is a preemptive scheduling algorithm in which each ready process receives the CPU for at most one time quantum before the CPU passes to the next process in the FIFO ready queue.

### Queue rule

1. A process that arrives joins the tail.
2. If the running process exhausts the quantum and still has work left, it joins the tail after arrivals that occurred during that quantum have been placed in the queue.
3. If the running process finishes before the quantum ends, the CPU is given to the process at the head immediately. The unused part of the quantum is not saved.
4. If several processes are ready at time 0, they join in the order written in the question.

### How it works

```text
Ready queue (FIFO) ----> CPU runs the head process
                              |
                    +---------+----------+
                    |                    |
              finishes early       quantum expires
                    |                    |
               leaves the CPU     process joins the tail
                    |                    |
                    +------> next head --+
```

```text
Algorithm: Round Robin

Step 1: Put processes that have arrived into the FIFO queue in arrival order.
Step 2: Remove the process at the head.
Step 3: Run it for min(quantum, remaining burst).
Step 4: Add any processes that arrived during this run to the tail.
Step 5: If the process still has remaining burst, add it to the tail.
Step 6: Record the completion time when remaining burst becomes 0.
Step 7: Repeat until the queue is empty and no process is still to arrive.
```

### Example — quantum = 2 on the common set

| Process | Arrival | Burst |
| --- | ---: | ---: |
| P1 | 0 | 5 |
| P2 | 1 | 3 |
| P3 | 2 | 8 |
| P4 | 3 | 6 |

**Trace:**

| Time | Event | Queue after the decision |
| --- | --- | --- |
| 0 | P1 starts | empty |
| 1 | P2 arrives | P2 |
| 2 | P3 arrives; P1 has used its quantum and has 3 left | P2, P3, P1 |
| 2–4 | P2 runs and has 1 left; P4 arrived at 3 | P3, P1, P4, P2 |
| 4–6 | P3 runs and has 6 left | P1, P4, P2, P3 |
| 6–8 | P1 runs and has 1 left | P4, P2, P3, P1 |
| 8–10 | P4 runs and has 4 left | P2, P3, P1, P4 |
| 10–11 | P2 runs its last unit and finishes | P3, P1, P4 |
| 11–13 | P3 runs and has 4 left | P1, P4, P3 |
| 13–14 | P1 runs its last unit and finishes | P4, P3 |
| 14–16 | P4 runs and has 2 left | P3, P4 |
| 16–18 | P3 runs and has 2 left | P4, P3 |
| 18–20 | P4 finishes | P3 |
| 20–22 | P3 finishes | empty |

**Gantt chart:**

```text
|P1|P2|P3|P1|P4|P2| P3|P1|P4|P3|P4|P3|
0  2  4  6  8  10 11  13 14 16 18 20 22
```

| Process | AT | BT | CT | TAT | WT | RT |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| P1 | 0 | 5 | 14 | 14 | 9 | 0 |
| P2 | 1 | 3 | 11 | 10 | 7 | 1 |
| P3 | 2 | 8 | 22 | 20 | 12 | 2 |
| P4 | 3 | 6 | 20 | 17 | 11 | 5 |

**Check for P2:**  
P2 arrives at 1, runs 2–4 and 10–11, and finishes at 11.  
TAT = 11 − 1 = 10. WT = 10 − 3 = 7.  
Gaps: 1–2 and 4–10, which are 1 + 6 = 7. RT = 2 − 1 = 1.

Average TAT = (14 + 10 + 20 + 17) / 4 = 61 / 4 = **15.25**  
Average WT = (9 + 7 + 12 + 11) / 4 = 39 / 4 = **9.75**  
Average RT = (0 + 1 + 2 + 5) / 4 = 8 / 4 = **2.00**

FCFS on this set had average response time 5.75 and average waiting time 5.75. Round Robin with quantum 2 cuts the average response time to 2.00 and raises the average waiting time to 9.75. That is the usual trade: interactive processes get the CPU sooner, while processes that need a long continuous burst are sliced and wait more often.

### Example — quantum = 4, all processes ready at time 0

| Process | Burst |
| --- | ---: |
| P1 | 24 |
| P2 | 3 |
| P3 | 3 |

```text
| P1 | P2 | P3 | P1 | P1 | P1 | P1 | P1 | P1 |
0    4    7    10   14   18   22   26   30
```

P2 finishes during its first turn because 3 < 4. P3 does the same. After time 10 only P1 remains, so it runs in successive quanta until time 30.

| Process | CT | TAT | WT | RT |
| --- | ---: | ---: | ---: | ---: |
| P1 | 30 | 30 | 6 | 0 |
| P2 | 7 | 7 | 4 | 4 |
| P3 | 10 | 10 | 7 | 7 |

Average WT = (6 + 4 + 7) / 3 = 17 / 3 = **5.67**  
Average TAT = (30 + 7 + 10) / 3 = 47 / 3 = **15.67**

P1’s waiting time is only the interval 4–10, which is 6. After that it is alone in the queue.

### Effect of the quantum

| Quantum | Behaviour |
| --- | --- |
| Very large, larger than every burst | No process is preempted. Round Robin becomes FCFS |
| Very small, approaching one instruction | Response is very quick, but context-switch overhead dominates |
| A moderate value, often 10–100 ms on a general-purpose system | Each interactive process runs soon, and the fraction of time lost in switching stays acceptable |

The useful quantum is much larger than the context-switch time. If the switch takes 1 ms, a quantum of 1 ms spends about half the CPU on switching. A quantum of 10 ms spends a much smaller fraction. The exact value is a system choice; examination problems always state it.

Number of turns for a process is roughly the burst divided by the quantum, rounded up. Each extra turn can cause a context switch. A smaller quantum therefore increases the number of switches.

### Advantages and limitations

| Advantages | Limitations |
| --- | --- |
| Fair turn for every ready process | Average waiting time can be worse than SJF or even FCFS |
| Good response time for interactive work | The quantum must be chosen carefully |
| No process keeps the CPU for more than one quantum while others are waiting | Many context switches if the quantum is too small |
| Simple FIFO rule plus a timer | Does not use burst length, so it does not specially prefer a short job except by accident |

### Common mistakes

- Putting the preempted process at the head. It goes to the tail.
- Giving a finishing process the rest of the quantum as idle time. The next process starts at once.
- Forgetting arrivals that occur during the quantum. They join the tail before the preempted process if they arrived while it was running.
- Treating Round Robin as non-preemptive. The timer is the preemption mechanism.
- Using WT = RT. A process usually runs, waits, and runs again.

### Important exam points

- FIFO queue plus a time quantum. Preemptive.
- Common set, q = 2: average WT = 9.75, average RT = 2.00.
- Bursts 24, 3, 3 with q = 4: average WT = 17/3 ≈ 5.67.
- Large quantum → FCFS. Very small quantum → context-switch overhead.
- Response time improves because each process reaches the CPU within about one queue cycle.

### University exam questions

#### 2-mark questions

1. Define Round Robin scheduling.
2. What is a time quantum?
3. Where is a process placed when its quantum expires?
4. What does Round Robin become if the quantum is very large?

#### 4/5-mark questions

1. Explain Round Robin with a Gantt chart for a given quantum.
2. Discuss the effect of a very small and a very large quantum.
3. Compare FCFS and Round Robin on response time.

#### 8/10-mark questions

1. Trace Round Robin for a process set with arrivals. Draw the Gantt chart and compute average waiting time and average response time.
2. Explain Round Robin. Why is it suitable for time-sharing, and what is the cost of a poor quantum?

### Practice problems

#### Easy

1. Is Round Robin preemptive?
2. A process finishes in 3 ms and the quantum is 10 ms. When is the next process dispatched?
3. Name the interrupt that ends a quantum.
4. If the quantum is larger than every burst, which algorithm does RR resemble?
5. Does the preempted process stay at the head?

#### Medium

1. Draw RR with q = 2 for P1 (0, 4), P2 (1, 3), P3 (2, 1), P4 (3, 2).
2. Compute average waiting time and average response time for that chart.
3. Explain the position of P1 in the queue at time 2.
4. Why can average waiting time rise when the quantum becomes very small, even before counting switch overhead?
5. Compare one quantum of RR with the full burst of FCFS.

#### Hard

1. Using the medium chart, compute CT, TAT, WT, and RT for each process.
2. Repeat the common set with q = 4 and compare average RT with q = 2.
3. A context switch costs 0.5 ms and the quantum is 2 ms. Out of a 2.5 ms cycle of switch plus quantum, what fraction is overhead?
4. Explain why P2’s response time on the common set is 1 when q = 2.
5. Why is “fair” in Round Robin not the same statement as “minimum average waiting time”?

**Answers**

- Medium chart: P1 0–2, P2 2–4, P3 4–5, P1 5–7, P4 7–9, P2 9–10.  
  CT = P1 7, P2 10, P3 5, P4 9.  
  WT = 3, 6, 2, 4. Average WT = 3.75.  
  RT = 0, 1, 2, 4. Average RT = 1.75.
- Common set with q = 4: Gantt P1 0–4, P2 4–7, P3 7–11, P4 11–15, P1 15–16, P3 16–20, P4 20–22.  
  Average WT = 9.25. Average RT = 4.00. Quantum 2 had average RT 2.00, which is better, and average WT 9.75, which is worse.
- Overhead fraction = 0.5 / 2.5 = 0.2, or 20 percent. This cost is not included in the usual examination Gantt chart unless the question states it.
- P2 arrives at 1 and first runs at 2, so RT = 1.

### MCQs

**Q1. Round Robin gives each process the CPU for at most:**

A. One time quantum, then possibly another turn later  
B. Its entire burst, with no timer  
C. One instruction per hour only  
D. The burst of the longest process  

**Answer:** A  

**Explanation:** The quantum limits one continuous turn. The process may be scheduled again after other processes.

**Q2. When the quantum expires, the running process is placed:**

A. At the tail of the ready queue, if it still has work  
B. Permanently in the waiting state  
C. At the head, ahead of processes that were already waiting  
D. In the terminated state even if burst remains  

**Answer:** A  

**Explanation:** The queue is FIFO. A preempted process waits behind the processes already in the queue.

**Q3. If the time quantum is extremely large, Round Robin behaves like:**

A. FCFS  
B. SRTF  
C. Earliest Deadline First  
D. A random scheduler  

**Answer:** A  

**Explanation:** No burst finishes the quantum early enough to be preempted, so processes run in queue order to completion.

**Q4. A very small quantum increases:**

A. The number of context switches  
B. The burst time stored in the program file  
C. The arrival time  
D. The size of the PCB fields for open files  

**Answer:** A  

**Explanation:** Each process is interrupted more often, and each interruption can cause a switch.

**Q5. On the common set with quantum 2, the average response time is:**

A. 5.75  
B. 2.00  
C. 9.75  
D. 15.25  

**Answer:** B  

**Explanation:** Response times are 0, 1, 2, and 5. The average is 2. The value 9.75 is the average waiting time.

**Q6. For bursts 24, 3, and 3 with quantum 4, all arriving at 0, P2’s waiting time is:**

A. 0  
B. 4  
C. 24  
D. 7  

**Answer:** B  

**Explanation:** P2 waits from 0 to 4 and then runs to completion at 7. WT = 4.

**Q7. If a process completes halfway through its quantum:**

A. The next ready process starts immediately  
B. The CPU stays idle for the rest of the quantum  
C. The quantum is added to the next process’s burst  
D. The process is restarted from arrival time  

**Answer:** A  

**Explanation:** The quantum is a maximum, not a fixed reservation that must be wasted.

**Q8. Round Robin is suitable for time-sharing mainly because:**

A. Each ready process receives the CPU within one cycle of the queue  
B. It always minimises average waiting time  
C. It never uses a timer  
D. It selects the globally shortest job  

**Answer:** A  

**Explanation:** The bounded wait for a first turn keeps response time low. Average waiting time is not guaranteed to be the minimum.

### Quick revision

#### Rule

FIFO queue. Run the head for at most one quantum. Preempted process goes to the tail. New arrivals go to the tail.

#### Common set, q = 2

Average WT = 9.75. Average RT = 2.00. Average TAT = 15.25.

#### Quantum

- Large quantum → FCFS.
- Small quantum → better response, more context switches.
- Quantum should be large compared with context-switch time.

#### Contrast with FCFS on the common set

FCFS average RT = 5.75 and average WT = 5.75.  
RR (q = 2) average RT = 2.00 and average WT = 9.75.
