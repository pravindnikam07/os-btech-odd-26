**Navigation:** [Unit index](00_Index.md) · [Previous: 3.2 Test-and-set and compare-and-swap](02_Test_and_Set_and_Compare_and_Swap.md) · [Next: 3.4 Semaphores](04_Semaphores.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 3 — Process Synchronization and Deadlocks

## Topic 3.3 — Peterson’s Solution

### Learning Outcomes

- Write Peterson’s entry and exit code for two processes.
- Show that the solution provides mutual exclusion, progress, and bounded waiting.
- State why the algorithm is a software solution and where it is fragile on modern hardware.

### Prerequisites

Critical-section requirements (Topic 3.1). The idea of a shared variable.

### Introduction

Peterson’s solution is a software solution for two processes. It uses two shared arrays of ordinary variables, with no special lock instruction. One array records who wants to enter. One variable records whose turn it is to wait. The combination stops both processes from being inside together, and it stops either process from being overtaken forever.

The algorithm is the standard classroom proof of the three requirements. Production locks use test-and-set or compare-and-swap, because modern processors and compilers can reorder ordinary memory operations unless the variables are declared to behave atomically.

### Definition

**Definition:**  
Peterson’s solution is a two-process software algorithm for the critical-section problem. Each process sets its own flag, gives the turn to the other process, and waits until the other process does not want to enter or the turn returns.

### The algorithm

Shared data, initially `flag[0] = flag[1] = false` and `turn` is either 0 or 1:

```text
repeat
    flag[i] = true;                 // process i wants to enter
    turn = j;                       // but give the other process the courtesy
    while (flag[j] and turn == j)
        ;                           // wait while j wants the section and it is j's turn

    critical section

    flag[i] = false;                // i no longer wants the section
    remainder section
until false
```

Process 0 uses i = 0 and j = 1. Process 1 uses i = 1 and j = 0. Each process writes only its own flag. Both may write `turn`.

```text
Process 0                         Process 1

flag[0] = true                    flag[1] = true
turn = 1                          turn = 0
while (flag[1] and turn == 1)     while (flag[0] and turn == 0)
critical section                  critical section
flag[0] = false                   flag[1] = false
```

### Why mutual exclusion holds

Suppose, for a contradiction, that both processes are in the critical section. Then both passed their while loops, so for each process the while condition was false.

Process 0 passed, so `flag[1]` was false or `turn` was 0.  
Process 1 passed, so `flag[0]` was false or `turn` was 1.

Both flags are true while a process is between setting its flag and clearing it, and both are inside, so both flags are true. The only remaining possibility is `turn == 0` and `turn == 1` at the times they passed. `turn` cannot be both values. The process that wrote `turn` last made the condition true for itself and false for the other. The last writer waits. The earlier writer is the one that may be inside. They are not inside together.

### Why progress holds

If only process i wants to enter, `flag[j]` is false, so i does not wait.

If both want to enter, `turn` is either 0 or 1. The while condition is true for only one of them. That process waits and the other enters. The choice is between the two processes that are trying to enter, and `turn` has already been set, so the choice is not put off.

### Why waiting is bounded

Suppose process i has set `flag[i]` and is waiting, and process j is inside or about to enter. Process j eventually leaves and sets `flag[j] = false`. Process i can then enter. Before process i enters, process j can get in at most once more: if j tries again it sets `turn = i`, so j’s own while loop waits while i still wants the section. After one such entry by j, i is the process that is allowed through. The bound is one turn of the other process.

| Requirement | How Peterson’s solution meets it |
| --- | --- |
| Mutual exclusion | The last process to assign `turn` waits; both flags cannot allow both loops to exit |
| Progress | A process whose partner does not want to enter does not wait; if both want to enter, `turn` selects one immediately |
| Bounded waiting | After i requests entry, j enters at most once before i enters |

### A short trace

Both flags start false. `turn` is 0. Process 0 and process 1 both become interested.

| Step | Action | flag[0] | flag[1] | turn | Who can enter |
| --- | --- | --- | --- | --- | --- |
| 1 | P0 sets flag[0] | true | false | 0 | |
| 2 | P1 sets flag[1] | true | true | 0 | |
| 3 | P0 sets turn = 1 | true | true | 1 | |
| 4 | P1 sets turn = 0 | true | true | 0 | P1 waits, because flag[0] is true and turn is 0 |
| 5 | P0 tests: flag[1] is true but turn is 0, not 1 | true | true | 0 | P0 enters |
| 6 | P0 sets flag[0] = false | false | true | 0 | P1’s test now fails; P1 enters |

P1 wrote `turn` last, so P1 is the one that waits. The trace can be reversed if P0 writes `turn` last. In every interleaving of the two entry sections, one process waits.

### Software solution and its limit

Peterson’s solution needs no test-and-set instruction. That is why it is called a software solution. It does need the shared writes to become visible in an order the proof assumes. A compiler may keep `flag[i]` in a register. A processor may reorder a write to `flag[i]` with a write to `turn`. Either change breaks the proof. On a real machine the variables have to be accessed with atomic operations or with an equivalent memory order. The algorithm remains the right way to see the three requirements. It is not, by itself, the lock used inside Linux.

It is also only a two-process algorithm. Test-and-set and compare-and-swap, and the semaphores built on them, are the usual tools once more than two processes share the data.

### Advantages and limitations

| Advantages | Limitations |
| --- | --- |
| Mutual exclusion, progress, and bounded waiting for two processes | Only two processes |
| No special hardware instruction in the textbook algorithm | Ordinary variables can be reordered by a compiler or a processor |
| The proof is short enough to write in an examination | The waiting loop is a busy wait |
| Shows the role of a turn variable separately from a flag | It is not the implementation used for general locks |

### Common mistakes

- Setting `turn = i` instead of `turn = j`. The process must give the turn away. If it keeps the turn, the mutual-exclusion argument changes and the published algorithm is no longer the one being used.
- Clearing the other process’s flag. Process i clears only `flag[i]`.
- Saying Peterson’s solution works for any number of processes. This algorithm is for two.
- Forgetting the while condition has two parts joined by and. A process waits only when the other wants to enter and it is the other’s turn.

### Important exam points

- The four shared names: `flag[0]`, `flag[1]`, `turn`, and the two process identities.
- Entry: `flag[i] = true; turn = j; while (flag[j] && turn == j);`
- Exit: `flag[i] = false;`
- All three requirements, with the one-sentence reason for each.
- Two processes only, and busy waiting.
- The last writer of `turn` is the process that waits.

### University exam questions

#### 2-mark questions

1. For how many processes is Peterson’s solution written?
2. What do `flag[i]` and `turn` mean?
3. Write the exit line of Peterson’s solution.
4. Name the three requirements it satisfies.

#### 4/5-mark questions

1. Write Peterson’s algorithm for process i.
2. Explain how mutual exclusion is achieved.
3. Explain bounded waiting in Peterson’s solution.

#### 8/10-mark questions

1. Explain Peterson’s solution. Prove mutual exclusion, progress, and bounded waiting.
2. Write the algorithm and a trace in which both processes try to enter and the last writer of `turn` waits.

### Practice problems

#### Easy

1. Which process’s flag does process i set to false on exit?
2. Does process i set `turn` to i or to j?
3. What is the initial value of each flag?
4. Is the wait a busy wait?
5. Can process 2 be added by copying the same two-line entry without a new algorithm?

#### Medium

1. Both processes set their flags, and process 0 writes `turn` last. Which process enters first?
2. Process 1 is in its remainder section and `flag[1]` is false. Does process 0 wait?
3. Explain progress for the case where only one process wants to enter.
4. After process 0 leaves, how many times may process 1 enter before process 0, which is already waiting, enters?
5. Why is a write kept only in a register fatal to the algorithm?

#### Hard

1. Write the mutual-exclusion argument without looking it up: assume both are inside and derive a contradiction from `turn`.
2. Interleave the entry code so that process 1 sets `turn` first and process 0 sets it second. State the final value of `turn` and who waits.
3. Explain why `while (flag[j])` without the turn test can block progress.
4. Explain why `while (turn == j)` without the flag test can block progress when the other process is in its remainder section.
5. A compiler moves `flag[i] = true` to after the assignment `turn = j`. Which part of the proof becomes unreliable?

### MCQs

**Q1. Peterson’s solution is designed for:**

A. Two processes  
B. Any number of processes without a change  
C. Only kernel threads on eight cores, with no shared variables  
D. Deadlock detection  

**Answer:** A  

**Explanation:** There are two flags and one turn variable. The algorithm as written is the two-process solution.

**Q2. The entry code of process i is:**

A. `flag[i] = true; turn = j; while (flag[j] && turn == j);`  
B. `flag[j] = true; turn = i; while (flag[i]);`  
C. `flag[i] = false; turn = i;`  
D. `while (test_and_set(&lock));`  

**Answer:** A  

**Explanation:** The process declares interest, gives the turn to the other process, and waits only if that other process is interested and still holds the turn.

**Q3. On leaving the critical section, process i executes:**

A. `flag[i] = false;`  
B. `flag[j] = false;`  
C. `turn = i;` only, leaving the flag true  
D. `flag[i] = true;`  

**Answer:** A  

**Explanation:** Clearing its own flag tells the other process that i is no longer a competitor.

**Q4. If both processes try to enter, the process that waits is:**

A. The one that wrote `turn` last  
B. Always process 0  
C. The one that wrote `turn` first  
D. Neither, because both enter  

**Answer:** A  

**Explanation:** The last assignment makes `turn` equal the other process’s identity, so the last writer’s while condition can remain true.

**Q5. Bounded waiting holds because after i has requested entry, j can enter at most:**

A. Once before i enters  
B. As many times as j wishes, with no limit  
C. Never, even if i is not interested  
D. n times for an arbitrary n added at runtime  

**Answer:** A  

**Explanation:** When j tries again it sets the turn to i, so j waits while i’s flag remains true.

**Q6. If process j is in its remainder section, process i:**

A. Does not wait, because `flag[j]` is false  
B. Waits until j finishes the remainder  
C. Clears `flag[j]`  
D. Sets `turn` to i and spins forever  

**Answer:** A  

**Explanation:** The while condition needs `flag[j]` to be true. A process in the remainder has cleared its flag.

**Q7. Peterson’s solution is called a software solution because:**

A. It uses shared variables rather than a special hardware lock instruction  
B. It cannot be written down  
C. It requires compare-and-swap in the entry line  
D. It schedules the CPU  

**Answer:** A  

**Explanation:** Flags and `turn` are ordinary shared variables in the textbook presentation.

**Q8. On a modern processor the textbook variables are not enough unless:**

A. The writes become visible in the order the proof assumes  
B. Interrupts are disabled for the whole program  
C. The machine has only one register  
D. `turn` is stored on disk  

**Answer:** A  

**Explanation:** Reordering or a stale register copy can break the entry order. Atomic access or an equivalent memory order is required in a real implementation.

### Quick revision

#### Code

```text
flag[i] = true
turn = j
while (flag[j] and turn == j)
    wait
critical section
flag[i] = false
```

#### Requirements

- Mutual exclusion: the last writer of `turn` waits.
- Progress: one of the interested processes enters without an indefinite delay.
- Bounded waiting: the other process enters at most once.

#### Limits

Two processes. Busy waiting. Real machines must not reorder the shared writes.
