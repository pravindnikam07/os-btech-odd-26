**Navigation:** [Unit index](00_Index.md) · [Previous: 2.1 Processes, PCB, and context switching](01_Processes_PCB_and_Context_Switch.md) · [Next: 2.3 Scheduling criteria](03_Scheduling_Criteria.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 2 — Processes, Threads and CPU Scheduling

## Topic 2.2 — Threads and Multithreading Models

### Learning Outcomes

- Define a thread and compare it with a process.
- Differentiate user-level threads and kernel-level threads.
- Explain the many-to-one, one-to-one, and many-to-many models.
- State what happens when one thread blocks in each model.

### Prerequisites

Process, address space, PCB, and context switch (Topic 2.1). System calls and kernel mode (Unit 1).

### Introduction

A process can contain more than one thread of execution. The threads share the process’s code, data, and open files, and each thread has its own program counter, registers, and stack. A word processor can format text in one thread, accept keystrokes in another, and save the file in a third, without creating three separate processes.

### Definition

**Definition:**  
A thread is the smallest unit of CPU execution inside a process. Threads of the same process share the address space and other process resources, and each thread has its own execution state.

A process is the unit of resource ownership. A thread is the unit of scheduling and execution inside that ownership.

### Key terminology

| Term | Meaning |
| --- | --- |
| Thread | One sequence of instructions inside a process |
| Multithreading | Supporting more than one thread in a process |
| User-level thread | A thread managed by a user-space library; the kernel does not see it as a separate schedulable entity |
| Kernel-level thread | A thread that the kernel creates and schedules |
| Thread control block | The saved program counter, registers, and stack pointer of one thread |
| Many-to-one | Many user threads mapped to one kernel thread |
| One-to-one | Each user thread mapped to one kernel thread |
| Many-to-many | Many user threads mapped to a smaller or equal number of kernel threads |

### What is shared and what is private

| Shared by threads of one process | Private to each thread |
| --- | --- |
| Code (text) | Program counter |
| Data section and heap | Registers |
| Open files and other OS resources | Stack |
| Process identity, such as the PID | Thread identifier and thread state |

```text
Process
+----------------------------------------------+
| Code, data, heap, open files                 |
|                                              |
|  Thread 1          Thread 2          Thread 3|
|  +----------+      +----------+      +------+|
|  | PC       |      | PC       |      | PC   ||
|  | registers|      | registers|      | regs ||
|  | stack    |      | stack    |      | stack||
|  +----------+      +----------+      +------+|
+----------------------------------------------+
```

Because the address space is shared, one thread can overwrite data that another thread is using. That problem is the subject of process synchronization in Unit 3. Creating a thread is still cheaper than creating a process, and switching between two threads of the same process does not require a change of address space.

### User-level and kernel-level threads

#### User-level threads

**Definition:**  
User-level threads are created and switched by a thread library in user space. The kernel schedules the process as a whole and has no separate PCB entry for each of these threads.

**How scheduling works:**

1. The library creates threads inside the process.
2. The library chooses which thread runs and switches the program counter, registers, and stack in user mode.
3. The kernel sees one schedulable entity.
4. If any thread makes a blocking system call, the kernel blocks the whole process. Every thread of that process stops.
5. On a multicore machine the process still occupies only one core, because the kernel has only one kernel thread to run.

The switch is fast because it does not enter the kernel. The blocking behaviour is the serious limitation.

#### Kernel-level threads

**Definition:**  
Kernel-level threads are known to the operating system. The kernel creates them, stores their registers, and schedules them.

**How scheduling works:**

1. A thread is created by a system call.
2. The kernel can place different threads of the same process on different cores.
3. If one thread blocks in a system call, the kernel can run another thread of the same process.
4. A switch between kernel threads enters the kernel, so it costs more than a pure user-level switch.
5. The number of threads is limited by kernel resources, not only by a user library.

Linux (through NPTL, the Native POSIX Thread Library) and modern Windows use kernel threads in a one-to-one mapping.

### Multithreading models

| Model | Mapping | Blocking of one thread | Multicore use | Example |
| --- | --- | --- | --- | --- |
| Many-to-one | Many user threads, one kernel thread | Blocks the whole process | One core for that process | Early thread libraries; Green threads style |
| One-to-one | One user thread, one kernel thread | Other threads of the process can run | Threads can run on several cores | Linux NPTL, Windows |
| Many-to-many | Many user threads onto many kernel threads, usually fewer kernel threads than user threads | A blocking thread blocks one kernel thread; other kernel threads can continue | Up to as many cores as kernel threads | Older Solaris and similar two-level designs |

#### Many-to-one

```text
User threads          Kernel
  T1 \
  T2  -->  one kernel thread  -->  one core
  T3 /
```

The library does the multiplexing. Thread creation is cheap. The process cannot make progress on another thread during a blocking call, and it cannot use more than one core.

#### One-to-one

```text
User thread T1  -->  kernel thread K1  -->  core
User thread T2  -->  kernel thread K2  -->  core
User thread T3  -->  kernel thread K3  -->  core
```

Concurrency is real: the kernel can run the threads at the same time on different cores, and a blocking call blocks only the calling thread. Each thread consumes kernel memory and scheduling time, so creating a very large number of threads has a cost.

#### Many-to-many

```text
User threads T1 T2 T3 T4 T5
        \  |  |  |  /
         kernel threads K1 K2 K3
              |    |    |
            cores that the kernel assigns
```

The programmer can create many user threads. The operating system allocates a smaller pool of kernel threads and multiplexes the user threads onto that pool. If one kernel thread blocks, another kernel thread can still run a ready user thread. The model is more flexible than many-to-one and can create fewer kernel objects than a strict one-to-one design. It is also more complicated to implement. Current general-purpose systems mostly use one-to-one.

A two-level variant of many-to-many also allows a particular user thread to be bound to one kernel thread, while the others remain multiplexed. The bound thread is used when an application needs a thread that the kernel will always see.

### Comparison of a process switch and a thread switch

| Parameter | Two processes | Two threads of one process |
| --- | --- | --- |
| Address space | Different; memory map changes | Same |
| Open files | Separate, unless inherited and then changed | Shared |
| What is saved | Full process context | Program counter, registers, and stack pointer |
| Protection between them | Separate address spaces | Shared data; the programmer must cooperate |
| Typical cost | Higher | Lower when the address space stays the same |

### Advantages and limitations

| Advantages of threads | Limitations |
| --- | --- |
| Faster creation than a new process | Shared data needs synchronization |
| Faster switch inside the same address space | One thread can corrupt another thread’s data |
| Overlap of computation and I/O inside one program | User-level threads block the whole process on a blocking call |
| Use of more than one core when the threads are kernel threads | One-to-one threads consume kernel resources |

### Applications

- A web browser fetches data, draws the page, and handles input in separate threads.
- A server creates a thread for each client request instead of a full process.
- A spreadsheet recalculates values while the user edits another cell.
- Word processing overlaps typing, spell-checking, and saving.

### Common mistakes

- Saying threads do not share memory. Threads of one process share code, data, and files. Each has a private stack and registers.
- Saying every thread switch is as expensive as a process switch. Sharing the address space removes the memory-map change.
- Claiming that a blocking call in a many-to-one library blocks only one thread. The kernel blocks the single kernel thread, so the whole process stops.
- Mixing up the models: one-to-one is not “one process, one thread.” It is one kernel thread for each user thread. The process may contain many such pairs.

### Important exam points

- Shared items versus private items.
- User-level thread: fast switch, whole process blocks, no parallel use of cores.
- Kernel-level thread: kernel schedules it, one thread can block alone, can run on another core.
- Three mapping diagrams: many-to-one, one-to-one, many-to-many.
- Linux NPTL is one-to-one.

### University exam questions

#### 2-mark questions

1. Define a thread.
2. Name two resources shared by threads of the same process.
3. What is a user-level thread?
4. What is the one-to-one multithreading model?
5. Why can a thread context switch be cheaper than a process context switch?

#### 4/5-mark questions

1. Differentiate a process and a thread.
2. Explain user-level and kernel-level threads.
3. Explain the many-to-one model with a diagram. What happens on a blocking system call?
4. Explain the one-to-one model and one system that uses it.

#### 8/10-mark questions

1. Explain user-level and kernel-level threads. Compare many-to-one, one-to-one, and many-to-many models with diagrams.
2. Why do servers use threads? Explain which model allows threads of one process to run on two cores, and why.

### Practice problems

#### Easy

1. Does each thread have its own stack?
2. Do threads of one process share the heap?
3. Which model maps each user thread to a kernel thread?
4. Name one resource that is private to a thread.
5. In which model does a blocking system call block every thread of the process?

#### Medium

1. Draw many-to-one and one-to-one mappings.
2. A process has four user-level threads on a many-to-one library and the machine has eight cores. How many cores can this process use at once? Why?
3. Explain why thread creation does not copy the entire address space.
4. Compare the effect of a blocking read in many-to-one and in one-to-one.
5. Why is synchronization required even though threads are “inside one program”?

#### Hard

1. An application wants thousands of concurrent tasks but also wants a blocking call to stop only the caller. Which model fits, and what cost remains?
2. Explain many-to-many in terms of both parallelism and blocking.
3. A context switch moves from thread T1 of process P to thread T2 of process Q. Which parts of the context must change, compared with a switch from T1 to another thread of P?
4. Why did operating systems move from many-to-one libraries toward one-to-one kernel threads on multicore machines?
5. Describe a two-level arrangement in which one user thread is bound to a kernel thread and the others are multiplexed.

### MCQs

**Q1. Threads of the same process share:**

A. Program counter and stack  
B. Code, data, and open files  
C. Nothing at all  
D. Only the CPU registers  

**Answer:** B  

**Explanation:** Code, data, heap, and open files are shared. The program counter, registers, and stack are private to each thread.

**Q2. A user-level thread is scheduled by:**

A. A user-space thread library  
B. The disk controller  
C. Only another computer  
D. The file name  

**Answer:** A  

**Explanation:** The kernel does not schedule each user-level thread. The library switches among them.

**Q3. In the many-to-one model, a blocking system call:**

A. Blocks only the calling user thread and lets the others run  
B. Blocks the single kernel thread, so the whole process waits  
C. Creates a new kernel for each thread  
D. Has no effect  

**Answer:** B  

**Explanation:** There is only one kernel thread. When it blocks, no user thread of that process can run.

**Q4. The one-to-one model allows:**

A. Only one thread in the entire computer  
B. Another thread of the process to run when one thread blocks, including on another core  
C. User threads with no kernel support and full parallelism  
D. Sharing of one stack by all threads  

**Answer:** B  

**Explanation:** Each user thread has a kernel thread, so the kernel can schedule them independently.

**Q5. Linux NPTL implements:**

A. Many-to-one  
B. One-to-one  
C. No threads  
D. One stack for every process in the system  

**Answer:** B  

**Explanation:** NPTL maps each POSIX thread to a kernel schedulable entity.

**Q6. Many-to-many means:**

A. Many user threads multiplexed onto many kernel threads  
B. Many CPUs inside one register  
C. Many processes forced to share one program counter  
D. One user thread and no kernel thread  

**Answer:** A  

**Explanation:** A pool of kernel threads carries a larger set of user threads.

**Q7. Which item is private to a thread?**

A. Heap of the process  
B. Stack  
C. Open file table of the process  
D. Code section  

**Answer:** B  

**Explanation:** Each thread needs its own stack for calls and local variables. The heap and code are shared.

**Q8. Thread creation is generally cheaper than process creation because:**

A. A new thread does not need a separate copy of the address space  
B. Threads cannot run instructions  
C. The kernel deletes the PCB  
D. Threads do not use a program counter  

**Answer:** A  

**Explanation:** The new thread shares the existing code, data, and files. It needs its own registers and stack.

### Quick revision

#### Definitions

- Thread: smallest unit of execution inside a process.
- User-level thread: managed by a library; kernel sees one entity.
- Kernel-level thread: scheduled by the kernel.

#### Models

- Many-to-one: fast user switch; blocking call stops the process; one core.
- One-to-one: kernel thread per user thread; true blocking isolation; Linux and Windows.
- Many-to-many: many user threads on a pool of kernel threads.

#### Shared versus private

Shared: code, data, heap, files. Private: PC, registers, stack.

#### Exam point

If the question asks which model runs threads of one process in parallel on several cores, the answer is one-to-one, or many-to-many up to the number of kernel threads. Many-to-one cannot.
