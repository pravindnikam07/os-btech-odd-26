**Navigation:** [Unit index](00_Index.md) · Previous: — · [Next: 2.2 Threads and multithreading models](02_Threads_and_Multithreading.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 2 — Processes, Threads and CPU Scheduling

## Topic 2.1 — Processes, Process States, PCB, and Context Switching

### Learning Outcomes

- Distinguish a program from a process.
- Draw the five-state process diagram and explain each transition.
- List the fields stored in a process control block.
- Explain context switching and why it is pure overhead.
- Differentiate a context switch from a mode switch.

### Prerequisites

Kernel, user mode and kernel mode, and the idea that the operating system allocates the CPU (Unit 1).

### Introduction

A program stored on disk does nothing by itself. When the operating system loads it and starts executing it, that running instance is a process. The CPU can execute only one process on one core at a time, so the operating system keeps every process’s identity and saved state. That record is the process control block. Moving the CPU from one process to another is a context switch.

### Definition

**Definition:**  
A process is a program in execution. It includes the program code, the current activity (program counter and registers), a stack, a data section, and a heap.

A program is a passive file. A process is an active execution of that file. Two users can run the same editor program and produce two processes, each with its own program counter, stack, and data.

### Key Terminology

| Term | Meaning |
| --- | --- |
| Program | Passive executable file stored on disk |
| Process | Program in execution |
| Program counter | Address of the next instruction to execute |
| Stack | Memory for function calls, parameters, and local variables |
| Heap | Memory allocated at run time |
| Text section | The program’s instructions |
| PCB | Process control block; the kernel’s record of one process |
| Context | The CPU state that must be saved to resume a process |
| Context switch | Saving the current process and restoring another |
| Mode switch | Changing between user mode and kernel mode |

### Process in memory

```text
High address
+------------------+
|       Stack      |  grows downward
|         |        |
|         v        |
|                  |
|         ^        |
|         |        |
|       Heap       |  grows upward
+------------------+
|   Data section   |  global variables
+------------------+
|   Text section   |  instructions
+------------------+
Low address
```

### Process states

**Definition:**  
The process state is the current activity of a process as recorded by the operating system.

| State | Meaning |
| --- | --- |
| New | The process is being created |
| Ready | The process has everything it needs except the CPU |
| Running | Instructions are being executed on a CPU |
| Waiting | The process is waiting for an event, such as I/O completion or a child process |
| Terminated | The process has finished execution |

```text
                admitted
    +-------+ --------------> +-------+
    |  New  |                 | Ready | <---------+
    +-------+                 +-------+           |
                                 |  |             |
                    scheduler    |  |  interrupt  |
                    dispatch     |  |  or yield   |
                                 v  |             |
                              +---------+    +----------+
                              | Running |--->| Waiting  |
                              +---------+    +----------+
                                 |   I/O or event wait   ^
                                 |                       |
                                 |         I/O or event  |
                                 |         completion ---+
                                 | exit
                                 v
                            +------------+
                            | Terminated |
                            +------------+
```

**How a process moves:**

1. **New → Ready.** The operating system admits the process and places it in the ready queue.
2. **Ready → Running.** The scheduler dispatches the process. The dispatcher loads its context onto the CPU.
3. **Running → Waiting.** The process requests an event that has not happened yet, such as a disk read.
4. **Waiting → Ready.** The event occurs. The process can run again, but it must wait for the CPU.
5. **Running → Ready.** The timer expires, or a higher-priority process becomes ready, and the CPU is taken away. This transition exists on a preemptive system.
6. **Running → Terminated.** The process finishes or is aborted. The kernel releases its resources.

A waiting process does not return directly to running. It becomes ready first, then the scheduler may select it.

### Process control block

**Definition:**  
A process control block (PCB), also called a task control block, is the kernel data structure that stores all the information needed to manage one process.

| PCB field | What it stores |
| --- | --- |
| Process identifier | PID, parent PID |
| Process state | New, ready, running, waiting, or terminated |
| Program counter | Next instruction |
| CPU registers | General registers, stack pointer, condition codes |
| CPU-scheduling information | Priority, queue pointers, scheduling parameters |
| Memory-management information | Base and limit, or page-table pointer |
| Accounting information | CPU time used, time limits, account numbers |
| I/O status | Open files, allocated devices, pending I/O |

The PCB is created when the process is created and removed when the process is destroyed. The ready queue and the device queues hold pointers to PCBs, not copies of the whole program.

```text
Ready queue
+------+    +------+    +------+
| PCB  | -> | PCB  | -> | PCB  |
| P1   |    | P4   |    | P7   |
+------+    +------+    +------+

Disk queue                      Printer queue
+------+    +------+            +------+
| PCB  | -> | PCB  |            | PCB  |
| P2   |    | P5   |            | P3   |
+------+    +------+            +------+
```

### Context switch

**Definition:**  
A context switch is the action of saving the CPU state of the currently running process into its PCB and loading the saved state of another process.

**How it works:**

```text
Process P is running
        ↓
Interrupt, system call, or yield
        ↓
Kernel saves P: program counter, registers, stack pointer → PCB of P
        ↓
Kernel updates the state of P (Ready or Waiting)
        ↓
Scheduler selects process Q
        ↓
Kernel loads Q: registers, program counter, stack pointer, memory map ← PCB of Q
        ↓
CPU resumes at Q's program counter
```

1. The CPU leaves the running process because of a timer interrupt, a system call, an I/O interrupt, or an explicit yield.
2. The kernel saves the program counter and registers into the PCB of the outgoing process.
3. The outgoing process is marked ready or waiting.
4. The short-term scheduler selects the next process.
5. The kernel restores that process’s registers and switches the memory map if the address space changes.
6. Execution continues at the restored program counter.

**Important points:**

- During the switch the CPU does no useful work for any user process. Context-switch time is overhead.
- The time depends on the number of registers, the speed of memory, and whether the address space (and TLB) must change.
- Threads of the same process can switch faster than two processes, because they share the address space. That comparison is completed in Topic 2.2.
- A mode switch is not always a context switch. A system call switches from user mode to kernel mode and may return to the same process. A context switch changes which process is running.

### Worked example

**Problem:**  
Process P is running. A timer interrupt occurs. Process Q is ready. Describe the PCB updates.

**Step 1:**  
The hardware saves the return address and switches to kernel mode.

**Step 2:**  
The kernel copies P’s program counter and registers into P’s PCB and sets P’s state to Ready.

**Step 3:**  
The scheduler selects Q. Q’s state in its PCB is Ready.

**Step 4:**  
The kernel copies Q’s saved registers and program counter from Q’s PCB onto the CPU and sets Q’s state to Running.

**Step 5:**  
The kernel returns to user mode at Q’s restored program counter. P is in the ready queue.

**Final answer:**  
P moves from Running to Ready. Q moves from Ready to Running. The CPU state of each process is stored in its own PCB.

### Comparison

| Parameter | Program | Process |
| --- | --- | --- |
| Nature | Passive file | Active execution |
| Program counter | None | One (or one per thread) |
| Lifetime | Remains on disk | Exists from creation until termination |
| Resources | Occupies disk space | Holds CPU state, memory, and open files |
| How many from one file | One file | Many processes can run one program |

| Parameter | Mode switch | Context switch |
| --- | --- | --- |
| What changes | User mode or kernel mode | Which process is running |
| PCB of another process | Not required | Required |
| Typical cause | System call or interrupt that returns to the same process | Timer, preemption, or blocking, followed by dispatch of another process |
| Overhead | Smaller | Larger, because registers and often the memory map are switched |

### Advantages and limitations

| Advantages of the process model | Limitations |
| --- | --- |
| Each program runs with its own state and protection | Creating a process and switching context both cost time |
| The ready queue lets the CPU stay busy when one process waits | A PCB and an address space are heavier than a thread |
| The kernel can suspend and resume work exactly where it stopped | A faulty design of the PCB would lose the only copy of a process’s registers |

### Common mistakes

- Calling the ready state “the process is running in the background.” Ready means it is waiting only for the CPU.
- Drawing an arrow from Waiting directly to Running. The event makes the process Ready. The scheduler later makes it Running.
- Treating a system call as a context switch in every case. The same process often continues after the call.
- Listing only the PID inside the PCB. Registers, the program counter, memory information, open files, and scheduling data belong there as well.

### Important exam points

- Five states and the six transitions, including Running → Ready on a timer.
- PCB field table.
- Context-switch steps, and the statement that the switch itself is overhead.
- Program versus process, and mode switch versus context switch.

### University exam questions

#### 2-mark questions

1. Define a process.
2. Differentiate a program and a process.
3. Name the five process states.
4. What is a process control block?
5. Why is a context switch called overhead?

#### 4/5-mark questions

1. Draw the process state diagram and explain each transition.
2. Explain the contents of a PCB.
3. Explain context switching with a diagram.
4. Differentiate a mode switch and a context switch.

#### 8/10-mark questions

1. Explain process states, the PCB, and context switching. Show how the ready queue and a device queue use PCBs.
2. Describe what the kernel saves and restores when a timer interrupt replaces process P with process Q.

### Practice problems

#### Easy

1. In which state is a process that has been created but not yet admitted?
2. Which state is the process in while the CPU is executing its instructions?
3. Name two items stored in a PCB besides the PID.
4. A process issues a disk read. Which state does it enter?
5. After the disk interrupt, which state does that process enter?

#### Medium

1. Explain why a waiting process cannot move directly to Running.
2. A timer interrupt occurs and the scheduler selects the same process again. Was a full switch to a different process required? Explain.
3. List the steps of a context switch from P to Q.
4. Two students start the same compiler. How many programs and how many processes are involved?
5. Why do device queues store PCB pointers?

#### Hard

1. A process is Running, then Waiting, then Ready, then Running. State one event that can cause each arrow.
2. Compare the work done on a system call that returns to the same process with the work done when the call blocks and another process runs.
3. The machine has two cores. Can two processes be in the Running state at the same time? What does the PCB of each still need?
4. Why must the memory-management field of the PCB be updated as part of a process switch?
5. A designer stores CPU registers only in the process stack and deletes them from the PCB. What goes wrong when the kernel switches processes?

### MCQs

**Q1. A process is:**

A. A passive file stored on disk  
B. A program in execution  
C. Only the program counter  
D. The ready queue itself  

**Answer:** B  

**Explanation:** A program is passive. A process is that program while it is executing, together with its state and resources.

**Q2. A process that is waiting only for the CPU is in the:**

A. Ready state  
B. Waiting state  
C. New state  
D. Terminated state  

**Answer:** A  

**Explanation:** Ready means the process could run immediately if the scheduler selected it. Waiting means it is blocked for some other event.

**Q3. When an I/O operation completes, the blocked process moves to:**

A. Running, with no action by the scheduler  
B. Ready  
C. Terminated  
D. New  

**Answer:** B  

**Explanation:** Completion makes the process eligible for the CPU. The scheduler later moves it from Ready to Running.

**Q4. The PCB stores:**

A. Only the process name printed by the user  
B. The information the kernel needs to manage and resume the process  
C. The entire hard disk  
D. Only the source code of the program  

**Answer:** B  

**Explanation:** The PCB holds the state, program counter, registers, scheduling data, memory information, accounting, and open files.

**Q5. Context-switch time is overhead because:**

A. The CPU performs useful user work faster during the switch  
B. No user process makes progress while state is being saved and restored  
C. The PCB is stored on paper  
D. The process changes from Waiting directly to Running  

**Answer:** B  

**Explanation:** Saving and restoring registers is kernel work. The user programs do not advance during that interval.

**Q6. A mode switch differs from a context switch because a mode switch:**

A. Always replaces the running process  
B. Changes privilege and can return to the same process  
C. Deletes the PCB  
D. Moves a process from New to Terminated  

**Answer:** B  

**Explanation:** User mode and kernel mode can change without selecting a different process.

**Q7. The program counter in the PCB is used to:**

A. Count how many processes exist  
B. Resume the process at the correct instruction  
C. Store the file name  
D. Set the printer priority only  

**Answer:** B  

**Explanation:** The saved program counter is the address of the next instruction.

**Q8. On a preemptive uniprocessor, a timer interrupt typically causes:**

A. Running → Ready, if another process is scheduled  
B. Waiting → Running directly  
C. Terminated → New  
D. Ready → Waiting  

**Answer:** A  

**Explanation:** The running process loses the CPU and becomes ready. Another ready process may then be dispatched.

### Quick revision

#### Key definitions

- Process: a program in execution.
- PCB: the kernel record used to manage and resume one process.
- Context switch: save one process’s CPU state and restore another’s.

#### States

New → Ready → Running, with Waiting beside Running, and Terminated after exit. Running can return to Ready on preemption.

#### PCB fields to write in an answer

PID and state, program counter, registers, scheduling information, memory information, accounting, I/O status and open files.

#### Exam points

- Waiting returns to Ready, not directly to Running.
- Context switch is overhead.
- A system call is a mode switch; it becomes a context switch only if a different process is dispatched.
