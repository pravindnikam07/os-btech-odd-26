**Navigation:** [Unit index](00_Index.md) · [Previous: 3.10 Deadlock prevention](10_Deadlock_Prevention.md) · [Next: 3.12 Deadlock detection and recovery](12_Deadlock_Detection_and_Recovery.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 3 — Process Synchronization and Deadlocks

## Topic 3.11 — Deadlock Avoidance: The Banker’s Algorithm

### Learning Outcomes

- Define a safe state and a safe sequence.
- Compute the Need matrix.
- Run the safety algorithm and produce a safe sequence.
- Grant or refuse a resource request.

### Prerequisites

Deadlock, and the difference between prevention and avoidance (Topics 3.9 and 3.10).

### Introduction

Avoidance lets a process hold some resources and ask for more. Before the request is granted, the system asks whether every process could still finish if each one later demanded its declared maximum. If such an order exists, the state is safe and the request may be granted. If no order exists, the state is unsafe and the process waits, even when the units it wants are free. The Banker’s algorithm is that test. The name is a banker who will not hand over cash unless, after the loan, every customer can still be given that customer’s declared maximum and can later repay.

### Definition

**Definition:**  
A state is safe when there is a safe sequence of all the processes. In the sequence ⟨P1, P2, …, Pn⟩, each Pi can obtain its remaining need from the resources that are free plus the resources held by P1 through P(i−1).

**Definition:**  
A state is unsafe when no safe sequence exists. An unsafe state is not a deadlock. It is a state in which a deadlock can follow if processes request up to their maxima.

**Definition:**  
The Banker’s algorithm grants a request only when the state after the grant is safe.

Each process declares Max in advance. If a process later needs more than Max, the assumption is false and the guarantee disappears.

### The matrices

Columns are resource types A, B, and C.

| Matrix | Meaning |
| --- | --- |
| Allocation | Units each process holds now |
| Max | The most each process is allowed to hold |
| Need | Max − Allocation |
| Available | Units held by nobody |

```text
Need[i][j] = Max[i][j] − Allocation[i][j]
```

### The safety algorithm

```text
Algorithm: Safety

Step 1: Work = Available. Finish[i] = false for every process.
Step 2: Find an i with Finish[i] = false and Need[i] ≤ Work
        in every resource type.
Step 3: If no such i exists, go to Step 5.
Step 4: Work = Work + Allocation[i]. Finish[i] = true.
        Append i to the sequence. Go to Step 2.
Step 5: The state is safe if every Finish[i] is true.
        Otherwise it is unsafe.
```

Adding Allocation to Work means “suppose this process finishes and returns what it holds.” The algorithm does not force the process to finish. It checks that an order exists.

### The request algorithm

```text
Algorithm: Resource request by Pi

Step 1: If Request ≤ Need[i] is false, Pi exceeded its claim. Reject it.
Step 2: If Request ≤ Available is false, the units are not free. Pi waits.
Step 3: Pretend to grant it:
            Available = Available − Request
            Allocation[i] = Allocation[i] + Request
            Need[i] = Need[i] − Request
Step 4: Run the safety algorithm.
        If the result is safe, keep the grant.
        If it is unsafe, restore the three matrices and make Pi wait.
```

### Worked example — the initial state is safe

| Process | Allocation | Max | Need |
| --- | --- | --- | --- |
| P0 | 0 1 0 | 7 5 3 | 7 4 3 |
| P1 | 2 0 0 | 3 2 2 | 1 2 2 |
| P2 | 3 0 2 | 9 0 2 | 6 0 0 |
| P3 | 2 1 1 | 2 2 2 | 0 1 1 |
| P4 | 0 0 2 | 4 3 3 | 4 3 1 |

Available = (3, 3, 2).

P0’s Need is (7 − 0, 5 − 1, 3 − 0) = (7, 4, 3). The other rows are the same subtraction.

| Step | Process | Comparison | Work after it returns Allocation |
| --- | --- | --- | --- |
| 1 | P1 | (1, 2, 2) ≤ (3, 3, 2) | (3, 3, 2) + (2, 0, 0) = (5, 3, 2) |
| 2 | P3 | (0, 1, 1) ≤ (5, 3, 2) | (5, 3, 2) + (2, 1, 1) = (7, 4, 3) |
| 3 | P4 | (4, 3, 1) ≤ (7, 4, 3) | (7, 4, 3) + (0, 0, 2) = (7, 4, 5) |
| 4 | P0 | (7, 4, 3) ≤ (7, 4, 5) | (7, 4, 5) + (0, 1, 0) = (7, 5, 5) |
| 5 | P2 | (6, 0, 0) ≤ (7, 5, 5) | (7, 5, 5) + (3, 0, 2) = (10, 5, 7) |

Every Finish flag is true. One safe sequence is ⟨P1, P3, P4, P0, P2⟩.

P0 cannot be first: 7 > 3. P2 cannot be first: 6 > 3.

### Worked example — P1 is granted (1, 0, 2)

1. P1’s Need is (1, 2, 2), and (1, 0, 2) ≤ (1, 2, 2).
2. Available is (3, 3, 2), and (1, 0, 2) ≤ (3, 3, 2).
3. Pretend grant:
   - Available = (2, 3, 0)
   - P1 Allocation = (3, 0, 2)
   - P1 Need = (0, 2, 0)
4. Safety from Work = (2, 3, 0):
   - P1: (0, 2, 0) ≤ (2, 3, 0). Work = (5, 3, 2).
   - P3: (0, 1, 1) ≤ (5, 3, 2). Work = (7, 4, 3).
   - P4: (4, 3, 1) ≤ (7, 4, 3). Work = (7, 4, 5).
   - P0: (7, 4, 3) ≤ (7, 4, 5). Work = (7, 5, 5).
   - P2: (6, 0, 0) ≤ (7, 5, 5). Work = (10, 5, 7).

The state is safe, so the grant is kept.

### Worked example — P0 is refused (0, 2, 0)

This request is tested in the state after P1’s grant, where Available is (2, 3, 0).

1. P0’s Need is (7, 4, 3), and (0, 2, 0) ≤ (7, 4, 3).
2. (0, 2, 0) ≤ (2, 3, 0). The units are free.
3. Pretend state: Available (2, 1, 0), P0 Allocation (0, 3, 0), P0 Need (7, 2, 3).
4. No Need fits (2, 1, 0):

| Process | Need | Reason it does not fit |
| --- | --- | --- |
| P0 | (7, 2, 3) | 7 > 2 |
| P1 | (0, 2, 0) | 2 > 1 |
| P2 | (6, 0, 0) | 6 > 2 |
| P3 | (0, 1, 1) | 1 > 0 |
| P4 | (4, 3, 1) | 4 > 2 |

The pretend state is unsafe. The matrices are restored and P0 waits. Free units are not a sufficient reason to say yes.

### Safe, unsafe, and deadlocked

| State | Meaning |
| --- | --- |
| Safe | Some order can satisfy every remaining maximum and finish every process |
| Unsafe | No such order exists. Deadlock is possible. It is not certain |
| Deadlock | Processes are already waiting for resources that this set will not release |

### Advantages and limitations

| Advantages | Limitations |
| --- | --- |
| A grant leaves a safe sequence | Max must be declared in advance |
| “Free” and “safe” are separate decisions | The test runs on every request |
| Deadlock is avoided rather than cleaned up | Only a modest number of resource types is practical |
| The arithmetic is a finite table | A false Max, or a process that does not return resources, removes the guarantee |

### Common mistakes

- Comparing Max with Work. The comparison uses Need.
- Adding Allocation to Work before the Need test.
- Forgetting one of the three updates: Available, Allocation, and Need.
- Keeping an unsafe pretend grant.
- Calling the method prevention. The four conditions are still possible; the unsafe request is what is refused.
- Insisting on only one sequence when another order also finishes every process.

### Important exam points

- Need = Max − Allocation.
- Initial Available (3, 3, 2). Safe sequence ⟨P1, P3, P4, P0, P2⟩.
- P1 request (1, 0, 2) is granted. Available becomes (2, 3, 0).
- P0 request (0, 2, 0) after that grant is refused. Pretend Available (2, 1, 0) fits nobody.
- Unsafe does not mean deadlocked.

### University exam questions

#### 2-mark questions

1. Define a safe state.
2. Write the formula for Need.
3. What is done with an unsafe pretend state?
4. Is an unsafe state already a deadlock?

#### 4/5-mark questions

1. State the safety algorithm.
2. From the standard snapshot, show why P1 can be first and why P0 cannot.
3. Differentiate safe, unsafe, and deadlocked.

#### 8/10-mark questions

1. Compute Need for the standard snapshot and produce a safe sequence. Show Work after each process.
2. Process P1’s request (1, 0, 2) and then P0’s request (0, 2, 0). Grant or refuse each request.

### Practice problems

#### Easy

1. Max is (5, 3, 3) and Allocation is (1, 1, 0). What is Need?
2. Available is (2, 1, 0) and Need is (2, 2, 0). May this process be selected?
3. Work is (2, 2, 2) and the selected Allocation is (1, 0, 1). What is the next Work?
4. If Request is greater than Need, what is the decision?
5. Is one safe sequence enough?

#### Medium

1. List Work after each process in ⟨P1, P3, P4, P0, P2⟩.
2. Why can the initial sequence not start with P2?
3. After P1 is granted (1, 0, 2), what are Available and P1’s Need?
4. Which single component stops P1 in the unsafe pretend state (2, 1, 0)?
5. Why can the banker refuse a request that fits in Available?

#### Hard

1. From the initial state, P3 requests (0, 1, 1). Is the result safe? Give a sequence.
2. From the initial state, P4 requests (3, 3, 0). Step 2 succeeds. What does the safety test say?
3. Available is (0, 0, 0) and every unfinished process has a positive Need. What is the result?
4. Give a second safe sequence for the initial state, starting with P3.
5. The table was declared safe, then a process requests more than Max. Why is the old sequence no longer a guarantee?

**Answers**

- Need = (4, 2, 3).
- (2, 2, 0) is not ≤ (2, 1, 0).
- Next Work = (3, 2, 3).
- A request above Need is rejected as an illegal claim.
- One safe sequence is enough.
- Initial Work values: (5, 3, 2), (7, 4, 3), (7, 4, 5), (7, 5, 5), (10, 5, 7).
- P2’s Need (6, 0, 0) exceeds Available A, which is 3.
- After the grant, Available is (2, 3, 0) and P1’s Need is (0, 2, 0).
- P1 needs 2 units of B and only 1 is in the pretend Available.
- Fitting in Available is only Step 2. Step 4 can still find the result unsafe.
- P3’s (0, 1, 1) leaves Available (3, 2, 1), P3’s Need (0, 0, 0), and P3’s Allocation (2, 2, 2). The state is safe. One sequence is ⟨P3, P1, P4, P0, P2⟩: Work moves through (5, 4, 3), (7, 4, 3), (7, 4, 5), (7, 5, 5), and (10, 5, 7).
- P4’s (3, 3, 0) is within Available (3, 3, 2), but the pretend state is unsafe, so P4 waits.
- Nobody can be selected, so the state is unsafe.
- ⟨P3, P1, P4, P0, P2⟩ is safe from the initial state: P3’s Need (0, 1, 1) ≤ (3, 3, 2), and Work then becomes (5, 4, 3).
- The algorithm assumed Max was a true ceiling. A larger demand can require a resource the safe sequence never reserved.

### MCQs

**Q1. Need equals:**

A. Max − Allocation  
B. Allocation − Available  
C. Max + Allocation  
D. Available only  

**Answer:** A  

**Explanation:** Need is what the process may still request, not what it already holds.

**Q2. A state is safe when:**

A. There is an order in which every process can obtain its remaining Need and finish  
B. Available is the zero vector  
C. A deadlock has already been detected  
D. Every process is running in its critical section  

**Answer:** A  

**Explanation:** That order is a safe sequence.

**Q3. In the standard snapshot, a safe sequence is:**

A. ⟨P1, P3, P4, P0, P2⟩  
B. ⟨P0, P1, P2, P3, P4⟩  
C. ⟨P2, P0, P1, P3, P4⟩  
D. The empty sequence  

**Answer:** A  

**Explanation:** P0 and P2 need more of resource A than the initial Available provides, so they cannot be first. The listed order passes every Need test.

**Q4. The safety algorithm adds Allocation[i] to Work:**

A. After Need[i] has been found less than or equal to Work  
B. Before looking at Need  
C. Only when the state is unsafe  
D. Instead of computing Need  

**Answer:** A  

**Explanation:** The addition represents the process finishing and returning its resources.

**Q5. P1’s request (1, 0, 2) from Available (3, 3, 2) is:**

A. Granted, because the resulting state is safe  
B. Refused, because it exceeds Need  
C. Refused, because the units are not free  
D. Granted without a safety test  

**Answer:** A  

**Explanation:** The request is within Need and Available, and ⟨P1, P3, P4, P0, P2⟩ remains safe. Available becomes (2, 3, 0).

**Q6. P0’s request (0, 2, 0), after that grant, is refused because:**

A. The pretend Available (2, 1, 0) satisfies no process’s Need  
B. (0, 2, 0) exceeds P0’s Max  
C. The units are not free  
D. P0 is not a process  

**Answer:** A  

**Explanation:** Step 2 succeeds. The safety test fails, so the pretend updates are discarded.

**Q7. An unsafe state:**

A. May lead to deadlock, and is not itself a deadlock  
B. Is always a deadlock  
C. Is always safe  
D. Means Available is negative  

**Answer:** A  

**Explanation:** Deadlock is present only when the waits already cannot be broken. Unsafe means no safe sequence exists.

**Q8. The Banker’s algorithm is avoidance because it:**

A. Refuses an unsafe request instead of making a deadlock condition impossible  
B. Numbers every mutex and denies circular wait  
C. Removes mutual exclusion  
D. Detects a cycle and kills a process  

**Answer:** A  

**Explanation:** Prevention changes the rules so a necessary condition cannot occur. The banker tests the next state.

### Quick revision

#### Formulas and steps

```text
Need = Max − Allocation
Work starts as Available
If Need[i] ≤ Work, then Work = Work + Allocation[i]
Grant a request only when the pretend state is safe
```

#### Standard numbers

- Available (3, 3, 2). Sequence ⟨P1, P3, P4, P0, P2⟩.
- Grant P1 (1, 0, 2). Available becomes (2, 3, 0).
- Refuse the later P0 (0, 2, 0). Pretend Available (2, 1, 0) fits no Need.

#### Words

Safe: a finishing order exists. Unsafe: no finishing order, deadlock not yet certain. Deadlock: the waits are already closed.
