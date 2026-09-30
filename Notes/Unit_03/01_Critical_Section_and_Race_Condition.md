**Navigation:** [Unit index](00_Index.md) · Previous: — · [Next: 3.2 Test-and-set and compare-and-swap](02_Test_and_Set_and_Compare_and_Swap.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 3 — Process Synchronization and Deadlocks

## Topic 3.1 — Critical Section, Race Condition, and Mutual Exclusion

### Learning Outcomes

- Identify the critical section of a concurrent program.
- Show a race condition with a lost update.
- State mutual exclusion, progress, and bounded waiting.
- Explain why a lock is required when threads share data.

### Prerequisites

Processes, threads, and the fact that threads of one process share data and the heap (Unit 2). Interrupts and preemption.

### Introduction

Two threads that only read their own local variables do not interfere. The difficulty starts when they share a variable, a buffer, or a file. The operating system may switch from one thread to the other in the middle of a read-modify-write. The result then depends on the order of those steps. That situation is a race condition, and the shared steps form a critical section. The rest of this unit is a sequence of ways to make that section safe.

### Definition

**Definition:**  
A critical section is a part of a program that accesses shared data and must not be executed by more than one process or thread at the same time.

**Definition:**  
A race condition is a situation in which the result depends on the unpredictable order in which processes read and write shared data.

**Definition:**  
Mutual exclusion means that if one process is inside its critical section, no other process is inside a critical section for the same shared data.

### Key terminology

| Term | Meaning |
| --- | --- |
| Shared variable | Data that more than one process or thread can access |
| Atomic operation | An operation that completes as a whole; nothing else observes it half-done |
| Remainder section | The part of the program that does not touch the shared data |
| Entry section | The code that asks permission to enter the critical section |
| Exit section | The code that announces that the critical section is finished |
| Lock | A variable whose value records whether the critical section is occupied |

### Structure of a concurrent process

```text
repeat
    entry section
    critical section          // use the shared data
    exit section
    remainder section         // private work
until false
```

The entry and exit sections are the solution. The critical section itself is the application’s use of the shared data. A correct solution does not depend on the relative speeds of the processes, and it does not assume that one process is faster than another.

### A race on a shared counter

`count` is 5. Two processes each add 1. The intended result is 7.

Each addition is three machine steps:

```text
load  count into a register
add   1
store the register into count
```

| Time | Process A | Process B | count in memory |
| --- | --- | --- | --- |
| 1 | load 5 | | 5 |
| 2 | | load 5 | 5 |
| 3 | add → 6 | | 5 |
| 4 | | add → 6 | 5 |
| 5 | store 6 | | 6 |
| 6 | | store 6 | 6 |

Both processes read 5. Both write 6. One update is lost. If B runs only after A has stored, the result is 7. The same source code produces two answers. That is the race.

The critical section is the load-add-store of `count`, not the whole program. Mutual exclusion forces one process to finish those three steps before the other begins them.

### The three requirements

A solution to the critical-section problem must satisfy all three.

| Requirement | Meaning |
| --- | --- |
| Mutual exclusion | At most one process is in the critical section at a time |
| Progress | If the critical section is empty and some processes want to enter, the choice of who enters is made only among those waiting processes, and the choice is not postponed forever |
| Bounded waiting | After a process has asked to enter, there is a limit on how many times other processes may enter before that process is allowed in |

Progress is about the system not freezing when nobody is inside. Bounded waiting is about one process not being overtaken forever. A solution can have mutual exclusion and still fail bounded waiting: a late process might never be selected while others keep winning the race for the lock.

Two further expectations are used when judging a solution:

- No process should be able to stay in the critical section forever. Other processes must get a turn.
- The solution must work even if processes run at different speeds. It must not assume “A is always faster than B.”

### Where the problem appears

- Two threads increment a shared counter.
- A producer writes a buffer slot while a consumer reads the same slot.
- Two processes update the same account balance.
- The kernel updates a ready queue from an interrupt handler and from a system call at the same time.

Disabling interrupts on a uniprocessor can protect a short kernel section, because the current code cannot be switched out. It fails on a multicore machine: another core continues to run. It is also too strong to give to a user process, which could disable interrupts and never turn them back on. The later topics therefore use hardware instructions, Peterson’s variables, semaphores, and locks.

### Common mistakes

- Calling every shared read a critical section. A race needs a use that can interleave, typically a read-modify-write or a pair of related updates.
- Treating “the program gave 6 once” as proof that the code is safe. A race can be invisible until the unlucky interleaving occurs.
- Confusing mutual exclusion with bounded waiting. A lock that one fast process can reacquire forever may still keep others out of the section.
- Disabling interrupts and calling that a general user-level solution. It is a kernel technique for one CPU, and it does not stop another core.

### Important exam points

- Definitions of critical section, race condition, and mutual exclusion.
- The lost-update trace on a counter, ending at 6 instead of 7.
- The three requirements, especially the difference between progress and bounded waiting.
- Entry section, critical section, exit section, remainder section.
- Why interrupt disabling is not enough on a multicore processor.

### University exam questions

#### 2-mark questions

1. Define a critical section.
2. Define a race condition.
3. Define mutual exclusion.
4. What is bounded waiting?

#### 4/5-mark questions

1. Explain the structure of a process that has a critical section.
2. Show a race condition on a shared counter with an interleaving table.
3. State and explain the three requirements of a critical-section solution.

#### 8/10-mark questions

1. Explain race conditions and mutual exclusion. Illustrate a lost update, and state why disabling interrupts does not solve the problem on a multicore machine.
2. Explain progress and bounded waiting. Give the meaning of each and show how they differ.

### Practice problems

#### Easy

1. Name the four parts of the critical-section loop.
2. Two processes add 1 to a counter that starts at 10, and the updates are lost. What wrong final value is possible?
3. Does mutual exclusion allow two processes in the same critical section?
4. Which requirement limits how long a process may be overtaken?
5. Is the remainder section shared-data code?

#### Medium

1. Write the six-step interleaving that turns two increments of 5 into a final value of 6.
2. Explain progress without using the words “bounded waiting.”
3. A lock is free and three processes are waiting. Which requirement says the decision must not be delayed forever?
4. Why can a user program not be allowed to disable interrupts for the whole critical section?
5. Two processes only read a shared table and never write it. Is there a race on that table?

#### Hard

1. Process A adds 1 and process B subtracts 1. The counter starts at 5. Show an interleaving whose result is 6, and one whose result is 4.
2. A solution lets process A enter whenever it asks, and B enters only when A is in its remainder section. A stays in a tight loop of critical section and a very short remainder. Which requirement fails?
3. Explain why “it worked in the laboratory ten times” is not a proof of mutual exclusion.
4. The kernel increments a queue length from a system call and from an interrupt on the same CPU. Why does a brief interrupt disable work there, and why does the same idea fail if the interrupt is handled on another core?
5. Distinguish a race condition from deadlock in one comparison: what is each process doing, and is forward progress of the others possible?

### MCQs

**Q1. A critical section is code that:**

A. Accesses shared data and must not be concurrent with another such access  
B. Contains only local variables  
C. Always runs in the remainder of the program  
D. Disables the disk  

**Answer:** A  

**Explanation:** The critical section is the use of shared data that has to be kept exclusive.

**Q2. A race condition means the result:**

A. Depends on the order of access to shared data  
B. Is independent of scheduling  
C. Is always the sum of the inputs  
D. Requires a deadlock  

**Answer:** A  

**Explanation:** Different interleavings of the same code can produce different results.

**Q3. Two increments of a counter that start at 5 can finish at 6 when:**

A. Both processes load 5 before either stores  
B. Each store completes before the other process loads  
C. The processes do not share the counter  
D. The counter is updated by one atomic instruction  

**Answer:** A  

**Explanation:** Both compute 6 from the old value, and the second store overwrites the first.

**Q4. Mutual exclusion requires that:**

A. At most one process is in the critical section  
B. Every process enters at the same time  
C. The remainder section is locked  
D. Interrupts stay disabled for the life of the process  

**Answer:** A  

**Explanation:** Exclusion is the “at most one” rule for the shared section.

**Q5. Bounded waiting limits:**

A. How many times others may enter after this process has requested entry  
B. The size of the process control block  
C. The number of CPUs in the computer  
D. The length of the remainder section only  

**Answer:** A  

**Explanation:** There must be a finite bound on being overtaken.

**Q6. Progress says that if nobody is in the critical section and somebody wants to enter, then:**

A. The choice is made among the processes that want to enter, and it is not postponed indefinitely  
B. The remainder section runs under a lock  
C. The slowest process must enter first, with no other rule allowed  
D. Interrupts are disabled on every core forever  

**Answer:** A  

**Explanation:** The system must be able to choose a waiting process and must not delay that choice forever.

**Q7. Disabling interrupts is an inadequate general solution because:**

A. Another core can still execute the critical section  
B. It speeds up every user program  
C. It is the same mechanism as a semaphore signal  
D. It implements bounded waiting on all machines by itself  

**Answer:** A  

**Explanation:** Interrupt disabling affects the core that executes the instruction. Other cores continue.

**Q8. The exit section is responsible for:**

A. Releasing the critical section so another process may enter  
B. Reading the shared data  
C. Creating a new process  
D. Increasing the time quantum  

**Answer:** A  

**Explanation:** Entry asks for permission. Exit gives the section up. The shared work sits between them.

### Quick revision

#### Definitions

- Critical section: exclusive use of shared data.
- Race condition: the result depends on interleaving.
- Mutual exclusion: at most one process inside.

#### Lost update

Counter starts at 5. Both load 5. Both store 6. One increment disappears.

#### Requirements

Mutual exclusion, progress, and bounded waiting. All three are required. Progress is not the same statement as bounded waiting.
