**Navigation:** [Unit index](00_Index.md) · [Previous: 1.1 OS Concepts and Generations](01_OS_Concepts_and_Generations.md) · [Next: 1.3 OS Services and System Calls](03_OS_Services_and_System_Calls.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 1 — OS Fundamentals and Architecture

## Topic 1.2 — Types of Operating Systems

### Learning Outcomes

- Explain batch, time-sharing, distributed, and real-time operating systems.
- Differentiate these types using response behavior, number of users, and design goal.
- Identify multiprogramming as the technique that makes efficient batch and time-sharing systems possible.
- Classify a given system description into the correct OS type.

### Prerequisites

Topic 1.1 (what an OS is, batch generation, and the idea of sharing the CPU).

### Introduction

Operating systems are built for different environments. A payroll center that runs jobs overnight does not have the same needs as an airline reservation terminal, a sensor that must react before a deadline, or a service spread across many computers. The four types are batch, time-sharing, distributed, and real-time.

### Definition

**Definition:**  
The type of an operating system is determined by the way it accepts work, shares the processor, and responds to users or to external events.

### Key Terminology

| Term | Meaning |
| --- | --- |
| Job | A program submitted for execution, together with its data and control information |
| Multiprogramming | Keeping more than one program in memory so the CPU can switch when a program waits |
| Time slice (quantum) | A short interval of CPU time given to an interactive process |
| Turnaround time | Time from job submission to completion |
| Response time | Time from a user request to the first response |
| Throughput | Number of jobs completed per unit time |
| Node | One computer in a distributed system |
| Deadline | The latest time at which a real-time result is still useful |

### Basic Concepts

#### Multiprogramming

**Definition:**  
Multiprogramming is a technique in which several programs reside in memory at the same time. The operating system assigns the CPU to another ready program whenever the current program waits for an event such as I/O.

A single cashier who must wait for each customer to search for cash wastes time. If several customers are prepared, the cashier can serve another customer during a wait. Multiprogramming does the same with the CPU.

- It increases CPU utilization.
- It requires memory protection so that one program cannot overwrite another.
- It is a technique, not a separate type of operating system. Batch systems and time-sharing systems both use it.

**Example:**  
Job A starts a disk read. While the disk works, the OS runs Job B. When the disk finishes, Job A can become ready again.

```text
Memory
+------------------+
| Operating system |
+------------------+
| Job 1            |
+------------------+
| Job 2            |
+------------------+
| Job 3            |
+------------------+

CPU: run Job 1 → Job 1 waits for I/O → run Job 2 → ...
```

### Types

| Type | Definition | Characteristics | Example |
| --- | --- | --- | --- |
| Batch | Collects jobs and executes them one after another without user interaction during the run | High throughput, long turnaround, no interactive response | Early payroll and census processing; second-generation monitors |
| Time-sharing | Many interactive users share the CPU through short time slices | Good response time, multiprogramming, protection between users | UNIX time-sharing, a shared Linux lab server |
| Distributed | Independent computers cooperate over a network and present coordinated services | Concurrency, no single shared clock, fault independence of nodes | A cluster of web servers, distributed file service |
| Real-time | Correctness depends on meeting timing deadlines, not only on logical results | Predictable latency, priority of time-critical tasks | Engine control, pacemaker monitor, industrial robot controller |

#### Batch operating system

**Explanation:**  
Jobs are grouped and processed without the user sitting at the console. The design goal is throughput and efficient use of an expensive machine.

**How it works:**

1. Users submit jobs offline.
2. The operator or a spooler forms a batch.
3. The monitor starts the next job when the current job finishes.
4. Output is returned after the batch, or after that job, completes.
5. The user cannot supply input in the middle of execution.

**Important points:**

- CPU utilization is better than pure manual operation.
- Debugging is difficult because the user sees errors only after the run.
- Turnaround time is the usual performance measure, not interactive response time.

#### Time-sharing operating system

**Explanation:**  
Many users are connected through terminals or remote sessions. Each user believes the computer is responding personally. The OS switches rapidly among ready processes.

**How it works:**

1. Several users start interactive programs.
2. Programs are multiprogrammed in memory.
3. The scheduler gives each ready process a short time slice.
4. When the slice ends, or the process waits for input, the CPU moves to another process.
5. A keystroke or command receives a quick response.

**Important points:**

- Response time is the main user-visible measure.
- Time-sharing needs a timer interrupt so one process cannot hold the CPU forever.
- Protection and fair sharing are mandatory.

**Example:**  
Twenty students are logged in to one department server. Each editor session runs for a few tens of milliseconds at a time. Switching is fast enough that each student sees a responsive editor.

#### Distributed operating system

**Explanation:**  
The work is spread across multiple computers. The machines communicate by messages. A distributed OS (or a distributed operating environment) coordinates these machines so that users can use remote resources.

**How it works:**

1. Each node has its own local processor and memory.
2. Nodes are connected by a network.
3. A request that cannot be satisfied locally is sent to another node.
4. Results and data are returned by message passing.
5. Failure of one node should not, by itself, stop every other node.

**Important points:**

- There is no single shared physical memory and no single global clock.
- The system can grow by adding nodes.
- Network delay and partial failure are normal design problems.
- A network of independent PCs, each running a separate OS with no coordination, is a computer network. It becomes a distributed system when the machines cooperate to provide a joint service.

**Example:**  
A file stored on one server is opened by a user on another machine. The distributed file service locates the file, checks permission, and transfers the data.

#### Real-time operating system

**Definition:**  
A real-time operating system (RTOS) schedules work so that critical tasks complete within defined time limits.

**Explanation:**  
In these systems, a late answer can be useless or dangerous. There are two categories:

| Category | Timing rule | Consequence of a miss | Example |
| --- | --- | --- | --- |
| Hard real-time | Deadline must be guaranteed | System failure, possible physical damage | Airbag controller, aircraft flight control |
| Soft real-time | Deadline is important but an occasional miss is tolerable | Quality falls, system continues | Video streaming, online audio |

**How it works:**

1. Sensors or events raise requests.
2. The OS assigns priorities according to timing needs.
3. A higher-priority real-time task preempts less urgent work.
4. The design bounds interrupt latency and scheduling delay.
5. The task produces its result before the deadline.

**Important points:**

- Predictability matters more than maximum average throughput.
- Hard real-time systems often use simple, analyzable schedulers.
- General-purpose desktop systems are not hard real-time systems.

### Comparison

| Parameter | Batch | Time-sharing | Distributed | Real-time |
| --- | --- | --- | --- | --- |
| Main goal | Throughput | Interactive response | Resource sharing across machines | Meet deadlines |
| User interaction during execution | None | Continuous | Depends on the service | Usually event-driven |
| Number of computers | Typically one | Typically one | Many | One or more embedded controllers |
| CPU sharing | Job after job; often multiprogrammed | Time slices among users | Local scheduler on each node | Priority by deadline or criticality |
| Important measure | Turnaround time, throughput | Response time | Scalability, availability | Latency, deadline miss |
| Failure effect | Job fails or batch stops | User session affected | Often limited to one node or service | May be safety-critical |

### Advantages and Limitations

| Type | Advantages | Limitations |
| --- | --- | --- |
| Batch | Simple operation, good for repetitive jobs, high throughput | No interaction, long turnaround |
| Time-sharing | Interactive use, many users, good utilization | Overhead of switching, needs protection and a timer |
| Distributed | Sharing, growth by adding machines, fault isolation | Network delay, complex consistency and partial failure |
| Real-time | Predictable response to events | Strict design limits, hard real-time rejects unbounded delays |

### Applications

- Batch: monthly payroll, examination result processing submitted as jobs.
- Time-sharing: Linux servers used by many students, traditional UNIX terminals.
- Distributed: worldwide web services running on many coordinated servers.
- Real-time: anti-lock braking, industrial process control, medical monitoring.

### Common Mistakes

- Defining time-sharing as “only one user.” Time-sharing is specifically multi-user and interactive. A single user running several programs is multitasking, which may exist without multiple users.
- Calling every networked application a distributed operating system. Distribution requires cooperating nodes and a coordinated service.
- Treating soft real-time and hard real-time as the same. Only hard real-time guarantees the deadline.
- Forgetting that multiprogramming is the supporting technique for efficient sharing, while batch and time-sharing are system types.

### Important Exam Points

- Four types with definition, goal, and one example each.
- Hard versus soft real-time.
- Response time versus turnaround time.
- Distributed system: separate memory, message passing, cooperation.
- Multiprogramming diagram: several jobs in memory, CPU switches on a wait.

### University Exam Questions

#### 2-Mark Questions

1. Define a time-sharing operating system.
2. Define a real-time operating system.
3. What is multiprogramming?
4. Differentiate hard real-time and soft real-time systems.
5. State one characteristic of a distributed system.

#### 4/5-Mark Questions

1. Explain batch processing with its advantages and limitations.
2. Differentiate batch and time-sharing operating systems.
3. Explain distributed operating systems with a diagram of cooperating nodes.
4. Explain real-time operating systems with examples of hard and soft real-time applications.

#### 8/10-Mark Questions

1. Explain batch, time-sharing, distributed, and real-time operating systems. Compare them on goal, interaction, and performance measure.
2. What is multiprogramming? Show how it is used in time-sharing systems. Why is a timer interrupt required?

### Practice Problems

#### Easy

1. Give one example of a batch application.
2. Which OS type is judged by deadline behavior?
3. Name the time interval assigned to an interactive process.
4. Does a distributed system have one shared physical memory for all nodes?
5. State the main goal of a time-sharing system.

#### Medium

1. A railway reservation counter must answer ticket clerks in about a second, for many clerks. Which OS type fits, and why?
2. A chemical plant valve must close within 20 ms or the plant is unsafe. Classify the system.
3. Why can CPU utilization be low in a single-programming system even when jobs are waiting?
4. Two office PCs exchange email through independent mail software. Explain whether this fact alone makes them one distributed OS.
5. Compare turnaround time and response time with one example each.

#### Hard

1. Design a short justification for using multiprogramming inside a batch installation that has several tape and disk jobs.
2. A video call may drop a frame but must not stop. An airbag controller must fire on time. Classify both and compare the scheduling goal.
3. Explain how failure of one node is handled differently in a distributed system than failure of the only CPU in a batch mainframe.
4. A time-sharing system has no timer interrupt. Describe the unfair situation that can occur.
5. A hospital system stores records on several servers and also runs heart-monitor alarms. Identify which requirements are distributed and which are real-time.

### MCQs

**Q1. In a batch system, the user:**

A. Interacts with the program at every instruction  
B. Submits the job and receives output later  
C. Must be the only person ever to use that model of computer  
D. Edits the kernel during the run  

**Answer:** B  

**Explanation:** Batch jobs run without interactive user input during execution.

**Q2. The main user-visible requirement of a time-sharing system is:**

A. Maximum overnight turnaround only  
B. Short response time for interactive requests  
C. Absence of a CPU  
D. Execution of only one program per day  

**Answer:** B  

**Explanation:** Time-sharing exists so many interactive users receive quick responses.

**Q3. Multiprogramming improves CPU utilization because:**

A. The CPU executes several instructions in one clock cycle by definition of multiprogramming  
B. When one job waits for I/O, another ready job can run  
C. Disks are removed from the system  
D. Jobs are never stored in memory  

**Answer:** B  

**Explanation:** The CPU switches to another memory-resident job instead of idling during I/O.

**Q4. A distributed system is characterized by:**

A. One shared physical memory for all processors and one global clock  
B. Autonomous nodes that cooperate by communication  
C. A ban on networks  
D. Only vacuum-tube hardware  

**Answer:** B  

**Explanation:** Nodes have local resources and coordinate through messages.

**Q5. A hard real-time system:**

A. Tries to meet deadlines but may miss them without failure  
B. Must meet deadlines; missing one is a failure  
C. Is defined only by a colorful screen  
D. Cannot use priorities  

**Answer:** B  

**Explanation:** In hard real-time systems, deadline guarantees are part of correctness.

**Q6. A timer interrupt is required in time-sharing mainly to:**

A. Prevent one process from occupying the CPU indefinitely  
B. Replace the file system  
C. Turn off multiprogramming  
D. Store user passwords  

**Answer:** A  

**Explanation:** The timer returns control to the OS at the end of a time slice.

**Q7. Soft real-time differs from hard real-time because:**

A. Soft real-time has no notion of time  
B. An occasional missed deadline degrades service but is tolerated  
C. Soft real-time forbids priorities  
D. Soft real-time uses only batch jobs  

**Answer:** B  

**Explanation:** Multimedia is the standard soft real-time example. A missed frame is undesirable but not a hard failure.

**Q8. Which pair is correctly matched?**

A. Batch — deadline guarantee for airbags  
B. Time-sharing — interactive multi-user response  
C. Distributed — single computer with no network  
D. Real-time — jobs submitted only on punched cards with no timing need  

**Answer:** B  

**Explanation:** Time-sharing is the interactive multi-user type. The other pairs mix definitions.

### Quick Revision

#### Key Definitions

- Batch: non-interactive job processing.
- Time-sharing: interactive multi-user system using time slices.
- Distributed: cooperating autonomous computers.
- Real-time: timing deadlines are part of correctness.
- Multiprogramming: several jobs in memory; CPU switches on a wait.

#### Important Concepts

- Hard real-time guarantees deadlines. Soft real-time tolerates occasional misses.
- Response time is for interactive systems. Turnaround time is for submitted jobs.

#### Terms to remember in words

- Throughput: jobs completed per unit time.
- Turnaround time: submission to completion.
- Response time: request to the first response.

CPU scheduling algorithms are covered in Unit 2. Here, the time slice is the mechanism that makes time-sharing possible.

#### Important Differences

- Multiprogramming is a technique. Time-sharing is a type of system that uses that technique interactively.
- A computer network becomes a distributed system when the nodes cooperate as one service.

#### Important Exam Points

- Comparison table of the four types.
- Diagram of three jobs in memory plus the OS.
- One example of hard real-time and one of soft real-time.

---

