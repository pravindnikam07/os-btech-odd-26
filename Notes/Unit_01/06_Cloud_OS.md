**Navigation:** [Unit index](00_Index.md) · [Previous: 1.5 Virtualization and Hypervisors](05_Virtualization_and_Hypervisors.md) · [Next: 1.7 Mobile OS](07_Mobile_OS.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 1 — OS Fundamentals and Architecture

## Topic 1.6 — Cloud OS Environments

### Learning Outcomes

- Explain what a cloud virtual-machine service provides to a user.
- Describe AWS EC2, Azure Virtual Machines, and Google Compute Engine.
- Relate these services to Type 1 virtualization.
- Differentiate a guest OS inside a cloud VM from the cloud provider’s management software.

### Prerequisites

Virtualization and Type 1 hypervisors (Topic 1.5).

### Introduction

AWS EC2, Azure Virtual Machines, and Google Compute Engine are **infrastructure services** that rent virtual computers. They are not a single operating-system kernel in the way Linux is. Each one is a cloud platform that uses a hypervisor to run the customer’s own operating system inside a virtual machine.

The customer chooses an OS image (for example, Linux or Windows). That guest OS runs inside a VM. The provider’s control software schedules the VM onto physical servers, attaches virtual disks and networks, and isolates customers.

### Definition

**Definition:**  
A cloud virtual-machine service is a provider-managed service that creates virtual computers on shared data-center hardware and lets customers boot and administer their own guest operating systems on those virtual computers.

### Key Terminology

| Term | Meaning |
| --- | --- |
| Cloud | Large provider data centers offering computing resources over the network |
| IaaS | Infrastructure as a Service: the provider supplies virtual machines, storage, and networks |
| Instance | One running virtual machine in a cloud service |
| Image | A stored template of a disk containing an operating system, used to boot an instance |
| AWS EC2 | Amazon Elastic Compute Cloud, Amazon’s virtual-server service |
| Azure VM | A virtual machine in Microsoft Azure |
| GCE | Google Compute Engine, Google Cloud’s virtual-server service |
| Tenant | A customer whose VMs share the provider’s physical machines with other customers |

### Basic Concepts

#### What the customer manages and what the provider manages

| Layer | Customer | Provider |
| --- | --- | --- |
| Applications and data | Yes | No, except optional managed services |
| Guest operating system | Yes (updates, users, software) | Provides images; does not daily-operate the guest for a basic VM |
| Hypervisor and physical server | No | Yes |
| Data-center network, power, cooling | No | Yes |

This division is what “cloud OS” means here. The guest OS is still Linux, Windows, or another ordinary OS. The cloud platform is the management and virtualization layer around that guest.

#### How an instance starts

```text
Customer requests an instance (size, region, image)
        ↓
Cloud control service accepts the request
        ↓
Scheduler selects a physical host with capacity
        ↓
Type 1 hypervisor starts a VM
        ↓
Guest OS boots from the chosen image
        ↓
Virtual disk and virtual network are attached
        ↓
Customer connects (for example, SSH or remote desktop)
```

**Important points:**

- “Elastic” in EC2 means the number and size of instances can change as demand changes.
- Inside the VM, the guest OS still has processes, files, system calls, and users.
- Isolation between tenants depends on the hypervisor and on the provider’s network controls.

### AWS EC2, Azure Virtual Machines, and Google Compute Engine

| Service | Provider | What it provides |
| --- | --- | --- |
| AWS EC2 | Amazon Web Services | Launches elastic virtual servers (instances) from machine images. The instance runs a guest OS on AWS virtualized hardware |
| Azure Virtual Machines | Microsoft Azure | Creates Windows or Linux VMs in Azure. The VM is the customer’s computer; Azure manages the physical host and hypervisor |
| GCE | Google Compute Engine | Google Cloud’s IaaS virtual-machine service. Instances boot from images onto Google’s virtualized infrastructure |

These three services follow the same idea. The difference among them is the provider and the service name, not a different kind of operating system.

#### AWS EC2

**Explanation:**  
A user selects a region, an instance type (how much CPU and memory), and an Amazon Machine Image. EC2 places the instance on a physical server in that region. The user administers the guest OS. Stopping the instance releases the running VM according to the service rules; the image and disks remain as stored data if they were kept on persistent storage.

**Example:**  
A department starts one Linux instance for a web assignment. Students use the guest Linux OS. Amazon runs the hypervisor and the data center.

#### Azure Virtual Machines

**Explanation:**  
Azure VMs provide the same pattern in Microsoft’s cloud. A user can select a Windows Server image or a Linux image. The guest OS is managed by the customer. Azure’s fabric and hypervisor manage placement, virtual disks, and virtual network interfaces.

**Example:**  
A laboratory that needs Windows Server for one experiment starts an Azure VM, signs in, and installs the required software inside that guest.

#### Google Compute Engine

**Explanation:**  
GCE is Google Cloud’s compute VM service. Instances are created from public or custom images and attached to persistent disks and VPC networks. The guest OS runs above Google’s virtualization infrastructure.

**Example:**  
A student project boots an Ubuntu image on GCE, copies the project files to the virtual disk, and runs them as ordinary Linux processes inside the guest.

### Comparison

| Parameter | Ordinary installed OS | Cloud VM service (EC2, Azure VM, GCE) |
| --- | --- | --- |
| Where it runs | On hardware the user owns or a local hypervisor | On provider hardware in a data center |
| Who manages the physical machine | The owner | The cloud provider |
| Who manages the guest OS | The owner | The customer of the VM |
| How capacity changes | Buy or install more hardware | Request more or larger instances |
| Examples | Linux on a lab PC | AWS EC2, Azure VMs, GCE |

### Advantages and Limitations

| Advantages | Limitations |
| --- | --- |
| A VM can be created in minutes without buying a server | The customer depends on the provider’s availability and network |
| CPU, memory, and disk size can be changed as the offering allows | Recurring cost, and a wrong size wastes money |
| The same guest OS skills (Linux or Windows) still apply | The provider’s hypervisor and management layer are outside the customer’s control |
| Isolation supports many tenants on one physical fleet | Shared hardware requires strong virtualization and network isolation |

### Applications

- Hosting a course website on a Linux EC2, Azure, or GCE instance.
- Running a short experiment that needs many machines for a few hours.
- Providing a Windows environment to a user whose personal computer runs another OS.
- Scaling a service by adding instances when traffic increases.

### Common Mistakes

- Writing that EC2, Azure VM, or GCE is itself the guest kernel. Each is a service that runs a guest kernel inside a VM.
- Describing the cloud as “a Type 2 hypervisor on a laptop.” These services are data-center platforms built on bare-metal virtualization.
- Claiming the provider daily administers every file inside the customer’s guest. On a basic VM, patching the guest OS is the customer’s job.
- Mixing these services with cloud products that never give the user a virtual machine. EC2, Azure Virtual Machines, and GCE each provide a VM.

### Important Exam Points

- Expansion of EC2, Azure VM, and GCE.
- Diagram: customer guest OS → provider hypervisor → physical host.
- IaaS division of responsibility.
- The word elastic: capacity can grow and shrink.
- One example of launching an instance from an image.

### University Exam Questions

#### 2-Mark Questions

1. Expand EC2 and GCE.
2. What is a cloud VM instance?
3. What is a machine image?
4. Who manages the physical server in AWS EC2: the customer or the provider?
5. Name the Azure service that provides virtual machines.

#### 4/5-Mark Questions

1. Explain AWS EC2 as a cloud virtual-machine service.
2. Compare what the customer manages and what the provider manages in a cloud VM.
3. Explain how virtualization is used by EC2, Azure VMs, and GCE.
4. Describe the steps to start a cloud VM instance.

#### 8/10-Mark Questions

1. Explain cloud VM services with reference to AWS EC2, Azure Virtual Machines, and Google Compute Engine. Show the relationship between the guest OS and the hypervisor.
2. Differentiate a locally installed operating system from a cloud VM. Discuss advantages and limitations of EC2, Azure VMs, and GCE.

### Practice Problems

#### Easy

1. Expand AWS and GCE.
2. Is a basic EC2 instance a physical laptop owned by the student?
3. What boots from a machine image?
4. Name two guest operating systems a customer might select.
5. Which layer does the customer not administer: guest applications or the provider hypervisor?

#### Medium

1. Draw the stack for a Linux guest on EC2.
2. A student stops using a local VirtualBox VM and rents an Azure VM. What responsibility remains with the student?
3. Explain “elastic” using a website that is busy only during examination-result week.
4. Why can two tenants run on the same physical server?
5. Compare an image and a running instance.

#### Hard

1. A guest Linux kernel inside GCE panics. Explain the effect on that instance and why a neighboring tenant’s instance is a separate guest.
2. Explain why system calls inside an EC2 Linux guest are handled by the guest kernel, while control of the physical host is handled below the guest.
3. A project needs 20 machines for two hours. Compare buying 20 PCs with using a cloud VM service, in terms of setup and responsibility.
4. Why is calling EC2 “an operating system that replaces Linux” inaccurate?
5. Explain the role of the Type 1 hypervisor in EC2, Azure Virtual Machines, and GCE. The three providers do not share one product name.

### MCQs

**Q1. AWS EC2 provides:**

A. Virtual servers (instances) on Amazon’s infrastructure  
B. A replacement for the idea of a process inside every guest  
C. Only printed batch cards  
D. A microkernel used as the guest on every Android phone  

**Answer:** A  

**Explanation:** EC2 is Amazon’s elastic virtual-server service.

**Q2. GCE stands for:**

A. Google Compute Engine  
B. General Call Executor  
C. Guest CPU Emulator as a scheduling algorithm  
D. Global Card Editor  

**Answer:** A  

**Explanation:** GCE is Google’s infrastructure VM service.

**Q3. In a basic cloud VM, the guest operating system is managed by:**

A. The customer  
B. The hardware fan, with no software  
C. The end user’s printer driver only  
D. A punched-card operator inside the guest kernel  

**Answer:** A  

**Explanation:** The provider manages the host and hypervisor. The customer administers the guest OS.

**Q4. An image in EC2, Azure, or GCE is:**

A. A template used to boot a virtual machine  
B. A CPU scheduling quantum  
C. A Type 2 desktop game  
D. The mode bit  

**Answer:** A  

**Explanation:** Instances are created from OS images.

**Q5. Azure Virtual Machines are:**

A. Microsoft’s cloud VM service  
B. A file-allocation method  
C. A page-replacement algorithm  
D. A first-generation vacuum-tube OS  

**Answer:** A  

**Explanation:** Azure VMs provide virtual computers in Microsoft Azure.

**Q6. Tenant isolation on a shared cloud host depends primarily on:**

A. The hypervisor and provider network controls  
B. All tenants sharing one kernel heap with no separation  
C. Disabling virtualization  
D. Using the same user password for every tenant  

**Answer:** A  

**Explanation:** Each tenant’s guest is a separate VM managed by the hypervisor.

**Q7. “Elastic” compute means:**

A. The amount of VM capacity can be increased or decreased as needed  
B. The CPU clock is made of elastic material  
C. Files cannot be stored  
D. Only one instance may ever be created  

**Answer:** A  

**Explanation:** Elasticity is the ability to change capacity with demand.

**Q8. A system call made by an application inside a cloud Linux instance is serviced by:**

A. That instance’s guest Linux kernel  
B. The customer’s printer, acting as the kernel  
C. A different tenant’s guest, chosen at random  
D. Firmware that bypasses all operating systems by definition  

**Answer:** A  

**Explanation:** The guest OS handles its own processes and system calls. The hypervisor handles the virtual hardware underneath.

### Quick Revision

#### Key Definitions

- Cloud VM service: rented virtual computer on provider hardware.
- Instance: a running VM. Image: the OS template.
- EC2, Azure Virtual Machines, and GCE.

#### Important Concepts

- Guest OS is customer-managed. Hypervisor and physical host are provider-managed.
- These services sit on Type 1 virtualization in the data center.

#### Sequence to remember

- Request → schedule onto a host → hypervisor starts the VM → guest boots from the image.

#### Important Differences

- Image versus instance.
- Guest OS versus cloud management layer.

#### Important Exam Points

- EC2, Azure Virtual Machines, and GCE provide virtual machines. They are not CPU scheduling algorithms.
- The table of what the customer manages and what the provider manages.

---

