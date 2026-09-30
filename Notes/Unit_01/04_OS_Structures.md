**Navigation:** [Unit index](00_Index.md) · [Previous: 1.3 OS Services and System Calls](03_OS_Services_and_System_Calls.md) · [Next: 1.5 Virtualization and Hypervisors](05_Virtualization_and_Hypervisors.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 1 — OS Fundamentals and Architecture

## Topic 1.4 — Operating System Structures

### Learning Outcomes

- Explain why the internal structure of an operating system matters.
- Describe monolithic, layered, microkernel, and hybrid structures.
- Differentiate these structures using kernel size, communication, speed, and reliability.
- Identify the structure of familiar systems such as traditional UNIX, Linux, Windows, and macOS.

### Prerequisites

Kernel, user mode, and system calls (Topics 1.1 and 1.3).

### Introduction

Operating-system structure means how the kernel and its services are organized inside the software, not the outside shape of a laptop. The organization affects speed, reliability, and how easily a part of the OS can be changed. The four structures are monolithic, layered, microkernel, and hybrid.

### Definition

**Definition:**  
An operating-system structure is the way kernel functions are divided into components and the way those components call or communicate with each other.

### Key Terminology

| Term | Meaning |
| --- | --- |
| Monolithic kernel | One large kernel program in a single address space |
| Layer | A level of software that uses only the level directly beneath it |
| Microkernel | A very small kernel that keeps only the most essential functions |
| User-space server | An OS service that runs as a process outside a microkernel |
| Message passing | Sending a request and receiving a reply between components |
| Hybrid kernel | A design that combines a larger kernel with some microkernel ideas |
| Module | A piece of kernel code that can be added or removed, often at run time |
| Address space | The range of memory a program is allowed to use |

### Basic Concepts

The designer chooses a structure by answering three questions:

1. Which functions run in kernel mode?
2. How does one function invoke another: by a direct call or by a message?
3. How much damage can a faulty component cause?

A direct call inside one kernel is fast. A message to a separate server is slower but can isolate faults.

### Types

| Type | Definition | Characteristics | Example |
| --- | --- | --- | --- |
| Monolithic | Most OS services run together inside one kernel | Fast direct calls, large kernel, a fault can crash the system | Traditional UNIX, Linux |
| Layered | Services are arranged in levels; each level uses the level below | Clean structure, easier debugging of a layer, overhead of crossing layers | THE operating system (Dijkstra) |
| Microkernel | Kernel contains only minimal mechanisms; other services run in user space | Small kernel, message passing, better isolation, extra communication cost | Mach, MINIX, QNX |
| Hybrid | Combines a substantial kernel with selected separation of components | Practical compromise between speed and structure | Windows NT family, macOS (XNU) |

#### Monolithic structure

**Explanation:**  
The kernel is one program. Process management, memory management, the file system, and device drivers call each other as ordinary procedures. They share the kernel address space.

**How it works:**

1. A user process issues a system call.
2. Control enters the single kernel.
3. The requested procedure calls other kernel procedures directly.
4. Drivers and file-system code run in kernel mode.
5. The result returns to the user process.

```text
+--------------------------------------+
|         User applications            |
+--------------------------------------+
                 | system call
+--------------------------------------+
| Monolithic kernel                    |
|  process | memory | files | drivers  |
|  (direct procedure calls)            |
+--------------------------------------+
                 |
+--------------------------------------+
|              Hardware                |
+--------------------------------------+
```

**Important points:**

- There are few mode switches inside one service, so the path is fast.
- A defective driver runs in kernel mode and can corrupt the whole kernel.
- Linux is monolithic, with loadable modules. Modules make it extensible. They do not turn Linux into a microkernel, because a loaded module still runs in kernel space.

**Example:**  
A file read in Linux enters the kernel, calls the file system, which calls the block layer, which calls the disk driver. These are function calls inside the kernel.

#### Layered structure

**Explanation:**  
The system is divided into layers. Layer 0 is the hardware. The top layer is the user interface. A layer may request services only from the layer immediately below it.

**How it works:**

1. The user request enters the top layer.
2. Each layer checks and translates the request.
3. The request moves down, one layer at a time.
4. The bottom layer operates the hardware.
5. Results travel back up the same sequence.

```text
Layer N     User interface
   |
Layer N-1   User programs / libraries
   |
Layer 2     File and device management
   |
Layer 1     Memory and process support
   |
Layer 0     Hardware
```

**Important points:**

- The structure is easy to explain and to test layer by layer.
- Strict layering can be slow because a request crosses many interfaces.
- Deciding which function belongs in which layer is difficult. A file system wants to sit above memory management, but memory management may need disk space for paging.
- The THE system is the classical textbook example of layering.

#### Microkernel structure

**Explanation:**  
The kernel is reduced to mechanisms that must be privileged: low-level process and thread support, address-space management, and inter-process communication. File systems, device drivers, and network services run as servers in user space.

**How it works:**

1. An application needs a file operation.
2. It sends a message, through the microkernel, to the file server.
3. The microkernel transfers the message. It does not itself implement the file system.
4. The file server may send another message to a disk-driver server.
5. The reply travels back by messages.
6. The application receives the result.

```text
 Application          File server         Driver server
 (user mode)          (user mode)         (user mode)
      \                   |                   /
       \                  |                  /
        +----------- microkernel -----------+
                    (kernel mode)
                          |
                       Hardware
```

**Important points:**

- A user-space driver can often fail without destroying the kernel.
- Message passing causes extra traps and copying, so a pure microkernel can be slower than a monolithic kernel.
- MINIX and Mach are standard examples. QNX is a commercial real-time microkernel.

**Example:**  
If a file server in a microkernel crashes, the microkernel can restart that server. In a monolithic kernel, a crashed file system that runs inside the kernel may bring down the machine.

#### Hybrid structure

**Explanation:**  
Commercial systems often refuse both extremes. They keep performance-critical services in the kernel, like a monolithic design, and also separate some subsystems or use a microkernel-derived core, like a microkernel design. This combination is called a hybrid kernel.

**How it works:**

1. The core privileged part handles scheduling, memory, and IPC.
2. Many services that a pure microkernel would place in user space are moved back into kernel space for speed.
3. Selected components remain modular.
4. User programs still enter through system calls.

**Important points:**

- Windows NT and its successors (Windows 2000, XP, 10, 11) are the usual hybrid example. The design was influenced by a small kernel idea, but many services run in kernel mode for performance.
- macOS uses XNU, which combines the Mach microkernel with BSD kernel services. That combination is hybrid.
- Hybrid is a practical engineering compromise. It is not “better” in every row of a comparison. It trades some fault isolation for fewer messages.

**Example:**  
On Windows, a graphics or driver component may run in kernel mode so that frequent calls do not pay a user-space message cost. The overall design is still described as hybrid because of its NT structure, not because every driver is a user process.

### Comparison

| Parameter | Monolithic | Layered | Microkernel | Hybrid |
| --- | --- | --- | --- | --- |
| Where services run | In the kernel | In defined layers, mostly kernel | Essential functions in kernel; servers in user space | Core plus many in-kernel services |
| Internal communication | Direct call | Call to the next lower layer | Message passing | Calls plus some separated components |
| Speed of a service | High | Reduced by layer crossings | Reduced by messages | Close to monolithic for in-kernel paths |
| Effect of a faulty driver or server | Can crash the OS | Can crash the OS if it is inside the kernel | Often limited to that server | Often can crash the OS if the part is in the kernel |
| Ease of extending | Loadable modules help | Add or change a layer carefully | Add a user-space server | Modules and subsystems |
| Textbook example | Traditional UNIX, Linux | THE | Mach, MINIX | Windows NT, macOS XNU |

### Advantages and Limitations

| Structure | Advantages | Limitations |
| --- | --- | --- |
| Monolithic | Fast; simple call path; mature systems such as Linux | Large trusted code; kernel faults are severe |
| Layered | Clear organization; a layer can be tested against the interface below | Layer-crossing cost; some functions do not fit one layer |
| Microkernel | Small privileged core; service faults are more isolated; flexible servers | Message-passing overhead |
| Hybrid | Balances performance and modular design in real products | More complex; isolation is incomplete for in-kernel parts |

### Applications

- Linux servers and Android phones use a monolithic Linux kernel. Android adds its own layers above that kernel; those layers are described with mobile operating systems.
- MINIX and QNX are microkernel systems. MINIX is widely used for study; QNX is used where a fault in one service must not stop the kernel.
- Desktop Windows and macOS use hybrid kernels.
- A layered diagram is still the clearest way to show how an operating system is organized, even when a real kernel is not strictly layered.

### Common Mistakes

- Saying “Linux is a microkernel” because it has modules. Modules are still kernel code. Linux is a modular monolithic kernel.
- Saying “a microkernel contains the file system inside the kernel.” The file system is a user-space server in the pure microkernel approach.
- Drawing layers but allowing every layer to call every other layer. Strict layering uses the adjacent lower layer.
- Treating hybrid as a synonym of microkernel. Hybrid systems put many services back into kernel mode for speed.

### Important Exam Points

- Four structures, one diagram each, one example each.
- Monolithic: direct calls, fast, poor isolation.
- Microkernel: message passing, small kernel, better isolation, slower.
- Linux versus Windows versus macOS classification.
- THE as the layered example.

### University Exam Questions

#### 2-Mark Questions

1. Define a monolithic kernel.
2. Define a microkernel.
3. What is a hybrid kernel? Give one example.
4. State one advantage of the layered approach.
5. Why is Linux not classified as a microkernel?

#### 4/5-Mark Questions

1. Explain the monolithic structure with a diagram.
2. Explain the microkernel structure. How does a file request reach a driver?
3. Differentiate monolithic and microkernel operating systems.
4. Explain the layered structure and one difficulty in assigning functions to layers.

#### 8/10-Mark Questions

1. Explain monolithic, layered, microkernel, and hybrid structures with diagrams and examples.
2. Compare the four structures on communication method, performance, and fault isolation. Classify Linux, Windows, and macOS.

### Practice Problems

#### Easy

1. Name one monolithic kernel.
2. In which structure do most services run as user-space servers?
3. What communication method does a pure microkernel use between an application and a file server?
4. Which classical system is the textbook example of layering?
5. State one reason a monolithic call path is fast.

#### Medium

1. A driver bug overwrites kernel memory and the machine stops. Which structures are exposed to this failure, and which structure reduces it?
2. Draw the path of a disk read in a microkernel.
3. Explain the phrase “modular monolithic” for Linux.
4. Why might paging and the file system be hard to place in a strict layer order?
5. Why do commercial hybrid kernels move some servers back into the kernel?

#### Hard

1. Compare the path of a file read in a monolithic kernel with the path in a pure microkernel. Explain where a mode switch occurs in each path.
2. A designer wants both high speed and restartable device drivers. Which structure is the starting point, and what performance issue must be solved?
3. Classify XNU using its Mach and BSD parts.
4. Explain how loadable modules change administration of a monolithic kernel without changing its privilege model.
5. A layered OS allows Layer 4 to call Layer 1 directly “as an optimization.” Explain which property of layering has been abandoned.

### MCQs

**Q1. In a monolithic kernel, services communicate mainly by:**

A. Direct procedure calls in one kernel address space  
B. Mandatory messages to user-space servers only  
C. Replacing the CPU  
D. User programs editing kernel memory freely  

**Answer:** A  

**Explanation:** Monolithic components share the kernel and call each other directly.

**Q2. A microkernel typically places the file system in:**

A. User space, as a server  
B. The hardware timer only  
C. The bootstrap ROM as the entire file system  
D. Every application, with no server  

**Answer:** A  

**Explanation:** Pure microkernels keep file services outside the privileged kernel.

**Q3. The main cost of the microkernel approach is:**

A. Message passing and extra mode switches  
B. The absence of any communication  
C. A requirement to use vacuum tubes  
D. Forbidding all device drivers  

**Answer:** A  

**Explanation:** Requests travel as messages between servers, which is slower than a direct kernel call.

**Q4. The THE system is an example of:**

A. A layered operating system  
B. A cloud hypervisor  
C. A first-generation plugboard  
D. A file name  

**Answer:** A  

**Explanation:** Dijkstra’s THE system is the classical layered OS example.

**Q5. Linux is best classified as:**

A. A modular monolithic kernel  
B. A pure microkernel with all drivers in user space  
C. A Type 2 hypervisor  
D. An application program  

**Answer:** A  

**Explanation:** Linux runs file systems and drivers in kernel space. Modules can be loaded, but they run in the kernel.

**Q6. Windows NT is commonly classified as:**

A. Hybrid  
B. A pure batch monitor with no kernel  
C. Firmware  
D. A strict five-layer THE copy with no in-kernel services beyond Layer 0  

**Answer:** A  

**Explanation:** The NT family uses a hybrid kernel structure.

**Q7. A faulty user-space server in a microkernel:**

A. Can often be restarted without crashing the microkernel  
B. Always destroys the CPU hardware  
C. Cannot be separated from the kernel because servers are not used  
D. Runs in kernel mode by definition of a server  

**Answer:** A  

**Explanation:** Isolation of servers is the reliability argument for microkernels.

**Q8. Strict layering means a layer calls:**

A. Only the adjacent lower layer  
B. Every layer, including higher layers, without restriction  
C. Only other user applications  
D. The file name of the layer  

**Answer:** A  

**Explanation:** The discipline of layering is that each layer depends only on the interface immediately below it.

### Quick Revision

#### Key Definitions

- Monolithic: one large kernel, direct calls.
- Layered: ordered levels, each using the level below.
- Microkernel: minimal kernel, user-space servers, messages.
- Hybrid: mixture used by Windows NT and macOS XNU.

#### Important Concepts

- Speed comes from fewer crossings and in-kernel calls.
- Reliability comes from keeping services out of kernel mode.
- Linux modules do not change the monolithic classification.

#### Path to remember

- Microkernel file request: application → message → file server → message → driver server → replies.
- A direct call inside a monolithic kernel is the short path. Message passing in a microkernel is the long path.

#### Important Differences

- Monolithic versus microkernel on location of services and on communication.
- Layered versus hybrid: layering is an organization rule; hybrid is a kernel-privilege compromise.

#### Important Exam Points

- Four diagrams.
- Examples: UNIX/Linux, THE, Mach/MINIX, Windows/macOS.
- Fault in a driver: monolithic impact versus microkernel impact.

---

