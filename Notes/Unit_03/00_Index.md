# Unit 3 — Process Synchronization and Deadlocks

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course:** BTech  
**Semester:** 3  
**Prerequisite:** Unit 2 (processes, threads, and the shared address space of threads)  
**Reference:** Operating System Concepts Essentials — Silberschatz, Galvin, Gagne (9th edition)

## Contents

| No. | Topic | File |
| --- | --- | --- |
| 3.1 | Critical section, race condition, and mutual exclusion | [01_Critical_Section_and_Race_Condition.md](01_Critical_Section_and_Race_Condition.md) |
| 3.2 | Hardware solutions: test-and-set and compare-and-swap | [02_Test_and_Set_and_Compare_and_Swap.md](02_Test_and_Set_and_Compare_and_Swap.md) |
| 3.3 | Peterson’s solution | [03_Petersons_Solution.md](03_Petersons_Solution.md) |
| 3.4 | Semaphores | [04_Semaphores.md](04_Semaphores.md) |
| 3.5 | Classical problems | [05_Classical_Problems.md](05_Classical_Problems.md) |
| 3.6 | Monitors and condition variables | [06_Monitors_and_Condition_Variables.md](06_Monitors_and_Condition_Variables.md) |
| 3.7 | Mutexes and spinlocks | [07_Mutexes_and_Spinlocks.md](07_Mutexes_and_Spinlocks.md) |
| 3.8 | Atomic operations and lock-free algorithms | [08_Atomic_and_Lock_Free.md](08_Atomic_and_Lock_Free.md) |
| 3.9 | Deadlock and the four necessary conditions | [09_Deadlock_and_Necessary_Conditions.md](09_Deadlock_and_Necessary_Conditions.md) |
| 3.10 | Deadlock prevention | [10_Deadlock_Prevention.md](10_Deadlock_Prevention.md) |
| 3.11 | Deadlock avoidance: Banker’s algorithm | [11_Bankers_Algorithm.md](11_Bankers_Algorithm.md) |
| 3.12 | Deadlock detection and recovery | [12_Deadlock_Detection_and_Recovery.md](12_Deadlock_Detection_and_Recovery.md) |
| — | Unit summary | [13_Unit_Summary.md](13_Unit_Summary.md) |

## Topics covered

- Critical section, race condition, and mutual exclusion, with the progress and bounded-waiting requirements
- Test-and-set and compare-and-swap
- Peterson’s solution for two processes
- Counting and binary semaphores, and the P and V operations
- Producer-consumer, readers-writers, and dining philosophers
- Monitors and condition variables
- Mutexes and spinlocks
- Atomic operations and a lock-free stack
- The four necessary conditions for deadlock
- Prevention, avoidance by the Banker’s algorithm, detection by a wait-for graph, and recovery
