**Navigation:** [Unit index](00_Index.md) · Previous: — · [Next: 1.2 Types of Operating Systems](02_Types_of_Operating_Systems.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 1 — OS Fundamentals and Architecture

## Topic 1.1 — Operating System Concepts and Generations

### Learning Outcomes

- Explain what an operating system is and why a computer needs one.
- Identify the two views of an operating system: resource manager and extended machine.
- Differentiate user mode and kernel mode.
- Explain the generations of operating systems and the main idea of each generation.
- Relate each generation to the hardware technology of that period.

### Prerequisites

Processor (CPU), main memory, secondary storage, and input/output devices, as covered in Fundamentals of Computer Systems.

### Introduction

An operating system (OS) is the first large software system that starts after the computer is powered on, and it stays in control until the computer is shut down. Application programs such as a browser, a compiler, or a database do not talk to the disk, the keyboard, or the memory chips directly. They request those services from the operating system.

This topic is the foundation of the whole subject. Process scheduling, memory management, file systems, and security all exist because the operating system must manage hardware and provide a usable interface to programs.

### Definition

**Definition:**  
An operating system is system software that manages computer hardware and software resources and provides common services so that application programs can run correctly, safely, and efficiently.

In other words, the operating system is the manager of the machine. It decides which program runs, where data is stored, and how devices are used. It also hides complicated hardware details from the programmer.

### Key Terminology

| Term | Meaning |
| --- | --- |
| Operating system | System software that manages hardware and provides services to applications |
| Kernel | The core part of the OS that runs with full hardware privilege |
| System software | Software that supports the computer itself (OS, drivers, loaders) |
| Application software | Software that performs user tasks (editor, browser, compiler) |
| Bootstrap program | Small firmware program that starts the computer and loads the OS |
| User mode | Restricted execution mode used by applications |
| Kernel mode | Privileged execution mode used by the OS kernel |
| Resource | Anything the OS manages: CPU time, memory, files, devices |
| Abstraction | A simpler interface that hides internal complexity |

### Basic Concepts

#### 1. Two views of an operating system

**Definition:**  
The operating system can be understood as a **resource manager** and as an **extended (virtual) machine**.

As a resource manager, the OS allocates CPU, memory, files, and devices and prevents programs from interfering with each other. As an extended machine, it presents a cleaner interface than the raw hardware. A program can say “read the next record of this file” instead of programming the disk controller.

- Both views describe the same OS. They are two ways of looking at it.
- The resource-manager view is important for scheduling, protection, and sharing.
- The extended-machine view is important for system calls and programmer convenience.

**Example:**  
When two students compile programs at the same time on a lab machine, the OS (resource manager) shares the CPU. When a program calls a file-read service, the OS (extended machine) performs the disk operation and returns the data.

#### 2. Kernel

**Definition:**  
The kernel is the central part of the operating system that always resides in memory and executes in privileged mode.

The kernel is the part that is allowed to execute sensitive instructions: changing memory maps, talking to devices, and switching the CPU from one program to another. Applications are not allowed to execute those instructions.

- It is loaded at boot time and remains in memory.
- It runs in kernel mode.
- It implements process, memory, file, and device management.
- A crash in the kernel usually stops the whole system.

**Example:**  
On Linux, the kernel handles scheduling and device drivers. A text editor runs outside the kernel and requests services through system calls.

#### 3. Dual-mode operation

**Definition:**  
Dual-mode operation means the CPU has at least two modes: **user mode** and **kernel mode**. A mode bit in the processor status indicates the current mode.

Think of a laboratory with a public work area and a locked equipment room. Students work in the public area. Only the lab in-charge may enter the equipment room. User mode is the public area. Kernel mode is the equipment room.

**How it works:**

1. The computer boots. The kernel starts in kernel mode.
2. The kernel loads a user program and switches the CPU to user mode.
3. The user program runs. Privileged instructions are forbidden.
4. When the program needs an OS service, it executes a special trap instruction.
5. The hardware switches to kernel mode and jumps to the OS.
6. After the service, the OS returns the CPU to user mode.

- Privileged instructions (for example, I/O instructions and instructions that change the mode bit) can execute only in kernel mode.
- The mode switch is done by hardware, so a user program cannot simply set itself to kernel mode.
- This separation is the basis of protection.

```text
        User program                         Kernel
        (user mode)                        (kernel mode)
             |                                   ^
             |  system call / interrupt / trap   |
             +---------------------------------->+
             |                                   |
             |<----------------------------------+
             |         return to user mode
```

#### 4. Bootstrap (booting)

**Definition:**  
Booting is the process of starting the computer and loading the operating system into memory.

**How it works:**

1. Power is turned on. The CPU begins execution at a fixed address.
2. The bootstrap program, stored in firmware (ROM, EEPROM, or flash; on PCs this role is played by BIOS or UEFI), runs a hardware check.
3. The bootstrap locates the OS kernel on secondary storage.
4. The kernel is loaded into main memory.
5. The kernel initializes data structures, devices, and the first user process.
6. The system becomes ready for user programs.

**Example:**  
On a PC, UEFI finds a bootable disk and loads the boot loader. The boot loader then loads the Linux or Windows kernel.

### Generations of Operating Systems

**Definition:**  
Generations of operating systems are historical stages of OS development. Each generation is linked to a major change in hardware technology and in the way programs were run.

**Explanation:**  
Early computers had no operating system. A programmer operated the machine directly. As machines became faster and more expensive to leave idle, software was introduced to load jobs, share the CPU, and support many users. Later generations added personal computers, networks, and today’s mobile and cloud systems.

| Generation | Approximate period | Hardware | OS idea | Example systems |
| --- | --- | --- | --- | --- |
| First | about 1945–1955 | Vacuum tubes | No real OS; manual operation | ENIAC, UNIVAC (programmed directly) |
| Second | about 1955–1965 | Transistors | Batch processing | IBM 7094 with IBSYS, early Fortran monitor systems |
| Third | about 1965–1980 | Integrated circuits | Multiprogramming and time-sharing | IBM OS/360, CTSS, MULTICS, early UNIX |
| Fourth | about 1980 onward | VLSI, microprocessors | Personal computers, GUIs, networks | MS-DOS, Windows, Mac OS, Linux |
| Modern extension | 2000s onward | Multicore, mobile SoCs, data centers | Mobile, virtualization, cloud, distributed services | Android, iOS, hypervisor-based clouds |

The usual classification has four generations. Mobile systems, virtualization, and cloud platforms continue the fourth generation. They are discussed later in this unit.

#### First generation — no operating system

**Explanation:**  
Machines used vacuum tubes. Programs were entered with plugboards or punched cards. One user controlled the whole machine. There was no resource sharing and almost no system software.

**Important points:**

- Setup time was large compared with running time.
- The machine sat idle while the next programmer prepared the job.
- Machine language was the normal programming method.

#### Second generation — batch systems

**Explanation:**  
Transistor machines were costly. Operators collected similar jobs (for example, several Fortran compilations) and ran them as a **batch**. A simple monitor program loaded one job after another. The user did not interact with the program while it ran.

**How it works:**

1. Programmers submit card decks to the operator.
2. The operator groups jobs into a batch and places them on tape or in the card reader.
3. The resident monitor loads the next job.
4. The job runs to completion (or until an error).
5. Output is collected and returned later.
6. The monitor automatically loads the following job.

**Important points:**

- CPU idle time caused by manual job setup was reduced.
- There was no interactive debugging.
- Turnaround time (submission to return of output) could be hours.

#### Third generation — multiprogramming and time-sharing

**Explanation:**  
Integrated circuits made computers faster. A single job often waited for I/O, leaving the CPU idle. **Multiprogramming** keeps several jobs in memory. When one job waits for I/O, the CPU runs another. **Time-sharing** extends this idea so that many interactive users each receive a short slice of CPU time.

**Important points:**

- Spooling (Simultaneous Peripheral Operation On-Line) used disks to queue input and output.
- Protection became necessary because several programs shared memory.
- UNIX and the ideas behind MULTICS belong to this generation.

#### Fourth generation — personal computers and networks

**Explanation:**  
Microprocessors made a computer cheap enough for one person. Early PC systems such as MS-DOS were simple single-user systems. Later systems added graphical interfaces, multitasking, and networking. Workstations and servers continued the UNIX line. Linux became a widely used kernel for servers, desktops, and later mobile devices.

**Important points:**

- Convenience for a single user became as important as CPU efficiency.
- Networking connected personal computers to shared servers.
- This generation leads directly to the mobile and cloud systems covered later in the unit.

### Advantages and Limitations of Having an Operating System

| Advantages | Limitations |
| --- | --- |
| Hides hardware complexity from application programmers | The OS itself consumes CPU time and memory |
| Allows safe sharing of CPU, memory, and devices | A kernel fault can stop every application |
| Provides a common service interface (system calls) | Extra layers can reduce the maximum possible speed of raw hardware |
| Enforces protection and accounting | A poorly designed OS can waste resources or be insecure |

### Applications

- Every general-purpose computer, phone, server, and many embedded products runs an OS or a small equivalent kernel.
- Compilers, databases, browsers, and laboratory programs all depend on OS services for files, processes, and I/O.
- Shared lab servers use the OS to run many student programs without letting one program destroy another.

### Common Mistakes

- Calling every program an operating system. A compiler or a browser is application software. The OS is the resource manager underneath.
- Treating the kernel and the whole OS as identical in every sentence. The kernel is the privileged core. Utilities, shells, and libraries are often distributed with the OS but are not all inside the kernel.
- Saying the first generation “used batch operating systems.” Batch monitors belong to the second generation. The first generation had essentially no OS.
- Confusing user mode with “a normal user account.” User mode is a CPU privilege level, not a login name.

### Important Exam Points

- Definition of an operating system (resource manager and extended machine).
- Kernel, user mode, kernel mode, and the mode bit.
- Sequence of booting.
- Generation table: hardware technology plus the OS idea of that generation.
- Batch processing begins in the second generation. Multiprogramming and time-sharing belong to the third.

### University Exam Questions

#### 2-Mark Questions

1. Define an operating system.
2. What is a kernel?
3. Differentiate user mode and kernel mode.
4. What is bootstrapping?
5. Name the hardware technology associated with the second and third generations of operating systems.

#### 4/5-Mark Questions

1. Explain the operating system as a resource manager and as an extended machine.
2. Explain dual-mode operation with a suitable diagram.
3. Describe the generations of operating systems.
4. Why was batch processing introduced? Explain its working.

#### 8/10-Mark Questions

1. Explain the generations of operating systems. For each generation, state the hardware base, the operating-system idea, and one limitation that the next generation reduced.
2. Explain the need for an operating system. Describe kernel, dual-mode operation, and the boot sequence.

### Practice Problems

#### Easy

1. State two goals of an operating system.
2. Who executes privileged instructions: a user program or the kernel?
3. In which generation were vacuum tubes used?
4. What is a batch of jobs?
5. Name the program that loads the kernel at power-on.

#### Medium

1. A machine is fast, but each job waits for card reading. Which generation’s idea reduces CPU idle time, and how?
2. Explain why a user program is not allowed to change the mode bit directly.
3. Draw the boot sequence from power-on to the first user process.
4. Compare the user’s experience in a second-generation batch system and a third-generation time-sharing system.
5. Why did personal computers of the early fourth generation not need the same emphasis on multi-user protection as a mainframe?

#### Hard

1. A university computer of 1968 keeps several jobs in memory and also supports interactive terminals. Identify the OS ideas being used and explain how they work together.
2. Argue, with technical reasons, whether a library function such as `printf` is part of the kernel.
3. Explain how the extended-machine view and the resource-manager view both appear when a program writes a file to disk.
4. A designer removes dual-mode hardware to “simplify” a teaching computer. What protection problems appear?
5. Place Android and a cloud virtual-machine service in the historical line of OS generations without treating them as first-generation systems. Justify the placement.

### MCQs

**Q1. An operating system is best described as:**

A. Application software that edits documents  
B. System software that manages resources and provides services to programs  
C. A high-level programming language  
D. A hardware device controller only  

**Answer:** B  

**Explanation:** The OS manages hardware resources and offers services. Editors and compilers are applications that use those services.

**Q2. Privileged instructions are executed in:**

A. User mode  
B. Kernel mode  
C. Any mode, with no restriction  
D. Only inside a compiler  

**Answer:** B  

**Explanation:** Privileged instructions are reserved for the kernel. Hardware rejects them in user mode.

**Q3. The bootstrap program is stored in:**

A. A user document folder only  
B. Non-volatile firmware so that it is available at power-on  
C. The CPU cache, which is empty at power-on  
D. An application installer  

**Answer:** B  

**Explanation:** Main memory is volatile. The initial startup code must be in firmware (ROM/flash, BIOS/UEFI).

**Q4. Batch operating systems became important in the:**

A. First generation  
B. Second generation  
C. Period before vacuum tubes  
D. Design of mechanical calculators  

**Answer:** B  

**Explanation:** Batch monitors were introduced on transistor machines to reduce idle time between jobs.

**Q5. Multiprogramming and time-sharing are characteristic ideas of the:**

A. First generation  
B. Second generation only, with no memory sharing  
C. Third generation  
D. Period in which computers had no processors  

**Answer:** C  

**Explanation:** Third-generation systems used ICs, kept multiple jobs in memory, and introduced interactive time-sharing.

**Q6. The mode bit is used to:**

A. Store the user’s password  
B. Indicate whether the CPU is in user mode or kernel mode  
C. Count the number of files on disk  
D. Select the programming language  

**Answer:** B  

**Explanation:** The hardware mode bit distinguishes restricted execution from privileged execution.

**Q7. Which statement about the first generation is correct?**

A. It used time-sharing UNIX systems  
B. Machines were operated manually and had essentially no OS  
C. It introduced graphical personal computers  
D. It used cloud hypervisors  

**Answer:** B  

**Explanation:** First-generation vacuum-tube machines were run directly by programmers or operators.

**Q8. The kernel must remain in memory because:**

A. User files cannot be stored on disk  
B. Its services and interrupt handling must be available continuously  
C. The compiler requires the kernel source at every keystroke  
D. Firmware cannot start a computer  

**Answer:** B  

**Explanation:** Device interrupts and system calls can occur at any time, so the kernel stays resident.

### Quick Revision

#### Key Definitions

- Operating system: system software that manages resources and provides services to applications.
- Kernel: privileged core of the OS, resident in memory.
- User mode / kernel mode: restricted mode and privileged mode of the CPU.
- Bootstrapping: loading the kernel from firmware-directed startup.

#### Important Concepts

- Resource-manager view and extended-machine view.
- Dual-mode operation and the trap into the kernel.
- Generations: vacuum tubes (no OS) → transistors (batch) → ICs (multiprogramming, time-sharing) → microprocessors (PCs and networks).

#### Important Algorithms

- Boot sequence: firmware → locate kernel → load kernel → initialize → start first process.

#### Important Differences

- User mode is a CPU privilege. A user account is a login identity.
- Kernel is the privileged core. The full OS distribution also contains tools and libraries.

#### Important Exam Points

- A definition of an operating system includes both the resource-manager view and the extended-machine view.
- Each generation is tied to its hardware and to the main software idea of that period.

---

