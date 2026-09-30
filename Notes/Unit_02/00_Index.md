# Unit 2 — Processes, Threads and CPU Scheduling

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course:** BTech  
**Semester:** 3  
**Prerequisite:** Unit 1 (operating system services, dual mode, and the kernel)  
**Reference:** Operating System Concepts Essentials — Silberschatz, Galvin, Gagne (9th edition)

## Contents

| No. | Topic | File |
| --- | --- | --- |
| 2.1 | Processes, process states, PCB, and context switching | [01_Processes_PCB_and_Context_Switch.md](01_Processes_PCB_and_Context_Switch.md) |
| 2.2 | Threads and multithreading models | [02_Threads_and_Multithreading.md](02_Threads_and_Multithreading.md) |
| 2.3 | Scheduling criteria | [03_Scheduling_Criteria.md](03_Scheduling_Criteria.md) |
| 2.4 | FCFS | [04_FCFS.md](04_FCFS.md) |
| 2.5 | SJF and SRTF | [05_SJF_and_SRTF.md](05_SJF_and_SRTF.md) |
| 2.6 | Round Robin | [06_Round_Robin.md](06_Round_Robin.md) |
| 2.7 | Priority scheduling and aging | [07_Priority_and_Aging.md](07_Priority_and_Aging.md) |
| 2.8 | Multilevel queue and multilevel feedback queue | [08_MLQ_and_MLFQ.md](08_MLQ_and_MLFQ.md) |
| 2.9 | Multicore scheduling | [09_Multicore_Scheduling.md](09_Multicore_Scheduling.md) |
| 2.10 | Real-time scheduling: RM and EDF | [10_Real_Time_RM_and_EDF.md](10_Real_Time_RM_and_EDF.md) |
| 2.11 | Linux CFS | [11_Linux_CFS.md](11_Linux_CFS.md) |
| — | Unit summary | [12_Unit_Summary.md](12_Unit_Summary.md) |

## Topics covered

- Process fundamentals, states (new, ready, running, waiting, terminated), PCB, and context switching
- User-level and kernel-level threads; many-to-one, one-to-one, and many-to-many models
- Scheduling criteria: CPU utilization, throughput, turnaround time, waiting time, and response time
- FCFS, SJF, SRTF, Round Robin, and priority scheduling with aging
- Multilevel queue and multilevel feedback queue
- Multicore scheduling: load balancing and processor affinity
- Real-time scheduling: Rate Monotonic and Earliest Deadline First
- Linux Completely Fair Scheduler

The numerical examples in FCFS, SJF, SRTF, and Round Robin use one common process set, so the Gantt charts can be compared directly. That comparison is collected in the unit summary.
