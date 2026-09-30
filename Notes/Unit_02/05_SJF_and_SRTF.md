**Navigation:** [Unit index](00_Index.md) · [Previous: 2.4 FCFS](04_FCFS.md) · [Next: 2.6 Round Robin](06_Round_Robin.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 2 — Processes, Threads and CPU Scheduling

## Topic 2.5 — SJF and SRTF

### Learning Outcomes

- Apply non-preemptive Shortest Job First.
- Apply Shortest Remaining Time First and show a preemption.
- Use one tie rule consistently.
- Compare both algorithms with FCFS on the same processes.

### Prerequisites

FCFS, Gantt charts, and the scheduling formulas (Topics 2.3 and 2.4).

### Introduction

Shortest Job First (SJF) selects the ready process whose next CPU burst is the shortest. Giving the CPU to a short process first reduces the time that other processes wait behind it, which is why SJF removes the convoy effect of FCFS. The preemptive form is called Shortest Remaining Time First (SRTF). It is also called preemptive SJF.

Both algorithms need the length of the next CPU burst. In a numerical problem that length is given. On a real system it must be predicted from previous bursts, often by an exponential average. The scheduling rule below assumes the length is known.

### Definition

**Definition:**  
SJF is a non-preemptive algorithm that, when the CPU is free, selects the ready process with the smallest next CPU burst.

**Definition:**  
SRTF is the preemptive form of SJF. If a new process arrives whose burst is strictly smaller than the remaining time of the running process, the running process is preempted.

### Tie rule

If two ready processes have the same burst, or the same remaining time, choose the one that arrived earlier. If they also arrived together, keep the order written in the question. Under SRTF, do not preempt when the new remaining time is only equal to the current remaining time.

### How SJF works

```text
CPU becomes free
        ↓
Look only at processes that have already arrived
        ↓
Select the one with the smallest burst
        ↓
Run it until it completes or blocks
        ↓
Repeat
```

```text
Algorithm: Non-preemptive SJF

Step 1: Start the clock at 0.
Step 2: Form the ready set: processes with arrival time ≤ clock that have not finished.
Step 3: If the ready set is empty, advance the clock to the next arrival and go to Step 2.
Step 4: Choose the process with the smallest burst. Break ties by earlier arrival.
Step 5: Run it for its entire burst. Record start and completion.
Step 6: Repeat until every process has finished.
Step 7: TAT = CT − AT, WT = TAT − BT, RT = start − AT.
```

### How SRTF works

```text
A process is running with remaining time R
        ↓
A new process arrives with burst B
        ↓
If B < R, preempt and run the new process
If B ≥ R, continue the current process
        ↓
When a process finishes, choose the ready process
with the smallest remaining time
```

```text
Algorithm: SRTF

Step 1: Keep the remaining time of every process. Initially it equals the burst time.
Step 2: At every arrival and every completion, select the ready process with the smallest remaining time.
Step 3: Run it until the next arrival or until it completes, whichever comes first.
Step 4: Subtract the executed time from its remaining time.
Step 5: If a new remaining burst is strictly smaller, switch.
Step 6: Completion time is the time at which remaining time becomes 0.
```

### Example — SJF on the common set

| Process | Arrival | Burst |
| --- | ---: | ---: |
| P1 | 0 | 5 |
| P2 | 1 | 3 |
| P3 | 2 | 8 |
| P4 | 3 | 6 |

**Step 1:**  
At time 0 only P1 has arrived. P1 runs from 0 to 5. It is not preempted.

**Step 2:**  
At time 5 the ready processes are P2 (burst 3), P3 (burst 8), and P4 (burst 6). The shortest is P2. P2 runs from 5 to 8.

**Step 3:**  
At time 8 the remaining work is P4 (6) and P3 (8). P4 runs from 8 to 14.

**Step 4:**  
P3 runs from 14 to 22.

**Gantt chart:**

```text
|  P1  | P2 |   P4   |    P3    |
0      5    8        14         22
```

| Process | AT | BT | CT | TAT | WT | RT |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| P1 | 0 | 5 | 5 | 5 | 0 | 0 |
| P2 | 1 | 3 | 8 | 7 | 4 | 4 |
| P3 | 2 | 8 | 22 | 20 | 12 | 12 |
| P4 | 3 | 6 | 14 | 11 | 5 | 5 |

Average TAT = (5 + 7 + 20 + 11) / 4 = 43 / 4 = **10.75**  
Average WT = (0 + 4 + 12 + 5) / 4 = 21 / 4 = **5.25**  
Average RT = **5.25**

FCFS on this same set had average waiting time 5.75. SJF reduces it to 5.25 by running P4 before the longer P3.

### Example — SRTF on the same set

**Step 1:**  
P1 runs from 0. At time 1, P1 has remaining time 4. P2 arrives with burst 3, and 3 < 4, so P1 is preempted.

**Step 2:**  
P2 runs from 1. At time 2, P3 arrives with burst 8. P2 has remaining time 2, which is still the smallest, so P2 continues. At time 3, P4 arrives with burst 6. P2 has remaining time 1, so P2 continues and finishes at time 4.

**Step 3:**  
Ready work: P1 remaining 4, P4 remaining 6, P3 remaining 8. P1 runs from 4 to 8 and finishes.

**Step 4:**  
P4 (6) is shorter than P3 (8). P4 runs from 8 to 14. P3 runs from 14 to 22.

**Gantt chart:**

```text
|P1| P2 |  P1  |   P4   |    P3    |
0  1    4      8        14         22
```

| Process | AT | BT | CT | TAT | WT | RT |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| P1 | 0 | 5 | 8 | 8 | 3 | 0 |
| P2 | 1 | 3 | 4 | 3 | 0 | 0 |
| P3 | 2 | 8 | 22 | 20 | 12 | 12 |
| P4 | 3 | 6 | 14 | 11 | 5 | 5 |

**Check for P1:**  
P1 runs 0–1 and 4–8. It waits from 1 to 4, so WT = 3. TAT = 8 − 0 = 8. WT = 8 − 5 = 3. RT = 0, because it started at time 0. Response time and waiting time are no longer equal.

Average TAT = (8 + 3 + 20 + 11) / 4 = 42 / 4 = **10.50**  
Average WT = (3 + 0 + 12 + 5) / 4 = 20 / 4 = **5.00**  
Average RT = (0 + 0 + 12 + 5) / 4 = 17 / 4 = **4.25**

### Example — all processes present at time 0

| Process | Burst |
| --- | ---: |
| P1 | 24 |
| P2 | 3 |
| P3 | 3 |

SJF and SRTF are the same when every process is ready at time 0 and no new process arrives: there is never a later, shorter arrival to cause a preemption.

P2 and P3 both have burst 3. P2 is written first, so P2 runs, then P3, then P1.

```text
| P2 | P3 |            P1            |
0    3    6                          30
```

Waiting times: P2 = 0, P3 = 3, P1 = 6. Average = (0 + 3 + 6) / 3 = **3**.

The same processes under FCFS in the order P1, P2, P3 had average waiting time 17. SJF gives 3.

### Why SJF minimises average waiting time

Suppose a short process stands behind a long one. Every process behind the short one, including the short one itself, waits for the long burst. Moving the short process forward removes that long wait from several processes and adds only a short wait to the long process. Among algorithms that run a selected burst to completion, choosing the shortest ready burst gives the minimum average waiting time, provided the burst times are known and all the compared processes are in the ready set.

This optimality does not remove two practical difficulties. The next burst must be known or estimated, and a continuing stream of short processes can make a long process wait for a very long time. That is starvation of the long process. Aging, discussed with priority scheduling, is the usual remedy when starvation matters.

### Comparison

| Parameter | SJF | SRTF |
| --- | --- | --- |
| Preemption | No | Yes, when a new burst is strictly shorter than the remaining time |
| Decision | When the CPU becomes free | Also when a process arrives |
| Information used | Full next burst | Remaining time |
| Response and waiting time | Equal | Often different |
| Average waiting time | Optimal among non-preemptive policies when bursts are known | Can be still lower, because a short arrival need not wait for the rest of a long burst |
| Overhead | No switch in the middle of a burst | More context switches |

On the common set:

| Algorithm | Average WT | Average RT |
| --- | ---: | ---: |
| FCFS | 5.75 | 5.75 |
| SJF | 5.25 | 5.25 |
| SRTF | 5.00 | 4.25 |

### Advantages and limitations

| Advantages | Limitations |
| --- | --- |
| Low average waiting time when bursts are known | The next burst is not known exactly in a real system |
| Avoids the convoy effect | Long processes can starve |
| SRTF reacts to a short arrival immediately | SRTF adds context switches |
| Simple to apply in examination problems | A wrong tie rule changes the Gantt chart |

### Common mistakes

- Preempting under SJF. If the question says SJF and does not say preemptive, run the selected process to completion.
- Including a process that has not arrived yet in the ready set.
- Preempting when the new burst is equal to the remaining time. These notes preempt only when it is strictly smaller.
- Using the original burst instead of the remaining time after a process has already run.
- Forgetting that SJF and SRTF are identical if all processes arrive at time 0.

### Important exam points

- SJF: non-preemptive, shortest burst among arrived processes.
- SRTF: preemptive, shortest remaining time, preempt only if strictly shorter.
- Common-set averages: SJF waiting time 5.25, SRTF waiting time 5.00.
- Classic bursts 24, 3, 3: FCFS average wait 17, SJF average wait 3.
- Optimality holds when burst times are known. Starvation of long jobs is the matching limitation.

### University exam questions

#### 2-mark questions

1. Define SJF.
2. Define SRTF.
3. State one difference between SJF and SRTF.
4. Why can a long process starve under SJF?

#### 4/5-mark questions

1. Explain non-preemptive SJF with a Gantt chart.
2. Explain SRTF and show one preemption.
3. Compare FCFS and SJF using processes of burst 24, 3, and 3.

#### 8/10-mark questions

1. For a given arrival-time table, draw SJF and SRTF Gantt charts and compute average waiting time for both.
2. Explain why SJF tends to minimise average waiting time, and state the practical difficulties of using it.

### Practice problems

#### Easy

1. Is standard SJF preemptive?
2. What is compared under SRTF: original burst or remaining time?
3. All processes are ready at time 0. Will SJF and SRTF produce the same chart?
4. Two ready processes have bursts 4 and 4. Which process is selected if the earlier arrival is chosen?
5. Which process is selected at time 0 if only one process has arrived?

#### Medium

1. Draw the SJF chart for P1 (0, 4), P2 (1, 3), P3 (2, 1), P4 (3, 2).
2. Draw the SRTF chart for the same processes. Preempt only when the new burst is strictly smaller.
3. Compute average waiting time for both charts.
4. Explain one preemption in that SRTF chart.
5. Why is P1 not preempted at time 1 in that SRTF chart?

#### Hard

1. Complete CT, TAT, WT, and RT for every process in the medium SJF and SRTF questions.
2. A running process has remaining time 5. A process of burst 5 arrives. Does SRTF, as defined here, preempt? What if the new burst is 4?
3. Show the starvation scenario: a process of burst 100 is ready, and every 1 time unit a new process of burst 1 arrives. What happens under SRTF?
4. Compare average waiting time of FCFS and SJF for bursts 24, 3, 3, all arrived at 0, FCFS order P1, P2, P3.
5. Explain why the optimality argument needs known burst times.

**Answers**

- SJF Gantt: P1 0–4, P3 4–5, P4 5–7, P2 7–10. Average WT = 2.50. Average TAT = 5.00.
- At time 1, P1 has remaining time 3 and P2 arrives with burst 3. Remaining times are equal, so P1 is not preempted.
- At time 2, P1 has remaining time 2 and P3 arrives with burst 1. SRTF preempts.
- SRTF Gantt: P1 0–2, P3 2–3, P1 3–5, P4 5–7, P2 7–10. Average WT = 2.25. Average TAT = 4.75.
- SRTF details: WT of P1, P2, P3, P4 = 1, 6, 0, 2. RT = 0, 6, 0, 2.
- A new burst of 5 does not preempt a remaining time of 5. A new burst of 4 does.
- Under the continuous arrival of burst-1 processes, the burst-100 process never has the shortest remaining time, so it waits indefinitely.

### MCQs

**Q1. Non-preemptive SJF selects:**

A. The ready process with the smallest burst time  
B. The process that arrived last  
C. The process with the largest burst time  
D. The process with the earliest deadline, ignoring burst time  

**Answer:** A  

**Explanation:** SJF means shortest next CPU burst among processes that have arrived.

**Q2. SRTF preempts the running process when a new process has:**

A. A larger burst than the original burst of the running process  
B. A strictly smaller burst than the remaining time of the running process  
C. The same process name  
D. A later arrival time only  

**Answer:** B  

**Explanation:** The comparison is with remaining time. The running process is preempted only when the new burst is strictly shorter.

**Q3. For bursts 24, 3, and 3, all ready at time 0, SJF average waiting time is:**

A. 17  
B. 3  
C. 24  
D. 0  

**Answer:** B  

**Explanation:** The order is 3, then 3, then 24. Waiting times are 0, 3, and 6. The average is 3.

**Q4. SJF and SRTF produce the same schedule when:**

A. All processes are ready at time 0 and no shorter job arrives later  
B. Every burst is different and arrivals are spread out  
C. The quantum is 2  
D. The system is Round Robin  

**Answer:** A  

**Explanation:** Preemption in SRTF is caused by a later arrival with a shorter remaining requirement. If everyone is already ready, no such arrival occurs.

**Q5. A practical difficulty of SJF is that:**

A. The next CPU burst must be known or predicted  
B. The ready queue cannot be stored  
C. It is always preemptive  
D. It ignores arrival time and runs future processes early  

**Answer:** A  

**Explanation:** The algorithm’s choice depends on the length of the next burst.

**Q6. On the set P1 (0, 5), P2 (1, 3), P3 (2, 8), P4 (3, 6), SRTF runs P2 during:**

A. 0–3  
B. 1–4  
C. 5–8  
D. 8–11  

**Answer:** B  

**Explanation:** P2 arrives at 1 with burst 3, which is shorter than P1’s remaining time 4. P2 then runs until it completes at time 4.

**Q7. Long processes can starve under SJF because:**

A. Short processes keep being selected ahead of them  
B. FCFS forbids short processes  
C. The PCB cannot store a long burst  
D. Completion time is always 0  

**Answer:** A  

**Explanation:** Whenever a shorter ready process exists, the long process is not chosen.

**Q8. Under SRTF, response time and waiting time:**

A. Are always equal  
B. Can differ, because a process may be preempted and wait again  
C. Are not defined  
D. Are both equal to the quantum  

**Answer:** B  

**Explanation:** Response time ends at the first dispatch. Waiting time includes later waits after preemption.

### Quick revision

#### Rules

- SJF: shortest full burst among arrived processes; do not preempt.
- SRTF: shortest remaining time; preempt only if the new burst is strictly smaller.
- Tie: earlier arrival, then the order given in the question.

#### Common set

| Algorithm | Average WT | Average RT |
| --- | ---: | ---: |
| FCFS | 5.75 | 5.75 |
| SJF | 5.25 | 5.25 |
| SRTF | 5.00 | 4.25 |

#### Classic contrast

Bursts 24, 3, 3. FCFS average WT = 17. SJF average WT = 3.

#### Limitation

Burst lengths must be known. Long processes can starve.
