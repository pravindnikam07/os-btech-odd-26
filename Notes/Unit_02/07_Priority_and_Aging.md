**Navigation:** [Unit index](00_Index.md) · [Previous: 2.6 Round Robin](06_Round_Robin.md) · [Next: 2.8 Multilevel queue and multilevel feedback queue](08_MLQ_and_MLFQ.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 2 — Processes, Threads and CPU Scheduling

## Topic 2.7 — Priority Scheduling and Aging

### Learning Outcomes

- Apply priority scheduling under a stated convention.
- Differentiate preemptive and non-preemptive priority scheduling.
- Explain starvation and how aging prevents it.

### Prerequisites

Preemption, ready queue, and waiting time (Topic 2.3).

### Introduction

Priority scheduling assigns each process a priority and gives the CPU to the ready process with the best priority. The priority may come from the type of work, a user request, or the operating system itself. The rule is easy to state and easy to misuse: if high-priority processes keep arriving, a low-priority process may never run. Aging raises the priority of a process that has been waiting, so that it eventually becomes eligible.

### Definition

**Definition:**  
Priority scheduling selects the ready process with the highest priority. It may be preemptive or non-preemptive.

**Convention:**  
A **smaller priority number means a higher priority**. Priority 1 runs before priority 2. If a question uses the opposite convention, follow the question. The Gantt chart is wrong if the convention is reversed.

**Definition:**  
Aging is a technique that improves the priority of a process as its waiting time increases, so that a low-priority process eventually runs.

### How it works

```text
Ready processes with priorities
        ↓
Select the best priority among those that have arrived
        ↓
Non-preemptive: run it until it blocks or finishes
Preemptive: if a better priority arrives, switch immediately
        ↓
A waiting low-priority process is aged
        ↓
Its priority improves until it is selected
```

```text
Algorithm: Non-preemptive priority

Step 1: When the CPU is free, consider processes with arrival time ≤ clock.
Step 2: Select the smallest priority number.
Step 3: If numbers are equal, select the earlier arrival.
Step 4: Run the process for its whole burst.
Step 5: Repeat.
```

```text
Algorithm: Preemptive priority

Step 1: At each arrival, compare the new priority with the running process.
Step 2: If the new priority number is strictly smaller, preempt.
Step 3: When the CPU is free, select the smallest priority number in the ready queue.
Step 4: Continue until every process completes.
```

```text
Algorithm: Aging (smaller number = higher priority)

Step 1: Assign each process an initial priority number.
Step 2: After every fixed waiting interval, decrease the priority number of a waiting process by 1, down to the best allowed number.
Step 3: Schedule by the updated numbers.
Step 4: A process that has waited long enough reaches the highest priority and is selected.
```

If the system uses a larger number as higher priority, aging adds to the number instead of subtracting. The idea is the same: waiting improves the chance of being selected.

### Example — non-preemptive, all processes ready at time 0

| Process | Burst | Priority |
| --- | ---: | ---: |
| P1 | 10 | 3 |
| P2 | 1 | 1 |
| P3 | 2 | 4 |
| P4 | 1 | 5 |
| P5 | 5 | 2 |

Priority 1 is the best, so the order is P2, P5, P1, P3, P4.

```text
|P2|   P5   |      P1      | P3 |P4|
0  1        6              16   18  19
```

| Process | Priority | BT | Start | CT | TAT | WT |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| P1 | 3 | 10 | 6 | 16 | 16 | 6 |
| P2 | 1 | 1 | 0 | 1 | 1 | 0 |
| P3 | 4 | 2 | 16 | 18 | 18 | 16 |
| P4 | 5 | 1 | 18 | 19 | 19 | 18 |
| P5 | 2 | 5 | 1 | 6 | 6 | 1 |

Average WT = (6 + 0 + 16 + 18 + 1) / 5 = 41 / 5 = **8.2**  
Average TAT = (16 + 1 + 18 + 19 + 6) / 5 = 60 / 5 = **12**

P4 has the poorest priority and waits 18 time units even though its burst is only 1. If processes of priority 1 kept arriving, P4 would wait still longer. That is starvation.

### Example — preemptive priority

| Process | Arrival | Burst | Priority |
| --- | ---: | ---: | ---: |
| P1 | 0 | 8 | 3 |
| P2 | 1 | 4 | 1 |
| P3 | 2 | 2 | 2 |

**Trace:**

1. P1 starts at 0. Its priority number is 3.
2. At time 1, P2 arrives with priority 1, which is better. P1 has used 1 and has 7 left. P2 preempts and runs from 1 to 5.
3. At time 2, P3 arrives with priority 2. P2’s priority 1 is still better, so P2 continues.
4. At time 5, P2 finishes. Ready processes are P1 (priority 3, remaining 7) and P3 (priority 2, remaining 2). P3 runs from 5 to 7.
5. P1 runs from 7 to 14.

```text
|P1|   P2   | P3 |     P1      |
0  1        5    7             14
```

| Process | CT | TAT | WT | RT |
| --- | ---: | ---: | ---: | ---: |
| P1 | 14 | 14 | 6 | 0 |
| P2 | 5 | 4 | 0 | 0 |
| P3 | 7 | 5 | 3 | 3 |

Average WT = (6 + 0 + 3) / 3 = **3**

P1’s response time is 0 and its waiting time is 6, because it was preempted and waited from 1 to 7.

### Aging

Consider the non-preemptive example and suppose a new priority-1 process arrives whenever the CPU becomes free. P4, with priority 5, never becomes the best process.

Aging changes P4’s number while it waits. Using “after every 4 time units of waiting, decrease the priority number by 1”:

| Waiting time of P4 | Priority number |
| ---: | ---: |
| 0 | 5 |
| 4 | 4 |
| 8 | 3 |
| 12 | 2 |
| 16 | 1 |

Once P4’s number reaches the range of the processes that have been occupying the CPU, the scheduler can select it. The interval and the step size are system parameters. The examination point is the direction of the change and the reason: a waiting process must not remain at a poor priority forever.

Aging is also used inside multilevel feedback queues, where a process that stays too long in a low queue is moved to a higher queue.

### Comparison

| Parameter | Non-preemptive priority | Preemptive priority |
| --- | --- | --- |
| When the choice is made | When the CPU becomes free | Also when a better priority arrives |
| Running process | Keeps the CPU until it leaves | Loses the CPU to a strictly better priority |
| Starvation | Possible | Possible |
| Remedy | Aging | Aging |
| Response of an important arrival | Waits for the current burst to finish | Can run as soon as it arrives |

SJF is a special case of priority scheduling in which the priority is the next burst length: a shorter burst has a better priority. SRTF is the preemptive version of that special case.

### Advantages and limitations

| Advantages | Limitations |
| --- | --- |
| Important work can be marked and run first | Low-priority processes can starve |
| The policy matches operating-system needs, such as giving a kernel daemon a better priority than a background job | Priority assignment can be arbitrary if it is not defined carefully |
| Aging is a simple correction for starvation | Aging parameters must be chosen; too fast and priorities become meaningless, too slow and starvation remains for a long time |
| Preemptive form reacts quickly to an urgent arrival | Preemption adds context switches |

### Common mistakes

- Assuming a larger number is always better. These notes use a smaller number as higher priority, and a question may do the opposite.
- Forgetting aging’s direction. If a smaller number is better, aging decreases the number.
- Drawing starvation as a deadlock. Starvation means a ready process waits indefinitely because the scheduler never selects it. Other processes can still run. Deadlock is a different problem, studied in Unit 3.
- Applying preemption in a question that says non-preemptive priority.

### Important exam points

- State the convention before the Gantt chart: here, priority 1 is better than priority 2.
- Standard five-process example: average waiting time 8.2.
- Starvation: a low priority never selected while better priorities keep arriving.
- Aging: waiting improves the priority until the process is selected.
- SJF is priority scheduling with priority equal to burst length.

### University exam questions

#### 2-mark questions

1. Define priority scheduling.
2. What is starvation in CPU scheduling?
3. What is aging?
4. Is SJF related to priority scheduling? State how.

#### 4/5-mark questions

1. Explain non-preemptive priority scheduling with the standard five-process example.
2. Explain starvation and aging with a numerical change in the priority number.
3. Differentiate preemptive and non-preemptive priority scheduling.

#### 8/10-mark questions

1. Schedule a given priority table. State the convention, draw the Gantt chart, and compute average waiting time. Explain whether any process is close to starvation.
2. Explain priority scheduling, starvation, and aging. Show a preemptive trace in which an important process arrives while another process is running.

### Practice problems

#### Easy

1. If a smaller number means a higher priority, which runs first: priority 2 or priority 5?
2. Does a non-preemptive priority scheduler switch when a better priority arrives?
3. What problem does aging solve?
4. If aging uses a larger number as higher priority, does the waiting process’s number increase or decrease?
5. Which process runs first in the five-process example?

#### Medium

1. Compute P4’s waiting time in the five-process example and explain why it is large.
2. Trace the preemptive example if P3’s priority is 1 and P2’s priority is 2. Arrivals and bursts stay the same.
3. Age a process from priority 6 toward priority 1 by 1 every 5 time units. After how long does it reach priority 1?
4. Why is the average waiting time 8.2 in the five-process example? Show the five waiting times.
5. Explain SJF as a priority rule in one sentence.

#### Hard

1. For the modified preemptive trace in medium question 2, draw the Gantt chart and compute average waiting time.
2. A priority-1 process of burst 2 arrives every 2 time units, starting at time 0. A priority-4 process of burst 3 is ready at time 0. Without aging, when does the priority-4 process run?
3. Apply aging to that priority-4 process: decrease its number by 1 after every 2 time units of waiting. When does it become priority 1?
4. Compare the preemptive priority trace in the notes with SRTF. What is being compared in each algorithm?
5. Why can two processes with the same priority need a second rule, and what second rule is used here?

**Answers**

- Five waiting times: 6, 0, 16, 18, 1. Sum 41. Average 8.2.
- Aging from 6 to 1 is five steps. At 5 time units per step, the number becomes 1 after 25 time units of waiting.
- Modified preemptive trace: P1 runs 0–1. At time 1, P2 has priority 2, which is better than P1’s priority 3, so P2 preempts. At time 2, P3 has priority 1, which is better than P2, so P3 runs 2–4. P2 has 3 left and runs 4–7. P1 has 7 left and runs 7–14.  
  Gantt: P1 0–1, P2 1–2, P3 2–4, P2 4–7, P1 7–14.  
  P1: TAT = 14, WT = 14 − 8 = 6.  
  P2: TAT = 7 − 1 = 6, WT = 6 − 4 = 2.  
  P3: TAT = 4 − 2 = 2, WT = 0.  
  Average WT = (6 + 2 + 0) / 3 = 8/3 ≈ 2.67.
- Without aging, a priority-1 burst of 2 arriving every 2 time units keeps the CPU forever. The priority-4 process does not run.
- Its number falls by 1 every 2 time units: 4, then 3, then 2, then 1. It reaches priority 1 after 6 time units of waiting. Whether it then runs depends on the priority-1 arrivals still present; the point of the question is the time at which its number becomes 1.

The medium question 2 answer is included above and checked.

### MCQs

**Q1. In the convention used here, the process selected is the ready process with:**

A. The smallest priority number  
B. The largest priority number  
C. The largest PID only  
D. The largest file size  

**Answer:** A  

**Explanation:** Priority 1 is higher than priority 2. A question that reverses the convention must be followed instead.

**Q2. Starvation in priority scheduling means:**

A. A ready low-priority process may never be selected  
B. Two processes each hold a lock the other needs  
C. The CPU utilization is exactly 100 percent by definition  
D. The quantum is equal to the burst  

**Answer:** A  

**Explanation:** The process is ready, but higher-priority work keeps taking the CPU. This is not the definition of deadlock.

**Q3. Aging prevents starvation by:**

A. Improving the priority of a process as it waits  
B. Deleting low-priority processes  
C. Turning the scheduler into FCFS with no priorities  
D. Freezing the priority numbers forever  

**Answer:** A  

**Explanation:** After enough waiting, the process becomes competitive with higher-priority work.

**Q4. If a smaller number is a better priority, aging:**

A. Decreases the priority number of a waiting process  
B. Increases that number  
C. Sets every priority to 0 at arrival and never changes it  
D. Swaps burst time with arrival time  

**Answer:** A  

**Explanation:** Decreasing the number moves the process toward priority 1.

**Q5. The average waiting time of the five-process non-preemptive example is:**

A. 8.2  
B. 12  
C. 19  
D. 0  

**Answer:** A  

**Explanation:** Waiting times 6, 0, 16, 18, and 1 average to 41/5 = 8.2. The value 12 is the average turnaround time.

**Q6. SJF can be viewed as priority scheduling in which the priority is:**

A. The length of the next CPU burst  
B. The process name in alphabetical order only  
C. The number of open files  
D. The size of the PCB  

**Answer:** A  

**Explanation:** A shorter burst is treated as a better priority.

**Q7. Preemptive priority scheduling switches when:**

A. A newly arrived process has a strictly better priority than the running process  
B. Any process arrives, regardless of priority  
C. The running process has a better priority than the arrival  
D. The clock reaches the burst of an unrelated completed process  

**Answer:** A  

**Explanation:** Only a better priority causes the preemption. An inferior arrival waits.

**Q8. In the five-process example, P2 runs first because:**

A. Its priority number is 1, the best in the set  
B. Its burst is the largest  
C. Its name is last in the alphabet  
D. It arrives at time 19  

**Answer:** A  

**Explanation:** All five are ready at time 0, and priority 1 outranks 2, 3, 4, and 5.

### Quick revision

#### Convention

Smaller priority number = higher priority, unless the question says otherwise.

#### Standard example

Order P2, P5, P1, P3, P4. Average WT = 8.2. Average TAT = 12.

#### Starvation and aging

Low priority may wait forever. Aging improves the priority of a waiting process. If a smaller number is better, the number decreases with waiting time.

#### Relations

- Non-preemptive priority does not switch in the middle of a burst.
- Preemptive priority switches for a strictly better arrival.
- SJF is priority by burst length. SRTF is its preemptive form.
