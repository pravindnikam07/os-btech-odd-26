**Navigation:** [Unit index](00_Index.md) · [Previous: 1.6 Cloud OS](06_Cloud_OS.md) · [Next: Unit 1 Summary](08_Unit_Summary.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 1 — OS Fundamentals and Architecture

## Topic 1.7 — Mobile Operating Systems

### Learning Outcomes

- Explain how a mobile operating system differs in role from a desktop OS while still performing OS functions.
- Describe the Android stack, especially the Linux kernel and the Hardware Abstraction Layer (HAL).
- Describe the iOS foundation, especially Darwin and the Core OS layer.
- Compare the Android stack with the iOS stack.

### Prerequisites

Kernel, system calls, and monolithic kernel structure (Topics 1.1, 1.3, and 1.4).

### Introduction

Phones and tablets run operating systems that manage the CPU, memory, storage, radio, sensors, and battery, and that provide services to apps. The two systems are:

- **Android**, built on the **Linux kernel**, with a **Hardware Abstraction Layer**.
- **iOS**, built on **Darwin**, with **Core OS** as the lowest software layer.

What matters here is the architecture: which kernel is used, and which layer sits next to it.

### Definition

**Definition:**  
A mobile operating system is an operating system designed for a handheld device. It manages device hardware and provides an environment for mobile applications, with strong attention to touch interaction, sensors, power, and a curated application model.

### Key Terminology

| Term | Meaning |
| --- | --- |
| Android | Mobile OS from Google, based on the Linux kernel |
| HAL | Hardware Abstraction Layer; a standard interface between Android software and device drivers |
| ART | Android Runtime; the managed runtime that executes app bytecode |
| Darwin | The open-source core of Apple’s operating systems, including the kernel and low-level services |
| XNU | The kernel inside Darwin; it combines Mach and BSD |
| Core OS | The lowest iOS layer, containing the kernel, file system, network stack, and basic security services |
| App | An application distributed to run on the mobile OS |

### Basic Concepts

A mobile OS still provides the services of Topic 1.3: it starts apps (processes), stores files, and controls devices. The device set is different from a desktop PC. It includes the cellular radio, GPS, camera, accelerometer, and a battery with a limited charge. The kernel or low-level layer must manage power, not only CPU time.

### Android

**Definition:**  
Android is a mobile operating system whose kernel is the Linux kernel. Above the kernel, Android adds the HAL, native libraries and the Android Runtime, the application framework, and apps.

**Explanation:**  
Linux supplies process management, memory management, a driver model, and security mechanisms. Phone hardware differs by manufacturer. The HAL defines a standard interface so that the rest of Android can use a camera, sensors, or audio without depending on one vendor’s driver details. Manufacturer-specific code sits behind that interface.

**How it works:**

1. The Linux kernel starts and initializes memory, processes, power management, and drivers.
2. The HAL loads the hardware module that matches the phone’s component.
3. Native libraries and ART provide graphics, database, and app-execution support.
4. The application framework offers services such as activity management and notifications to apps.
5. The user launches an app. The framework asks Linux, through the layers below, for processes, files, and devices.

```text
+----------------------------------+
|          Applications            |
+----------------------------------+
|     Application framework        |
+----------------------------------+
|  Native libraries  |     ART     |
+----------------------------------+
|   Hardware Abstraction Layer     |
+----------------------------------+
|          Linux kernel            |
|  (drivers, memory, processes,    |
|   power, binder IPC support)     |
+----------------------------------+
|             Hardware             |
+----------------------------------+
```

**Important points:**

- The kernel is Linux, so the kernel structure at the bottom is monolithic, as studied in Topic 1.4.
- HAL is not the kernel. It sits above the kernel and below the framework.
- Apps do not call arbitrary kernel internals. They use the framework. The framework’s path eventually reaches the kernel through system calls, as on any Linux system.
- ART executes Android apps. Older Android versions used Dalvik. Current Android uses ART.

**Example:**  
A camera app calls the camera API in the framework. The framework uses the camera HAL module. That module uses the kernel camera driver. The app never programs the camera registers.

### iOS

**Definition:**  
iOS is Apple’s mobile operating system. Its foundation is Darwin. The lowest layer of the iOS software stack is Core OS, which includes the kernel and other basic system services.

**Explanation:**  
Darwin contains the XNU kernel and low-level UNIX-derived services. XNU combines the Mach kernel with BSD services, so the kernel is hybrid, as described with operating-system structures. Core OS exposes low-level interfaces for files, threads, networking, and security. Higher layers provide application services, media, and the touch interface. Darwin and Core OS are the foundation.

**How it works:**

1. Firmware starts Darwin’s kernel (XNU).
2. Core OS initializes the file system, network stack, security services, and device support.
3. Core Services and higher layers start the services that apps use.
4. An app request that needs a file or the network passes down to Core OS.
5. XNU performs the privileged operation and returns the result upward.

```text
+----------------------------------+
|      Applications (UI layer)     |
+----------------------------------+
|      Media and app services      |
+----------------------------------+
|         Core Services            |
+----------------------------------+
|            Core OS               |
|   Darwin: XNU kernel, file       |
|   system, network, security      |
+----------------------------------+
|             Hardware             |
+----------------------------------+
```

The names of the upper layers have changed across iOS versions. The stable facts are these: **Darwin is the core, XNU is its kernel, and Core OS is the bottom layer.**

**Important points:**

- Darwin is not a separate phone that runs beside iOS. It is the foundation inside iOS.
- Core OS is a layer of the stack. It includes the kernel; it is not a second kernel next to Darwin.
- Privileged operations still occur in the kernel inside Core OS. Apps run above it with restricted rights.

**Example:**  
An app saves a photograph. The upper framework handles the user action. Core OS and its file-system component allocate storage and write the file through the kernel.

### Comparison

| Parameter | Android | iOS |
| --- | --- | --- |
| Kernel | Linux kernel | XNU, inside Darwin |
| Kernel classification | Monolithic Linux kernel | Hybrid (Mach + BSD) |
| Syllabus layer above or around the kernel | HAL above the Linux kernel | Core OS as the lowest iOS layer, containing Darwin’s services |
| Who adapts diverse phone hardware | HAL modules and vendor drivers | Apple controls the hardware and the OS together |
| App relationship to the kernel | Through the Android framework, libraries, HAL, then Linux | Through higher iOS layers, then Core OS |
| Typical device | Many manufacturers’ phones and tablets | Apple iPhone and iPad |

### Advantages and Limitations

| System | Advantages | Limitations |
| --- | --- | --- |
| Android (Linux + HAL) | Runs on many manufacturers’ devices; HAL gives a common hardware interface; Linux kernel is mature | Device differences still require vendor HAL modules and drivers |
| iOS (Darwin + Core OS) | Hardware and OS are designed together; a single Core OS foundation for Apple devices | Not available as a general OS for other manufacturers’ phones |

These points are architectural. They are not a ranking of products.

### Applications

- Android phones and tablets, and many dedicated devices that ship Android.
- iPhone and iPad devices running iOS.
- Both systems run the user’s apps as processes, store photos and messages as files, and control radios and sensors as devices. They are concrete cases of the services in Topic 1.3.

### Common Mistakes

- Writing that Android’s kernel is ART or the HAL. The kernel is Linux. ART is the runtime. HAL is the hardware interface layer.
- Writing that Darwin replaces Core OS. Core OS is the bottom layer and Darwin is the core technology inside that foundation.
- Drawing HAL below the Linux kernel. HAL is above the kernel and below native libraries and the framework.
- Treating a mobile OS as “not an operating system.” It is an OS with additional device and power constraints.

### Important Exam Points

- Android stack diagram with Linux kernel at the bottom and HAL directly above it.
- One sentence: HAL standardizes access to hardware so upper layers stay independent of the vendor.
- iOS: Darwin, XNU (Mach + BSD), Core OS at the bottom.
- Comparison of the kernel and of the named layer next to it: HAL on Android, Core OS on iOS.
- Both systems still use the privileged-kernel idea from Topic 1.1.

### University Exam Questions

#### 2-Mark Questions

1. Which kernel does Android use?
2. What is the Hardware Abstraction Layer in Android?
3. What is Darwin in the context of iOS?
4. What is the Core OS layer?
5. Where is HAL placed relative to the Linux kernel?

#### 4/5-Mark Questions

1. Explain the Android architecture with a diagram, emphasizing the Linux kernel and HAL.
2. Explain the iOS foundation using Darwin and Core OS.
3. Describe the path of a camera request from an Android app to the hardware.
4. Differentiate the kernels of Android and iOS.

#### 8/10-Mark Questions

1. Explain mobile operating systems with reference to Android (Linux kernel and HAL) and iOS (Darwin and Core OS). Draw both stacks.
2. Compare Android and iOS architecture. Show how each still provides process, file, and device services through a privileged kernel.

### Practice Problems

#### Easy

1. Name the kernel used by Android.
2. Is HAL above or below the Linux kernel?
3. Name the kernel inside Darwin.
4. Which iOS layer is the lowest: Core OS or a user app?
5. Expand HAL.

#### Medium

1. Draw the Android stack from the app down to hardware.
2. Explain why a phone vendor writes a HAL module.
3. Describe XNU in one sentence.
4. A student places Darwin beside iOS as a second OS. Correct the diagram.
5. Trace saving a file on iOS down to Core OS.

#### Hard

1. Relate Android’s Linux kernel to the monolithic structure studied earlier. Does HAL change that classification?
2. Relate XNU to the hybrid structure. Identify the two parts that justify the classification.
3. Compare an Android camera call and a Linux desktop read system call. Where do they become similar?
4. Explain power management as an OS duty on a phone, using the kernel’s role rather than an app’s role.
5. Why can the same Android framework run on two phones with different camera chips?

### MCQs

**Q1. Android uses which kernel?**

A. Linux kernel  
B. A batch-card monitor as the only kernel  
C. No kernel; apps program hardware directly  
D. The THE layered kernel as shipped by Dijkstra  

**Answer:** A  

**Explanation:** Android is built on the Linux kernel.

**Q2. In Android, HAL is:**

A. A layer above the Linux kernel that abstracts device-specific hardware  
B. A replacement CPU  
C. A cloud VM image format  
D. A page-replacement algorithm  

**Answer:** A  

**Explanation:** The Hardware Abstraction Layer gives upper Android layers a standard hardware interface.

**Q3. The correct Android order from lowest to highest is:**

A. Linux kernel, HAL, libraries and runtime, framework, apps  
B. Apps, Linux kernel, HAL, hardware, framework  
C. HAL, hardware, Linux kernel, apps  
D. Core OS, Darwin, EC2, HAL  

**Answer:** A  

**Explanation:** The kernel is at the bottom of the software stack. HAL sits immediately above it.

**Q4. Darwin is:**

A. The core of iOS, including the XNU kernel  
B. Google’s HAL implementation  
C. A Type 2 hypervisor used only for batch jobs  
D. A file-system allocation method  

**Answer:** A  

**Explanation:** Darwin is the foundation of iOS and includes XNU.

**Q5. XNU consists of:**

A. Mach and BSD components  
B. Linux and HAL only  
C. FIFO and LRU  
D. EC2 and GCE  

**Answer:** A  

**Explanation:** XNU is a hybrid of Mach and BSD.

**Q6. Core OS in iOS is:**

A. The lowest layer, providing kernel and basic system services  
B. A user-downloaded game  
C. The Android runtime  
D. A Type 1 hypervisor product from VirtualBox  

**Answer:** A  

**Explanation:** Core OS is the bottom layer of the iOS stack.

**Q7. An Android app normally obtains hardware services by:**

A. Using framework APIs, which pass through HAL to kernel drivers  
B. Editing Linux kernel memory from the app window  
C. Replacing the hypervisor called Dom0  
D. Issuing privileged I/O with no kernel  

**Answer:** A  

**Explanation:** Apps stay above the framework. HAL and the kernel perform the hardware-specific work.

**Q8. Which pairing is correct?**

A. Android — Linux kernel and HAL; iOS — Darwin and Core OS  
B. Android — Darwin; iOS — EC2  
C. Android — Xen Dom0 as the phone kernel; iOS — VirtualBox  
D. Both systems use only a Type 2 hypervisor and no local kernel  

**Answer:** A  

**Explanation:** Android is built on the Linux kernel with HAL above it. iOS is built on Darwin, and Core OS is the lowest layer.

### Quick Revision

#### Key Definitions

- Android kernel: Linux. HAL: hardware interface above that kernel.
- iOS foundation: Darwin. Kernel: XNU (Mach + BSD). Lowest layer: Core OS.

#### Important Concepts

- Mobile OSs still manage processes, memory, files, and devices.
- HAL isolates vendor hardware from the Android framework.
- Darwin is inside iOS, not a second OS beside it.

#### Paths to remember

- Android device use: app → framework → HAL → kernel driver → hardware.
- iOS low-level service: app → upper layers → Core OS / XNU → hardware.

#### Important Differences

- Linux monolithic kernel on Android versus hybrid XNU on iOS.
- HAL (Android) versus Core OS (iOS) are not equivalent names. HAL is a hardware-abstraction layer above Linux. Core OS is the bottom iOS layer that contains the kernel.

#### Important Exam Points

- Two stack diagrams.
- ART is the Android runtime. The kernel under Android is Linux.
- XNU is hybrid because it combines Mach and BSD.

---

