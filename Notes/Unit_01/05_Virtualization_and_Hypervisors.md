**Navigation:** [Unit index](00_Index.md) · [Previous: 1.4 OS Structures](04_OS_Structures.md) · [Next: 1.6 Cloud OS](06_Cloud_OS.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 1 — OS Fundamentals and Architecture

## Topic 1.5 — Virtualization and Hypervisors

### Learning Outcomes

- Explain virtualization and the role of a hypervisor.
- Differentiate Type 1 and Type 2 hypervisors.
- Identify VMware ESXi, Xen, and KVM as Type 1 systems and VirtualBox and VMware Workstation as Type 2 systems.
- Explain how a guest operating system relates to the host and the hardware.

### Prerequisites

Kernel, user and kernel mode, and the idea that the OS controls the hardware (Topics 1.1 and 1.4).

### Introduction

A normal OS expects to own the CPU, the memory, and the devices. Virtualization inserts a software layer that creates the appearance of several separate machines on one physical computer. Each appearance is a **virtual machine**. The software layer is the **hypervisor**, also called a virtual machine monitor (VMM).

This topic connects operating systems to data centers and to the cloud services in the next topic.

### Definition

**Definition:**  
Virtualization is a technique that presents virtual versions of computing resources, such as a complete computer, so that software can run as if it had those resources to itself.

**Definition:**  
A hypervisor is the software layer that creates, runs, and manages virtual machines and allocates real hardware to them.

### Key Terminology

| Term | Meaning |
| --- | --- |
| Virtual machine (VM) | A software-created computer that runs a guest OS |
| Guest OS | The operating system installed inside a VM |
| Host OS | The operating system already running on the hardware, used by a Type 2 hypervisor |
| Hypervisor / VMM | The manager of virtual machines |
| Type 1 hypervisor | Runs directly on the hardware (bare metal) |
| Type 2 hypervisor | Runs as an application on a host OS |
| Hardware-assisted virtualization | CPU features (Intel VT-x, AMD-V) that make virtualization correct and faster |
| Consolidation | Running several VMs on one physical server |

### Basic Concepts

#### Why the OS cannot be fooled by a simple copy

An operating system executes privileged instructions. If two guest kernels both believe they are in kernel mode, both will try to control the same real devices. The hypervisor must intercept those sensitive actions and perform them safely on behalf of the correct guest.

**How it works:**

1. The hypervisor keeps control of the real hardware.
2. It creates a VM with a virtual CPU, virtual memory, and virtual devices.
3. The guest OS boots inside that VM and runs its own applications.
4. Ordinary guest user instructions run directly on the CPU when it is safe to do so.
5. Sensitive guest kernel instructions are trapped to the hypervisor.
6. The hypervisor checks the request, updates that guest’s virtual hardware state, and uses the real device if needed.
7. The guest continues. It sees the behavior of a private machine.

```text
Applications
    |
Guest operating system
    |
Hypervisor  (allocates real CPU, memory, devices)
    |
Physical hardware
```

- Each guest has its own kernel and its own user processes.
- A fault in one guest should not directly corrupt another guest’s memory.
- The hypervisor is part of the trusted computing base because it controls all guests.

### Types

| Type | Definition | Characteristics | Example |
| --- | --- | --- | --- |
| Type 1 (bare metal) | Hypervisor runs directly on the hardware | No general-purpose host OS underneath; used on servers; lower overhead | VMware ESXi, Xen, KVM |
| Type 2 (hosted) | Hypervisor runs on top of a host operating system | Easier on a desktop; host OS uses the devices; extra layer | Oracle VirtualBox, VMware Workstation, VMware Fusion |

#### Type 1 hypervisor

**Explanation:**  
The physical machine boots into the hypervisor. Guest operating systems run above it. This is the common design in server rooms because the extra host operating system is not needed.

```text
+-------------+  +-------------+  +-------------+
| Guest OS    |  | Guest OS    |  | Guest OS    |
| (VM 1)      |  | (VM 2)      |  | (VM 3)      |
+-------------+  +-------------+  +-------------+
|          Type 1 hypervisor                  |
+---------------------------------------------+
|            Physical hardware                |
+---------------------------------------------+
```

**Examples:**

| System | Type | Remark |
| --- | --- | --- |
| VMware ESXi | Type 1 | Bare-metal server hypervisor from VMware. VMware Workstation is a different product and is Type 2 |
| Xen | Type 1 | Bare-metal hypervisor. A privileged guest called Dom0 manages devices and other guests |
| KVM | Type 1 | Kernel-based Virtual Machine. It is a Linux kernel module that uses Intel VT-x or AMD-V. It turns the Linux kernel into a hypervisor. QEMU is often used with KVM to emulate devices |

**Important points about KVM:**

- KVM is not a second operating system sitting on top of Linux in the Type 2 sense. The virtualization function is inside the host Linux kernel, and guests run on the hardware through that kernel. KVM is a **Type 1** hypervisor: write it as “KVM, Type 1, Linux kernel module.”

**Important points about Xen:**

- Xen itself is the thin hypervisor next to the hardware.
- Dom0 is a privileged virtual machine that starts other guests (DomU).
- Device access is handled through this privileged domain.

#### Type 2 hypervisor

**Explanation:**  
The user installs a normal OS, then installs the hypervisor as a program. The hypervisor asks the host OS for CPU time, memory, and disk files that represent virtual disks. This is convenient for laboratories and laptops.

```text
+------------------+   +------------------+
| Guest OS (VM 1)  |   | Guest OS (VM 2)  |
+------------------+   +------------------+
|        Type 2 hypervisor (application)  |
+-----------------------------------------+
|              Host operating system      |
+-----------------------------------------+
|              Physical hardware          |
+-----------------------------------------+
```

**Examples:**

| System | Type | Remark |
| --- | --- | --- |
| Oracle VM VirtualBox | Type 2 | Runs on Windows, Linux, or macOS host |
| VMware Workstation | Type 2 | Desktop hosted hypervisor. VMware ESXi is a different product and is Type 1 |

**How a Type 2 VM obtains a disk:**

1. The guest thinks it has a hard disk.
2. The hypervisor stores that disk as a large file on the host file system.
3. A guest disk write becomes a write to that file, performed through the host OS.
4. The guest does not see the host file name.

### Comparison

| Parameter | Type 1 | Type 2 |
| --- | --- | --- |
| Position | On bare hardware | On a host OS |
| Host OS required | No general-purpose host under the hypervisor | Yes |
| Typical use | Servers, cloud, data centers | Desktops, student labs, testing |
| Overhead | Lower, because there is no extra host OS layer | Higher, because requests can pass through the host |
| Device control | Hypervisor (or a privileged domain such as Xen Dom0) | Host OS drivers |
| Examples | VMware ESXi, Xen, KVM | VirtualBox, VMware Workstation |

### Advantages and Limitations

| Advantages | Limitations |
| --- | --- |
| Several OS environments share one physical machine | The hypervisor is a critical layer; its failure stops the guests |
| A guest can be paused, copied, or moved as a unit | Sensitive instructions must be intercepted, which adds overhead |
| Isolation between guests supports testing and multi-tenant servers | A weak hypervisor or shared hardware can still leak information if poorly designed |
| Students can run Linux and Windows on one laptop | Type 2 performance is limited by the host operating system |

### Applications

- Server consolidation in a data center: many lightly used servers become VMs on one host.
- Software testing: a program is tested on a guest without reformatting the laboratory PC.
- Cloud computing: AWS, Azure, and Google start customer VMs using hypervisor technology (next topic).
- Security laboratories: untrusted software runs inside a guest.

### Common Mistakes

- Classifying all VMware products as one type. ESXi is Type 1. Workstation and Fusion are Type 2.
- Classifying VirtualBox as Type 1. It needs a host OS, so it is Type 2.
- Classifying KVM as Type 2 only because Linux is installed. KVM is Type 1: the Linux kernel itself acts as the hypervisor.
- Saying the guest OS runs “instead of” a kernel. The guest has its own kernel. The hypervisor sits below it.
- Confusing dual boot with virtualization. Dual boot runs one OS at a time. Virtualization runs guests concurrently.

### Important Exam Points

- Definitions of virtualization, VM, guest, and hypervisor.
- Two diagrams: Type 1 and Type 2.
- Example table: ESXi, Xen, KVM, VirtualBox, and the VMware desktop exception.
- One sentence on Xen Dom0 and one on KVM as a kernel module.
- Hardware assistance: VT-x / AMD-V.

### University Exam Questions

#### 2-Mark Questions

1. Define a hypervisor.
2. What is a guest operating system?
3. Draw the layer order of a Type 1 hypervisor.
4. Classify VirtualBox and Xen.
5. State one advantage of virtualization.

#### 4/5-Mark Questions

1. Differentiate Type 1 and Type 2 hypervisors with diagrams.
2. Explain how KVM and Xen provide virtualization.
3. Explain why privileged instructions of a guest must be intercepted.
4. Compare VMware ESXi and VMware Workstation.

#### 8/10-Mark Questions

1. Explain virtualization. Describe Type 1 and Type 2 hypervisors with examples of VMware, Xen, KVM, and VirtualBox.
2. With diagrams, explain how a guest operating system runs on a Type 1 hypervisor and on a Type 2 hypervisor. State where device access is handled in Xen.

### Practice Problems

#### Easy

1. Which type of hypervisor runs on bare metal?
2. Name the privileged Xen domain that manages other guests.
3. Is VirtualBox Type 1 or Type 2?
4. What is a virtual machine?
5. Name one CPU feature that assists virtualization.

#### Medium

1. A laptop runs Windows. Inside an application, a student boots Ubuntu. Classify the hypervisor style.
2. A data-center server boots directly into ESXi and then starts ten Linux guests. Draw the layers.
3. Why is a guest kernel not allowed to program the physical disk controller directly?
4. Explain KVM’s relationship to the Linux kernel in two or three sentences suitable for a 4-mark answer.
5. Differentiate dual boot and virtualization.

#### Hard

1. Compare the path of a guest disk write on ESXi and on VirtualBox.
2. Explain the role of Dom0 in Xen and why the other guests are called unprivileged domains.
3. A lab instructor claims “KVM is Type 2 because the command is issued from Linux.” Write the correction.
4. Why can several guests share one CPU without sharing one kernel data structure?
5. A hypervisor crash and a guest crash do not have the same effect. Explain both.

### MCQs

**Q1. A hypervisor is:**

A. Software that creates and manages virtual machines  
B. A user document  
C. A kind of mechanical batch card  
D. The guest’s word processor  

**Answer:** A  

**Explanation:** The hypervisor, or VMM, manages VMs and real hardware.

**Q2. Type 1 hypervisors run:**

A. Directly on the hardware  
B. Only as a browser plug-in with no hardware control  
C. Inside a guest spreadsheet  
D. Without any CPU  

**Answer:** A  

**Explanation:** Type 1 is bare-metal virtualization.

**Q3. Oracle VirtualBox is a:**

A. Type 2 hypervisor  
B. Type 1 data-center hypervisor that replaces the host OS  
C. File system  
D. CPU scheduling algorithm  

**Answer:** A  

**Explanation:** VirtualBox is installed on a host operating system.

**Q4. Xen and KVM are classified as:**

A. Type 1 hypervisors  
B. Type 2 desktop applications only, with no privileged role  
C. Batch monitors  
D. Device drivers with no virtualization role  

**Answer:** A  

**Explanation:** Both are bare-metal designs. KVM is implemented as a Linux kernel module.

**Q5. VMware ESXi differs from VMware Workstation because:**

A. ESXi is Type 1 and Workstation is Type 2  
B. They are the same layer with two names  
C. Workstation runs on bare metal and ESXi cannot run guests  
D. ESXi is a programming language  

**Answer:** A  

**Explanation:** The product family contains both a server hypervisor and hosted desktop hypervisors.

**Q6. In Xen, Dom0 is:**

A. The privileged domain that manages devices and other guests  
B. A user file extension  
C. The name of a Type 2 host game  
D. A page-replacement algorithm  

**Answer:** A  

**Explanation:** Dom0 is the privileged control domain of Xen.

**Q7. The guest operating system:**

A. Runs inside the virtual machine and has its own kernel  
B. Replaces the hypervisor and owns every guest’s memory with no control  
C. Is stored only in the mode bit  
D. Cannot run applications  

**Answer:** A  

**Explanation:** Each VM boots its own OS above the hypervisor.

**Q8. Hardware-assisted virtualization (VT-x or AMD-V) is used to:**

A. Support correct and efficient execution of guest privileged operations  
B. Remove the need for any CPU  
C. Replace the file system of every guest with punch cards  
D. Disable all isolation  

**Answer:** A  

**Explanation:** These CPU extensions help the hypervisor run guests safely and with less software emulation.

### Quick Revision

#### Key Definitions

- Virtualization: virtual resources that behave like a private machine.
- Hypervisor: manager of VMs.
- Type 1: on bare metal. Type 2: on a host OS.

#### Important Concepts

- ESXi, Xen, KVM → Type 1.
- VirtualBox and VMware Workstation → Type 2.
- Xen Dom0 is privileged. KVM is a Linux kernel module using VT-x/AMD-V.

#### Path to remember

- Guest sensitive instruction → trap to hypervisor → check → emulate or perform on real hardware → return to guest.
- A Type 1 stack has one fewer software layer than a Type 2 stack, because there is no host operating system under the hypervisor.

#### Important Differences

- Type 1 versus Type 2 diagrams.
- Dual boot (one OS at a time) versus virtualization (concurrent guests).

#### Important Exam Points

- Name the VMware product. ESXi and Workstation are not the same type.
- Both stacks: Type 1 on bare hardware, and Type 2 on a host operating system.

---

