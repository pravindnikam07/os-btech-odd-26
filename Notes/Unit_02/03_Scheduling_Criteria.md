**Navigation:** [Unit index](00_Index.md) · [Previous: 2.2 Threads and multithreading models](02_Threads_and_Multithreading.md) · [Next: 2.4 FCFS](04_FCFS.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 2 — Processes, Threads and CPU Scheduling

## Topic 2.3 — CPU Scheduling Criteria

### Learning Outcomes

- State the job of the short-term scheduler and the dispatcher.
- Differentiate preemptive and non-preemptive scheduling.
- Define CPU utilization, throughput, turnaround time, waiting time, and response time.
- Compute the three time measures from arrival time, burst time, and a Gantt chart.

### Prerequisites

Process states, ready queue, and context switch (Topic 2.1).

### Introduction

Whenever more than one process is ready, the operating system must choose which one receives the CPU. The short-term scheduler makes that choice. The dispatcher then performs the context switch. Different algorithms optimise different measures: a batch system cares about throughput and turnaround time, and an interactive system cares about response time.

### Definition

**Definition:**  
CPU scheduling is the activity of selecting a ready process and assigning the CPU to it.

**Definition:**  
The short-term scheduler, or CPU scheduler, selects the next process from the ready queue. The dispatcher gives that process control of the CPU.

### Key terminology

| Term | Meaning |
| --- | --- |
| Burst time (BT) | CPU time the process needs in its current CPU burst |
| Arrival time (AT) | Time at which the process enters the ready queue |
| Completion time (CT) | Time at which the process finishes the burst under study |
| Turnaround time (TAT) | Completion time minus arrival time |
| Waiting time (WT) | Time spent in the ready queue |
| Response time (RT) | Time from arrival until the process first receives the CPU |
| Quantum | Maximum continuous CPU time given to a process before preemption |
| Dispatch latency | Time from the scheduler’s decision until the chosen process actually runs |
| Preemptive | The CPU can be taken from a running process |
| Non-preemptive | A running process keeps the CPU until it blocks or finishes |

### Scheduler and dispatcher

```text
Ready queue
    |
    v
Short-term scheduler  ---- selects the next process
    |
    v
Dispatcher
    |  1. switch context
    |  2. switch to user mode
    |  3. jump to the process's program counter
    v
Selected process runs
```

Dispatch latency is the time taken by the dispatcher. It is overhead. A long dispatch latency is especially harmful for interactive and real-time work.

The long-term scheduler, where one exists, controls how many processes are admitted into memory. The short-term scheduler runs much more often and chooses among processes that are already ready.

### Preemptive and non-preemptive scheduling

| Decision point | Non-preemptive | Preemptive |
| --- | --- | --- |
| A running process finishes or blocks | Scheduler runs | Scheduler runs |
| A new process arrives | Running process continues | Scheduler may replace the running process |
| A timer interrupt occurs | No timer preemption | Scheduler may replace the running process |
| Typical algorithms | FCFS, non-preemptive SJF, non-preemptive priority | SRTF, Round Robin, preemptive priority, CFS |

Non-preemptive scheduling is simpler. Preemptive scheduling is required for time-sharing, because one process must not be allowed to hold the CPU for the whole session.

### The five criteria

| Criterion | Meaning | Usual goal |
| --- | --- | --- |
| CPU utilization | Percentage of time the CPU is doing useful work | Keep it high |
| Throughput | Number of processes completed per unit time | Keep it high |
| Turnaround time | CT − AT, from arrival to completion | Keep it low |
| Waiting time | Time spent waiting in the ready queue | Keep it low |
| Response time | Time from arrival until the first CPU allocation | Keep it low |

**Formulas used in numerical problems:**

```text
Turnaround time = Completion time − Arrival time
Waiting time    = Turnaround time − Burst time
Response time   = Time of first CPU allocation − Arrival time
```

Waiting time equals turnaround time minus burst time when the burst time is the CPU time actually required. The process is either running or waiting in the ready queue during its turnaround interval, so the part that is not burst time is waiting time. This formula remains valid for preemptive algorithms. Adding the gaps on the Gantt chart gives the same waiting time.

For a non-preemptive algorithm the process runs only once, so response time and waiting time are equal. For a preemptive algorithm they differ: response time stops at the first dispatch, and waiting time includes every later stay in the ready queue.

CPU utilization and throughput are system-wide measures. Turnaround time, waiting time, and response time are computed per process and then averaged.

### Worked calculation from a Gantt chart

**Given:**  
P2 arrives at time 1 and has burst 3. On a Round Robin chart it runs during 2–4 and 10–11, and it completes at time 11.

**Turnaround time:**  
CT − AT = 11 − 1 = 10

**Waiting time:**  
TAT − BT = 10 − 3 = 7

Check by gaps. P2 is ready from time 1. It waits from 1 to 2 (1 unit) and from 4 to 10 (6 units). Total waiting time = 7. The two methods agree.

**Response time:**  
First dispatch is at time 2. RT = 2 − 1 = 1

### What each criterion favours

- High utilization and throughput favour keeping the CPU busy and finishing short work, but they do not by themselves describe an interactive user’s experience.
- Low turnaround time favours finishing each process soon after it arrives. Shortest-job methods aim at this.
- Low waiting time is the usual figure of merit in scheduling numericals.
- Low response time favours giving every interactive process a short turn quickly. Round Robin is designed for this, even when average waiting time is not the smallest.

An algorithm can improve one measure and worsen another. The comparison in the unit summary shows Round Robin producing a better average response time and a worse average waiting time than FCFS on the same processes.

### Common mistakes

- Using burst time as turnaround time. Turnaround time includes waiting.
- Computing waiting time as completion time minus burst time and forgetting to subtract the arrival time. The safe order is CT − AT first, then subtract BT.
- Treating response time as the time until the process finishes. Response time ends when the process first gets the CPU.
- Assuming response time equals waiting time for Round Robin or SRTF. That equality holds for non-preemptive runs only.
- Ignoring the priority convention or the tie rule printed in the question. Those rules change the Gantt chart.

### Important exam points

- Five criteria and whether each should be high or low.
- TAT = CT − AT, WT = TAT − BT, RT = first start − AT.
- Dispatcher steps and dispatch latency.
- Preemptive decision points: arrival and timer. Non-preemptive: only when the running process leaves the CPU voluntarily.
- For non-preemptive scheduling, RT = WT.

### University exam questions

#### 2-mark questions

1. Define throughput.
2. Write the formula for turnaround time.
3. Define response time.
4. What is dispatch latency?
5. Differentiate preemptive and non-preemptive scheduling in one or two lines.

#### 4/5-mark questions

1. Explain the five CPU scheduling criteria.
2. Explain the role of the short-term scheduler and the dispatcher.
3. Show, with a short Gantt chart, that response time and waiting time can differ.
4. Why does a time-sharing system need preemption?

#### 8/10-mark questions

1. Explain scheduling criteria with formulas. Show a complete calculation of CT, TAT, WT, and RT for one process from a Gantt chart.
2. Compare preemptive and non-preemptive scheduling. State at which events the scheduler is invoked in each case.

### Practice problems

#### Easy

1. If completion time is 16 and arrival time is 2, what is the turnaround time?
2. If turnaround time is 14 and burst time is 8, what is the waiting time?
3. A process arrives at 3 and first runs at 8. What is the response time?
4. Should waiting time be maximised or minimised?
5. Name the component that actually loads the selected process onto the CPU.

#### Medium

1. Explain why WT = TAT − BT.
2. Give one situation that calls the scheduler under non-preemptive scheduling, and one extra situation under preemptive scheduling.
3. Distinguish throughput and CPU utilization.
4. A non-preemptive process has WT = 6. What is its response time? Why?
5. Why is a large dispatch latency a problem for interactive use?

#### Hard

1. A process arrives at 1, runs from 4 to 6 and from 9 to 12, and has burst 5. Compute TAT, WT, and RT. Check WT by adding the ready-queue gaps.
2. Explain a case in which raising throughput is not the same goal as lowering response time.
3. Two processes have the same burst time. Why can their waiting times still differ?
4. Why is average waiting time alone a weak description of a Round Robin system?
5. The CPU is busy for 90 ms in a 100 ms interval and completes 5 processes. State the utilization and the throughput.

### MCQs

**Q1. Turnaround time is:**

A. Completion time minus arrival time  
B. Burst time minus arrival time, ignoring completion  
C. The quantum only  
D. Dispatch latency multiplied by the number of files  

**Answer:** A  

**Explanation:** TAT = CT − AT. It covers the whole stay from arrival to completion.

**Q2. Waiting time equals:**

A. Burst time  
B. Turnaround time minus burst time  
C. Arrival time plus completion time  
D. Response time plus burst time in every preemptive schedule  

**Answer:** B  

**Explanation:** Of the turnaround interval, the burst is the time spent running. The rest is waiting in the ready queue.

**Q3. Response time is measured until:**

A. The process completes  
B. The process first receives the CPU  
C. The process is created on disk  
D. The PCB is deleted  

**Answer:** B  

**Explanation:** Response time is about the first response, not about completion.

**Q4. The dispatcher:**

A. Admits new jobs from disk into the system for the long term only  
B. Switches context and starts the process selected by the scheduler  
C. Replaces the file system  
D. Computes the nice value of the disk  

**Answer:** B  

**Explanation:** The scheduler chooses. The dispatcher performs the switch and transfers control.

**Q5. Under non-preemptive scheduling, the scheduler is invoked when:**

A. The running process blocks or finishes  
B. Every new arrival must preempt the CPU  
C. A timer always removes the running process  
D. A ready process edits another process’s PCB  

**Answer:** A  

**Explanation:** The running process keeps the CPU until it leaves voluntarily by blocking or completing.

**Q6. For a non-preemptive schedule, response time equals waiting time because:**

A. The process runs only once, so the wait before the first run is the entire wait  
B. Burst time is always zero  
C. Arrival time is always equal to completion time  
D. The dispatcher never runs  

**Answer:** A  

**Explanation:** There is no later return to the ready queue. The first start is also the only start.

**Q7. CPU utilization is:**

A. The fraction of time the CPU is doing useful work  
B. The number of context switches stored in a file  
C. Always equal to response time  
D. The size of the PCB  

**Answer:** A  

**Explanation:** Utilization measures how busy the processor is.

**Q8. Throughput is:**

A. Processes completed per unit time  
B. The waiting time of the longest process only  
C. The number of registers in the PCB  
D. Dispatch latency  

**Answer:** A  

**Explanation:** Throughput counts completed processes, for example processes per second.

### Quick revision

#### Formulas

```text
TAT = CT − AT
WT  = TAT − BT
RT  = first CPU time − AT
```

#### Criteria

| Measure | Goal |
| --- | --- |
| CPU utilization | High |
| Throughput | High |
| Turnaround time | Low |
| Waiting time | Low |
| Response time | Low |

#### Preemption

- Non-preemptive: FCFS and non-preemptive SJF. The running process is not forced off the CPU.
- Preemptive: SRTF, Round Robin, preemptive priority. Arrival or the timer can cause a scheduling decision.
- Non-preemptive numericals: RT = WT. Preemptive numericals: compute RT and WT separately.
