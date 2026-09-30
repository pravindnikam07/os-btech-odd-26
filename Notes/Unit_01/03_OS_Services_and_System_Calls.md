**Navigation:** [Unit index](00_Index.md) · [Previous: 1.2 Types of Operating Systems](02_Types_of_Operating_Systems.md) · [Next: 1.4 OS Structures](04_OS_Structures.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 1 — OS Fundamentals and Architecture

## Topic 1.3 — Operating System Services and System Calls

### Learning Outcomes

- List the major services provided by an operating system.
- Explain why application programs use services instead of operating hardware directly.
- Define a system call and describe the steps of a system-call execution.
- Classify system calls for process control, file management, and device management.
- Differentiate a system call from an ordinary library call.

### Prerequisites

Dual-mode operation and the kernel (Topic 1.1).

### Introduction

Users and programs need a stable set of operations: start a program, read a file, write to a device, and stop a program that fails. These are operating-system services. A program requests a privileged service through a **system call**, which is the programming interface between a process and the OS.

### Definition

**Definition:**  
An operating-system service is a function the OS performs for the user, for an application, or for its own efficient and secure operation.

**Definition:**  
A system call is a controlled request by a running program that transfers execution to the kernel so that a privileged service can be performed.

### Key Terminology

| Term | Meaning |
| --- | --- |
| System call | Explicit request from a process to the kernel |
| API | Application Programming Interface; a function library used by programmers |
| Trap | A software interrupt that switches the CPU to kernel mode |
| System-call number | An integer that identifies which service the kernel should run |
| Process | A program in execution |
| File | A named collection of related data stored by the file system |
| Device | A hardware unit such as a disk, keyboard, printer, or network interface |
| Return value | The result or error code passed back to the caller |

### Basic Concepts

#### Services for the user and for the system

OS services fall into two groups: services that help the user directly, and services that keep the system efficient and secure. Silberschatz uses the same grouping.

| Service | Who it mainly helps | What it does |
| --- | --- | --- |
| Program execution | User | Loads a program, runs it, and ends it normally or abnormally |
| I/O operations | User | Performs input and output because devices are shared and privileged |
| File-system manipulation | User | Creates, deletes, reads, writes, and searches files and directories |
| Communication | User | Exchanges information between processes on the same computer or over a network |
| Error detection | User and system | Detects CPU, memory, I/O, and program errors and handles them |
| Resource allocation | System | Assigns CPU, memory, files, and devices to jobs |
| Accounting | System | Records how much of each resource a user or process consumed |
| Protection and security | System | Controls access to resources and defends the system from misuse |

**Protection and security**

- **Protection** is the internal mechanism that decides whether a process may access a resource.
- **Security** is the defense of the system against external and internal threats, including authentication of users.

Security mechanisms are covered in Unit 6. Here, protection and security are simply two of the services an operating system provides.

#### Application programming interface and system call

**Explanation:**  
Most programmers do not write the trap instruction themselves. They call a function in a language library, such as a C library function. That function places arguments in the agreed registers or on the stack, places the system-call number, and executes the trap. The kernel performs the work and returns.

```text
Application
    |
    |  calls library function (API)
    v
Library routine
    |
    |  sets system-call number and arguments
    |  executes trap
    v
Kernel system-call handler
    |
    |  runs the requested service
    v
Return to library, then to application
```

**Important point:**  
A library call that does not enter the kernel is not a system call. A library call becomes a system call only when it traps into the kernel. For example, computing a square root in a math library can be done entirely in user mode. Opening a file cannot.

### How a System Call Works

```text
User process
  ↓
Prepare arguments and system-call number
  ↓
Execute trap instruction
  ↓
Hardware switches to kernel mode and saves the return context
  ↓
Kernel verifies arguments and dispatches using the system-call number
  ↓
Kernel service runs (process, file, or device operation)
  ↓
Result or error code is placed for the caller
  ↓
Hardware returns to user mode at the instruction after the trap
  ↓
User process continues
```

**Step explanation:**

1. The process needs a privileged action, such as reading a file.
2. Arguments identify the resource and the operation. The system-call number selects the service.
3. The trap is the only legal way for the process to enter the kernel.
4. The hardware saves enough state to resume the process later and raises the privilege level.
5. The kernel checks the request. An illegal address or a forbidden file is rejected.
6. The corresponding kernel routine runs.
7. A status value tells the caller whether the call succeeded.
8. The mode bit returns to user mode. The process continues with limited privilege.

### Types of System Calls

System calls are grouped by the kind of work they do. Process control, file management, and device management are the groups used most often. Information maintenance and communication complete the standard classification.

| Type | Purpose | Typical operations | Example calls (UNIX-style names) |
| --- | --- | --- | --- |
| Process control | Create and manage processes | End, abort, load, execute, create, wait, allocate memory | `fork`, `exec`, `exit`, `wait` |
| File management | Operate on files | Create, delete, open, close, read, write, get or set attributes | `open`, `read`, `write`, `close` |
| Device management | Operate on devices | Request a device, release it, read, write | `ioctl`, `read`/`write` on a device |
| Information maintenance | Get or set system data | Time, date, process id, system information | `getpid`, `time`, `alarm` |
| Communication | Exchange data between processes | Send, receive, create a connection | `pipe`, `send`, `recv` |

#### Process control

**Explanation:**  
A running program must be able to start another program, wait for it, and terminate. These actions change OS process tables, so they are system calls.

**Example:**  
A shell reads a command name, creates a new process, and asks the kernel to run that program. The shell then waits until the child finishes. Creation, execution, and waiting are process-control system calls.

**Important points:**

- Ending a process is also a system call because the kernel must release memory and open files.
- A process that divides into a parent and a child requires kernel help. The user program cannot safely copy the process table by itself.

#### File management

**Explanation:**  
Files have names, locations on disk, permissions, and current read/write positions. The kernel keeps this information. A user program requests create, open, read, write, and close operations.

**Example:**  
A C program opens `marks.txt`, reads bytes into a buffer, and closes the file. `open`, `read`, and `close` enter the kernel. The kernel checks permission and performs the disk I/O.

**Important points:**

- The file name is a user-level identifier. The kernel converts it to the actual storage location.
- Permission is checked inside the kernel, not left to the application.

#### Device management

**Explanation:**  
Devices are shared and their controllers accept only privileged commands. The OS presents devices so that a program can request, use, and release them. In UNIX, many devices appear as special files, so `read` and `write` are used, while control operations use a device-control call.

**Example:**  
A program sends a page to a printer. It requests the printer, writes the data, and releases the printer. The kernel queues the job if another print is already active.

**Important points:**

- User programs do not program the device controller registers directly.
- Device system calls and file system calls look similar when the OS uses a uniform interface. The destination is a device rather than a regular file.

### Worked Example

#### Example — Reading the first byte of a file

**Problem:**  
Describe what happens when a user program reads the first byte of an existing file named `data.txt`.

**Given:**  
The program is in user mode. The file exists and the user has read permission.

**Approach:**  
Trace the API, the trap, the kernel checks, and the return.

**Step 1:**  
The program calls a library read routine with the file name or with a file descriptor previously returned by open.

**Step 2:**  
The library places the system-call number for read and the arguments (descriptor, buffer address, count = 1) and executes the trap.

**Step 3:**  
The CPU switches to kernel mode. The kernel checks that the descriptor is valid, the buffer address is inside the process address space, and the file was opened for reading.

**Step 4:**  
If the byte is not already in a memory buffer, the kernel asks the disk device to transfer the required block, waits for the device, and copies one byte to the user buffer.

**Step 5:**  
The kernel sets the return value to 1 (one byte read) and returns to user mode.

**Solution:**  
The user program receives the byte in its buffer and the count 1. The program never issues disk-controller commands itself.

**Final Answer:**  
`open`/`read` system calls, kernel permission and address checks, device transfer if needed, then return of the byte and the count.

### Advantages and Limitations

| Advantages of the system-call interface | Limitations |
| --- | --- |
| One stable way for every program to request services | A trap and argument check add overhead |
| Kernel can check permissions before any privileged action | A buggy kernel service still affects the caller |
| Hardware differences can be hidden behind the same calls | Programs depend on the OS interface; calls differ across operating systems |
| Errors are reported through defined return codes | Frequent calls in a tight loop can slow a program |

### Applications

- Shells and command interpreters use process-control calls to start commands.
- Editors and compilers use file calls to read source and write output.
- Print spoolers and media players use device calls.
- Every ordinary application on Linux, Windows, Android, or iOS reaches hardware through this interface, even when a framework hides the call.

### Comparison

| Parameter | System call | Ordinary library call |
| --- | --- | --- |
| Privilege change | Yes, switches to kernel mode | No, stays in user mode |
| Who implements the service | Kernel | User-level library |
| Purpose | Protected access to OS-managed resources | Convenience routines, sometimes wrapping a system call |
| Cost | Trap plus kernel checks | A normal function call, if no trap is made |
| Example | `open`, `read`, `fork` | A pure arithmetic helper that does not enter the kernel |

### Common Mistakes

- Writing that a system call is “a call from the OS to the user.” The request goes from the process to the kernel.
- Listing `printf` itself as the system call. `printf` is a library function. It may eventually use a write system call.
- Forgetting argument checking. The kernel must not trust addresses supplied by the user.
- Mixing the five classes. Process control is not file management. Device management is not information maintenance.

### Important Exam Points

- Definition of a system call.
- Numbered steps of trap → dispatch → service → return.
- At least three examples each for process, file, and device calls.
- Table of user services and system services.
- Diagram: application → API → trap → kernel → return.

### University Exam Questions

#### 2-Mark Questions

1. Define a system call.
2. What is the purpose of the system-call number?
3. List four services provided by an operating system.
4. Give two examples of process-control system calls.
5. Differentiate protection and security in one or two lines.

#### 4/5-Mark Questions

1. Explain the steps involved in handling a system call.
2. Explain file-management system calls with examples.
3. Explain device-management system calls. Why must device control be privileged?
4. Differentiate a system call and a library function.

#### 8/10-Mark Questions

1. Explain operating-system services. Separate services visible to the user from services required for efficient and secure operation.
2. Describe the types of system calls. Explain process, file, and device management calls with a diagram of the path from a user program into the kernel.

### Practice Problems

#### Easy

1. Name the instruction type that switches a process into the kernel for a service.
2. Give two file-management system calls.
3. Which service records resource usage: accounting or compilation?
4. State one reason I/O is an OS service.
5. Does a system call run in user mode or kernel mode during the service?

#### Medium

1. A student says “`printf` is a system call.” Correct the statement precisely.
2. List the checks the kernel should perform before writing to a file.
3. Classify these operations: create process, get time, read disk block, send a message.
4. Why is “end process” a system call rather than a simple return from `main` with no kernel involvement?
5. Explain resource allocation with CPU and printer as examples.

#### Hard

1. Trace `fork` followed by `exec` from the parent shell’s point of view. Identify each system-call class.
2. A process passes a buffer address that belongs to another process. At which step must this be stopped, and why?
3. Explain how the same read interface can apply to a regular file and to a device, and where the handling differs inside the kernel.
4. Describe what accounting and protection each contribute when many students use one time-sharing server.
5. A designer allows user programs to write device-controller registers “for speed.” Explain the protection failure.

### MCQs

**Q1. A system call is:**

A. A request from a user process that causes entry into the kernel  
B. A hardware fan controller  
C. A call from the kernel to a compiler  
D. An unconditional jump that stays in user mode  

**Answer:** A  

**Explanation:** The process traps into the kernel to request a privileged service.

**Q2. Which of the following is a process-control system call?**

A. Create or terminate a process  
B. Only change the desktop wallpaper file’s pixels in user memory  
C. Compute a square of an integer with no OS help  
D. Rename the CPU chip  

**Answer:** A  

**Explanation:** Creating and terminating processes updates kernel process data and requires a system call.

**Q3. `open`, `read`, and `close` are examples of:**

A. File-management system calls  
B. CPU manufacturing steps  
C. Bootstrap firmware commands only  
D. Mathematical operators  

**Answer:** A  

**Explanation:** These operations manipulate files through the kernel.

**Q4. Device-management calls are privileged because:**

A. Device controllers are shared hardware and must be programmed safely  
B. Devices do not exist on computers  
C. Files cannot be read  
D. User mode is faster for raw register writes by definition in all machines  

**Answer:** A  

**Explanation:** Direct device programming by any user would break sharing and protection.

**Q5. The usual path of a programmer’s request is:**

A. Application → API/library → trap → kernel  
B. Kernel → user hardware switch → compiler power supply  
C. Disk → user mode with no checks → kernel source editor  
D. Application → device controller registers directly, as the normal protected path  

**Answer:** A  

**Explanation:** The API prepares the system call. The trap enters the kernel.

**Q6. Accounting as an OS service means:**

A. Keeping records of resource usage  
B. Teaching financial accounting as a university subject  
C. Removing all passwords  
D. Disabling the scheduler  

**Answer:** A  

**Explanation:** The OS records CPU time, storage, and similar usage.

**Q7. Error detection is required because:**

A. Hardware and programs can fail, and the OS must notice and respond  
B. Correct programs make hardware infallible  
C. System calls cannot return a status  
D. Memory never stores data  

**Answer:** A  

**Explanation:** The OS detects and handles CPU, memory, I/O, and program errors.

**Q8. Which statement is correct?**

A. Every library function is a system call  
B. A system call runs its service in kernel mode  
C. File names are written directly into the disk controller by user code in a protected OS  
D. The system-call number is optional and never used  

**Answer:** B  

**Explanation:** The kernel service executes in kernel mode. Not every library function traps into the kernel.

### Quick Revision

#### Key Definitions

- System call: controlled entry from a process into the kernel for a service.
- OS services: program execution, I/O, file operations, communication, error detection, resource allocation, accounting, protection and security.

#### Important Concepts

- API prepares arguments. The trap changes mode. The kernel dispatches on a number.
- Process, file, and device calls are the main classes. Information maintenance and communication complete the standard list.
- A system call costs a privilege switch, a check of the arguments, and the work of the service itself.

#### Important Differences

- System call enters the kernel. A pure library call does not.
- Protection is internal access control. Security includes defense against threats.

#### Important Exam Points

- The path from the application through the API into the kernel, and back.
- UNIX-style names such as `open`, `read`, `fork`, and `exit`, together with what each one does.

---

