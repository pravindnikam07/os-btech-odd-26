**Navigation:** [Unit index](00_Index.md) · [Previous: 1.7 Mobile OS](07_Mobile_OS.md) · Next: —

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 1 — OS Fundamentals and Architecture

# Unit Summary

Unit 1 explains what an operating system is, how OS ideas developed, which major kinds of systems exist, how programs request services, how the kernel is organized, and how virtualization extends that organization to desktops, clouds, and phones.

| Topic | Key Concept | Important Formula/Algorithm | Complexity/Key Point |
| --- | --- | --- | --- |
| OS concepts and generations | OS as resource manager and extended machine; dual mode; boot | Boot: firmware → kernel → first process | First: no OS; second: batch; third: multiprogramming and time-sharing; fourth: PCs and networks |
| Types of OS | Batch, time-sharing, distributed, real-time | Multiprogramming switch when a job waits; time slice for interactive use | Hard real-time guarantees deadlines; soft real-time tolerates occasional misses |
| Services and system calls | User services and system services; trap into the kernel | Prepare number and arguments → trap → dispatch → return | Required classes: process, file, device |
| OS structures | Monolithic, layered, microkernel, hybrid | Microkernel path uses messages between servers | Linux: modular monolithic; Windows and macOS: hybrid; THE: layered; Mach/MINIX: microkernel |
| Virtualization | Hypervisor manages guest OS instances | Sensitive guest instruction traps to the hypervisor | Type 1: ESXi, Xen, KVM; Type 2: VirtualBox, VMware Workstation |
| Cloud VM services | Guest OS on provider hardware | Request → place VM → boot image | EC2, Azure VMs, GCE; customer manages the guest |
| Mobile OS | Phone OS stacks | Android: app → HAL → Linux; iOS: request → Core OS / XNU | Android: Linux + HAL; iOS: Darwin + Core OS |

### Important Definitions

- Operating system, kernel, user mode, kernel mode.
- Batch, time-sharing, distributed, and real-time OS.
- System call.
- Monolithic kernel, layered OS, microkernel, hybrid kernel.
- Hypervisor, Type 1, Type 2, guest OS.
- Cloud VM instance and image.
- HAL, Darwin, Core OS.

### Important Formulas

Throughput, turnaround time, and response time are remembered as definitions. This unit does not use a numerical formula for them.

### Important Algorithms

- Boot sequence.
- System-call handling sequence.
- Microkernel request path.
- Guest instruction interception by a hypervisor.
- Cloud instance start sequence.
- Android hardware-request path.

### Important Diagrams

- Dual-mode trap and return.
- Three jobs in memory under multiprogramming.
- Monolithic kernel, layers, and microkernel servers.
- Type 1 and Type 2 hypervisor stacks.
- Cloud guest above a provider hypervisor.
- Android stack and iOS stack.

### Important Comparisons

- Batch versus time-sharing versus distributed versus real-time.
- System call versus library call.
- Monolithic versus microkernel.
- Type 1 versus Type 2.
- Android versus iOS kernels and named layers.

### Most Important Exam Topics

- Definition of an OS with both views, plus user mode and kernel mode.
- Generation table and the four OS types, including hard and soft real-time.
- System-call steps and process, file, and device examples.
- Four OS structures with examples: Linux, THE, Mach/MINIX, Windows/macOS.
- Type 1 and Type 2 diagrams: ESXi, Xen, and KVM are Type 1; VirtualBox and VMware Workstation are Type 2.
- EC2, Azure VMs, and GCE as VM services, with the guest OS separated from the provider hypervisor.
- Android Linux kernel and HAL; iOS Darwin, XNU, and Core OS.

