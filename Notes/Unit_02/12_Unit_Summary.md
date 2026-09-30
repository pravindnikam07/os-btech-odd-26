**Navigation:** [Unit index](00_Index.md) · [Previous: 2.11 Linux CFS](11_Linux_CFS.md) · Next: —

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 2 — Processes, Threads and CPU Scheduling

# Unit summary

Unit 2 follows a program from the moment it becomes a process through to the decision that places it on a CPU. The first part is the process and the thread: states, the process control block, context switching, and the three multithreading models. The second part is CPU scheduling. The same four processes are used for FCFS, SJF, SRTF, and Round Robin so that the averages can be compared. Priority scheduling, multilevel queues, multicore placement, real-time deadlines, and Linux CFS are the remaining policies.

## Common process set

| Process | Arrival | Burst |
| --- | ---: | ---: |
| P1 | 0 | 5 |
| P2 | 1 | 3 |
| P3 | 2 | 8 |
| P4 | 3 | 6 |

| Algorithm | Gantt chart | Average TAT | Average WT | Average RT |
| --- | --- | ---: | ---: | ---: |
| FCFS | P1 0–5, P2 5–8, P3 8–16, P4 16–22 | 11.25 | 5.75 | 5.75 |
| SJF | P1 0–5, P2 5–8, P4 8–14, P3 14–22 | 10.75 | 5.25 | 5.25 |
| SRTF | P1 0–1, P2 1–4, P1 4–8, P4 8–14, P3 14–22 | 10.50 | 5.00 | 4.25 |
| RR, q = 2 | P1 0–2, P2 2–4, P3 4–6, P1 6–8, P4 8–10, P2 10–11, P3 11–13, P1 13–14, P4 14–16, P3 16–18, P4 18–20, P3 20–22 | 15.25 | 9.75 | 2.00 |

Round Robin has the best average response time on this set and the worst average waiting time. SRTF has the best average waiting time of the four. FCFS is simplest and suffers when a long burst blocks short ones.

A second, classic comparison, with every process ready at time 0:

| Order or algorithm | Bursts | Average waiting time |
| --- | --- | ---: |
| FCFS in the order 24, 3, 3 | 24, 3, 3 | 17 |
| SJF | 3, 3, 24 | 3 |
| Round Robin, quantum 4 | 24, 3, 3 | 17/3 ≈ 5.67 |

The drop from 17 to 3 is the convoy effect being removed.

## Topic table

| Topic | Key concept | Important formula or rule | Point to remember |
| --- | --- | --- | --- |
| Process | Program in execution; five states | Running returns to Ready on preemption; Waiting returns to Ready, not to Running | Context switch saves and restores the PCB and is overhead |
| PCB | Kernel record of one process | PID, state, PC, registers, scheduling, memory, accounting, open files | Queues store PCB pointers |
| Thread | Unit of execution inside a process | Share code, data, and files; private PC, registers, and stack | Many-to-one blocks the whole process; one-to-one does not |
| Criteria | How a schedule is judged | TAT = CT − AT, WT = TAT − BT, RT = first start − AT | Non-preemptive: RT = WT |
| FCFS | Earliest arrival, no preemption | FIFO queue | Convoy effect; average WT 17 for bursts 24, 3, 3 |
| SJF | Shortest burst among arrived processes | Non-preemptive | Average WT 3 for those bursts; long jobs can starve |
| SRTF | Shortest remaining time | Preempt only if the new burst is strictly smaller | On the common set, average WT = 5.00 |
| Round Robin | FIFO plus a quantum | Exhausted quantum → tail of the queue | Large quantum becomes FCFS; q = 2 gives average RT = 2.00 |
| Priority | Best priority runs | These notes: smaller number = higher priority | Standard example average WT = 8.2; aging prevents starvation |
| MLQ | Several queues, fixed class | Higher queue first; each queue has its own algorithm | A process does not move between queues |
| MLFQ | Queues with feedback | Enter at the top; full quantum → demote; long wait → promote | I/O-bound work stays high; CPU-bound work sinks |
| Multicore | One thread runs on one core | Soft affinity prefers the last core; hard affinity restricts the core | Push and pull migration balance per-core queues |
| RM | Fixed priority by period | Shorter period, higher priority; U ≤ n(2^(1/n) − 1) is sufficient | Two-task bound ≈ 0.828 |
| EDF | Dynamic priority by deadline | Nearest absolute deadline; U ≤ 1 when deadline = period | The set with U = 34/35 misses under RM and meets deadlines under EDF |
| CFS | Fair share for ordinary Linux tasks | time = L × weight / Σ weight; vruntime += Δ × 1024 / weight | Smallest vruntime, leftmost in a per-CPU red-black tree |

## Important definitions

- Process, PCB, context switch, thread.
- User-level thread, kernel-level thread, many-to-one, one-to-one, many-to-many.
- Turnaround time, waiting time, response time, dispatch latency.
- FCFS, SJF, SRTF, Round Robin, priority, aging.
- Multilevel queue, multilevel feedback queue.
- Soft affinity, hard affinity, push migration, pull migration.
- Rate Monotonic, Earliest Deadline First, virtual runtime.

## Important formulas

```text
TAT = CT − AT
WT  = TAT − BT
RT  = first CPU time − AT

RM:   U ≤ n (2^(1/n) − 1)          sufficient, deadline = period
EDF:  U ≤ 1                         deadline = period, one processor

CFS:  time_i = L × weight_i / Σ weight
      vruntime += Δ × (1024 / weight)
```

Nice 0 has weight 1024. For two tasks the RM bound is approximately 0.828. For many tasks it approaches ln 2 ≈ 0.693.

## Important diagrams

- Five-state process diagram, including Running → Ready.
- PCB fields and a ready queue of PCB pointers.
- User threads mapped many-to-one, one-to-one, and many-to-many.
- Gantt charts for FCFS, SJF, SRTF, and Round Robin.
- Three MLQ levels, and an MLFQ demotion path from Q0 to Q2.
- Push and pull between two cores.
- RM timeline for periods 50 and 100, and the RM miss at time 7 for tasks (2, 5) and (4, 7).
- CFS red-black tree with the leftmost node selected.

## Important comparisons

- Program and process; mode switch and context switch.
- User-level and kernel-level threads; the three mapping models.
- FCFS, SJF, SRTF, and Round Robin on the common set.
- Preemptive and non-preemptive priority; starvation and aging.
- MLQ and MLFQ.
- Soft and hard affinity; push and pull migration.
- RM and EDF.
- Round Robin and CFS.

## Examination priorities

- State diagram and PCB contents, with the context-switch steps.
- Shared and private thread resources, and what a blocking call does in many-to-one and one-to-one.
- A complete numerical table: Gantt chart, CT, TAT, WT, and the average. State the tie rule and, for priority, whether a smaller number is better.
- Convoy effect: average waiting time 17 against SJF average 3.
- Quantum too large and too small.
- Aging, in one sentence and with the direction of the priority change.
- MLFQ demotion rule.
- RM bound and one EDF timeline.
- CFS: smallest vruntime, and the 8 ms / 4 ms weight example.
