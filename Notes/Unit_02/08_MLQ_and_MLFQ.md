**Navigation:** [Unit index](00_Index.md) · [Previous: 2.7 Priority scheduling and aging](07_Priority_and_Aging.md) · [Next: 2.9 Multicore scheduling](09_Multicore_Scheduling.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 2 — Processes, Threads and CPU Scheduling

## Topic 2.8 — Multilevel Queue and Multilevel Feedback Queue

### Learning Outcomes

- Explain a multilevel queue and the way a higher queue is preferred.
- Explain a multilevel feedback queue and the movement of a process between queues.
- Differentiate a fixed assignment from a feedback assignment.
- Apply a small MLFQ trace with stated quanta.

### Prerequisites

Round Robin, FCFS, priority scheduling, and aging (Topics 2.6 and 2.7).

### Introduction

One ready queue and one rule are awkward when the system has different kinds of work. An interactive shell, a background compilation, and a system task do not deserve the same quantum. A multilevel queue keeps a separate queue for each class. A multilevel feedback queue goes further: a process can move from one queue to another according to the way it uses the CPU.

### Definition

**Definition:**  
A multilevel queue (MLQ) scheduler uses two or more ready queues. Each queue has its own scheduling algorithm, and a process remains in the queue assigned to its class.

**Definition:**  
A multilevel feedback queue (MLFQ) scheduler also uses several queues, but a process may move between them. The move depends on its CPU behaviour and on how long it has waited.

### Multilevel queue

A typical division is:

| Queue | Class of process | Algorithm inside the queue |
| --- | --- | --- |
| Q0, highest | System processes | Round Robin with a short quantum, or a fixed high priority |
| Q1 | Interactive processes | Round Robin |
| Q2, lowest | Batch or background processes | FCFS |

```text
Q0  System        RR, quantum 4     highest
Q1  Interactive   RR, quantum 8
Q2  Batch         FCFS              lowest
```

**How dispatch works:**

1. The class of a process, such as system or batch, fixes its queue when the process is created. It does not move.
2. Q0 is examined first. If any process is in Q0, the CPU is given to Q0’s own scheduler.
3. Q1 receives the CPU only when Q0 is empty.
4. Q2 receives the CPU only when Q0 and Q1 are empty.
5. A batch process can therefore wait for a long time if interactive work never stops. That is starvation of the lower queue. Aging or a reserved fraction of CPU time for each queue is used when that wait must be bounded.

Another arrangement gives each queue a percentage of the CPU, for example 50 percent to interactive work and 20 percent to batch work. The notes below use strict priority of queues, because that is the usual examination diagram. If the question states percentages, follow the question.

### Example — MLQ with strict queue priority

All three processes arrive at time 0.

| Process | Queue | Burst |
| --- | --- | --- |
| S1 | Q0, RR quantum 4 | 6 |
| I1 | Q1, RR quantum 4 | 5 |
| B1 | Q2, FCFS | 4 |

Q0 is never left while it has work. S1 runs 0–4, still has 2, and is the only process in Q0, so it runs 4–6 and finishes. Q0 is now empty. I1 runs 6–10, has 1 left, then runs 10–11 and finishes. Q1 is empty. B1 runs 11–15.

```text
|  S1  | S1 |   I1   |I1|  B1  |
0      4    6        10 11     15
```

| Process | CT | WT |
| --- | ---: | ---: |
| S1 | 6 | 0 |
| I1 | 11 | 6 |
| B1 | 15 | 11 |

B1 is ready at time 0 and receives no CPU until time 11, because both higher queues are served first. The waiting time is a property of the queue order, not of B1’s burst.

### Multilevel feedback queue

MLFQ tries to discover the kind of process by watching it.

**Rules:**

1. A new process enters the highest queue.
2. Each queue has a quantum. The lowest queue runs FCFS.
3. If a process uses a whole quantum, it is moved to the next lower queue.
4. If a process finishes, or blocks for I/O, before the quantum ends, it is not demoted. A process that blocks returns to the same queue when it becomes ready again.
5. The scheduler always runs a process from the highest queue that is not empty.
6. A process that has waited too long in a low queue is moved back to a higher queue. That is aging, and it limits starvation.

```text
New process
    |
    v
+--------+  used full quantum   +--------+  used full quantum   +------+
| Q0     | ------------------> | Q1     | ------------------> | Q2   |
| q = 4  |                     | q = 8  |                     | FCFS |
+--------+                     +--------+                     +------+
    ^                                                               |
    |                         waited too long                       |
    +---------------------------------------------------------------+
```

An interactive process or a process that blocks for I/O often leaves the CPU before the quantum ends, so it stays in a high queue and remains responsive. A CPU-bound process uses quantum after quantum and sinks to Q2, where it runs in longer stretches without disturbing the interactive queues.

### Example — MLFQ

Queues: Q0 quantum 4, Q1 quantum 8, Q2 FCFS.  
Both processes arrive at time 0 and are CPU-bound. P1 is placed in Q0 before P2.

| Process | Burst |
| --- | ---: |
| P1 | 18 |
| P2 | 5 |

**Trace:**

1. Both enter Q0. P1 runs 0–4, uses the quantum, and is demoted to Q1 with 14 left.
2. P2 runs 4–8, uses the quantum, and is demoted to Q1 with 1 left.
3. Q0 is empty. In Q1, P1 runs for a quantum of 8, from 8 to 16, and is demoted to Q2 with 6 left.
4. P2 has 1 left. It runs 16–17 and finishes. It does not need the whole Q1 quantum.
5. Q0 and Q1 are empty. P1 runs in Q2 from 17 to 23 and finishes.

```text
| P1 | P2 |     P1     |P2|   P1   |
0    4    8            16 17      23
     Q0   Q0           Q1  Q1     Q2
```

| Process | CT | TAT | WT |
| --- | ---: | ---: | ---: |
| P1 | 23 | 23 | 5 |
| P2 | 17 | 17 | 12 |

P1, the longer CPU-bound process, is pushed downward. P2 still waits through P1’s turns because P1 was ahead in each queue, but P2 finishes without ever entering Q2. If P2 had blocked after 1 ms of CPU, rule 4 would have kept it in the high queue.

### Comparison

| Parameter | Multilevel queue | Multilevel feedback queue |
| --- | --- | --- |
| Number of queues | Several | Several |
| Assignment | Fixed by process class | A new process starts at the top and may move |
| Algorithm in a queue | Chosen for that class: RR or FCFS | Usually RR in the upper queues and FCFS in the lowest |
| Response to CPU-bound behaviour | No automatic demotion | Demotion after a full quantum |
| Response to I/O-bound behaviour | Stays in its assigned queue | Stays high if it blocks before the quantum ends |
| Starvation of low queues | Possible if a higher queue never empties | Reduced by moving a long-waiting process upward |
| What must be known in advance | The class of the process | Not the class; the scheduler infers behaviour |

### Advantages and limitations

| Advantages | Limitations |
| --- | --- |
| Different classes of work can have different policies | MLQ needs a correct class for each process |
| Interactive work can be preferred without sorting every process by a single number | A strict higher-queue-first rule can starve batch work |
| MLFQ adapts when a process changes from interactive to CPU-bound | MLFQ has several parameters: number of queues, quanta, and the aging interval |
| Aging in MLFQ gives a long-waiting process another chance at a high queue | A badly sized quantum demotes interactive processes too quickly, or fails to demote CPU-bound ones |

### Common mistakes

- Saying a process in an MLQ changes queue when its quantum expires. That movement is MLFQ. In MLQ the queue is fixed.
- Serving a lower queue while a higher queue still has a ready process, under the strict-priority rule.
- Demoting a process that blocked for I/O before the quantum ended. Under the rules above it stays in its queue.
- Forgetting that each queue can have its own algorithm. The queues are not all the same Round Robin.

### Important exam points

- MLQ diagram: system, interactive, batch, with RR and FCFS inside the queues.
- Higher queue first, unless the question gives CPU percentages.
- MLFQ: enter at the top, demote after a full quantum, leave I/O-bound processes high, age a process that waits too long in a low queue.
- One numerical trace with the queue of each slice labelled.

### University exam questions

#### 2-mark questions

1. Define a multilevel queue scheduler.
2. Define a multilevel feedback queue.
3. Does a process change queue in MLQ?
4. Why is a CPU-bound process demoted in MLFQ?

#### 4/5-mark questions

1. Explain MLQ with a diagram of three queues.
2. State the MLFQ rules for demotion and for I/O.
3. Differentiate MLQ and MLFQ.

#### 8/10-mark questions

1. Explain multilevel queue and multilevel feedback queue with diagrams. Show how starvation can occur in MLQ and how MLFQ reduces it.
2. Trace an MLFQ schedule for two processes. State the quantum of each queue on the Gantt chart.

### Practice problems

#### Easy

1. Name a suitable algorithm for a batch queue.
2. Where does a new process enter under the MLFQ rules above?
3. A higher queue has one ready process. Does the batch queue run?
4. What happens to an MLFQ process that uses its whole quantum?
5. What is the fixed information that places a process in an MLQ queue?

#### Medium

1. Draw the MLQ example in this topic and state B1’s waiting time.
2. Explain why B1 waits even though its burst is only 4.
3. In the MLFQ example, why is P1 in Q2 at time 17?
4. A process in Q0 runs for 1 ms of a 4 ms quantum and then blocks for keyboard input. Is it demoted?
5. Give one way to stop permanent starvation of Q2 in an MLQ.

#### Hard

1. Add a second system process S2, burst 4, to the MLQ example, arriving at time 0 after S1 in Q0. Redraw the chart until B1 finishes.
2. Using the MLFQ rules, schedule P1 burst 10 and P2 burst 3, both ready at 0, P1 queued first. Q0 quantum 4, Q1 quantum 8, Q2 FCFS.
3. Explain a process that is demoted and later promoted. What observation causes each move?
4. Why can a single Round Robin queue not copy the MLQ result of the worked example?
5. Compare aging in priority scheduling with the upward move in MLFQ.

**Answers**

- B1’s waiting time in the worked MLQ example is 11.
- A process that blocks before the quantum ends is not demoted.
- Hard 1: S2 is also in Q0, so Round Robin alternates. S1 runs 0–4 and has 2 left. S2 runs 4–8 and finishes. S1 runs 8–10 and finishes. I1 runs 10–14 and has 1 left, then runs 14–15 and finishes. B1 runs 15–19. B1’s waiting time is 15.
- Hard 2: P1 runs 0–4 and moves to Q1 with 6 left. P2 runs 4–7 and finishes inside Q0, so it is not demoted. Q0 is empty. P1 runs 7–13 in Q1 and finishes with 2 ms of the quantum unused. CT of P2 = 7, CT of P1 = 13.

### MCQs

**Q1. In a multilevel queue, a process:**

A. Stays in the queue of its class  
B. Is demoted after every quantum by definition of MLQ  
C. Has no algorithm inside its queue  
D. Shares one stack with every other queue  

**Answer:** A  

**Explanation:** MLQ assigns the queue permanently. Movement between queues is the feedback queue.

**Q2. Under strict MLQ priority, the batch queue runs when:**

A. Every higher queue is empty  
B. The batch process has a shorter burst than a system process  
C. A system process is still ready  
D. The quantum of the batch queue is smaller than zero  

**Answer:** A  

**Explanation:** A higher non-empty queue is served first.

**Q3. In MLFQ, a process that consumes a full quantum is:**

A. Moved to a lower-priority queue  
B. Deleted  
C. Moved to a higher queue immediately  
D. Given the CPU forever  

**Answer:** A  

**Explanation:** Using the whole quantum is treated as CPU-bound behaviour and the process is demoted.

**Q4. An interactive process tends to stay in a high MLFQ queue because:**

A. It often blocks before the quantum expires  
B. It always has the longest CPU burst  
C. MLFQ forbids I/O  
D. It is placed directly in the FCFS queue at creation  

**Answer:** A  

**Explanation:** The rules demote a process that uses a full quantum. A process that blocks early is not demoted.

**Q5. Aging in MLFQ:**

A. Moves a process that has waited too long up to a higher queue  
B. Removes the highest queue  
C. Sets every quantum to the burst of the batch process  
D. Converts MLFQ into non-preemptive FCFS only  

**Answer:** A  

**Explanation:** The upward move is how a low queue avoids permanent starvation.

**Q6. MLQ and MLFQ both:**

A. Use more than one ready queue  
B. Change a process’s queue after every quantum  
C. Ignore the class of a system process  
D. Require deadline equals period  

**Answer:** A  

**Explanation:** Multiple queues are common to both. Only MLFQ moves processes according to feedback.

**Q7. A background compilation is most naturally placed, in an MLQ, in:**

A. The batch queue  
B. The highest system queue, with a quantum of one instruction  
C. No queue, because batch work is not scheduled  
D. The PCB of an unrelated interactive process  

**Answer:** A  

**Explanation:** Compilations are CPU-bound background work. The batch queue, often FCFS, is the matching class.

**Q8. In the MLFQ worked example, P2 finishes in:**

A. Q1  
B. Q2  
C. A new queue created at time 23  
D. The terminated queue of another process, without running  

**Answer:** A  

**Explanation:** P2 is demoted from Q0 to Q1 and completes during its short Q1 turn. It never reaches Q2.

### Quick revision

#### MLQ

Fixed queues. Example: system RR, interactive RR, batch FCFS. Higher queue runs first. A process does not move.

#### MLFQ rules

- Enter at the top.
- Full quantum → demote.
- Block before the quantum ends → stay.
- Always serve the highest non-empty queue.
- Long wait in a low queue → promote.

#### Worked results

- MLQ: S1 finishes at 6, I1 at 11, B1 at 15. B1 waits 11.
- MLFQ: P1 burst 18 finishes at 23 in Q2. P2 burst 5 finishes at 17 in Q1.
