**Navigation:** [Unit index](00_Index.md) · [Previous: 3.11 Banker’s algorithm](11_Bankers_Algorithm.md) · [Next: Unit summary](13_Unit_Summary.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 3 — Process Synchronization and Deadlocks

## Topic 3.12 — Deadlock Detection and Recovery

### Learning Outcomes

- Build a wait-for graph and read a cycle as deadlock.
- State when a cycle is not enough, because a resource type has several instances.
- Recover by process termination or by resource preemption.
- Explain how a victim can starve if it is always the process rolled back.

### Prerequisites

Resource-allocation graphs and the four conditions (Topic 3.9). The idea of a safe-state search (Topic 3.11) helps with the multiple-instance detector, which reuses the same table walk.

### Introduction

Detection allows deadlock. Periodically, or when processes have been waiting too long, the system asks whether a deadlock is already present. If each resource type has one instance, the question is a cycle in a wait-for graph. If a type has several instances, the system runs a matrix algorithm in the style of the safety test. Recovery then kills processes or takes resources away until the cycle breaks. Detection is attractive when deadlocks are rare and the cost of prevention or avoidance would be paid on every request.

### Definition

**Definition:**  
Deadlock detection is an algorithm that decides whether a set of processes is presently deadlocked.

**Definition:**  
Recovery is the action that breaks a detected deadlock, either by terminating one or more processes or by preempting resources from a victim.

### Wait-for graph

Start from the resource-allocation graph. Remove the resource nodes. An edge Pi → Pk remains when Pi is waiting for a resource that Pk holds. The new graph has only process nodes.

```text
Resource-allocation graph          Wait-for graph

X → A → Y → B → X                  A → B
                                   ^   |
                                   +---+

A waits for Y, and B holds Y, so A → B.
B waits for X, and A holds X, so B → A.
```

```text
Algorithm: single-instance detection

Step 1: For every process that is waiting, draw an edge to the process
        that holds the resource being requested.
Step 2: Search for a cycle.
Step 3: A cycle means the processes on it are deadlocked.
Step 4: No cycle means there is no deadlock under the single-instance rule.
```

The five philosophers, each waiting for the neighbour who holds the other chopstick, are a cycle of length 5 in the wait-for graph.

A cycle is decisive only when each resource has one instance. If the printer type has two instances, “A waits for a printer held by B” is incomplete: a free instance, or an instance held by a process that is not in the cycle, may satisfy A. Multiple instances use the algorithm below.

### Several instances of a resource type

The matrices are the ones used by the banker, with one change. There is no Max. Processes have already stated a current Request, which may be smaller than any old maximum. Available, Allocation, and Request are the inputs.

```text
Algorithm: detection with multiple instances

Step 1: Work = Available. Finish[i] = false.
        If Allocation[i] is all zeros, Finish[i] = true,
        because that process holds nothing that others need.
Step 2: Find an i with Finish[i] = false and Request[i] ≤ Work.
Step 3: If no such i exists, go to Step 5.
Step 4: Work = Work + Allocation[i]. Finish[i] = true. Go to Step 2.
Step 5: Every false Finish[i] is a deadlocked process.
        If every Finish[i] is true, there is no deadlock.
```

The walk assumes that a process whose current request fits in Work can finish and return its allocation. A process that cannot be given this chance, and that is not already finished, is in the deadlocked set. This is a detection of the present request, not a promise about a future maximum. That is why it is not the Banker’s algorithm, even though the arithmetic looks similar.

**Example.**  
Available = (0, 0). Two processes.

| Process | Allocation | Request |
| --- | --- | --- |
| P0 | 1 0 | 0 1 |
| P1 | 0 1 | 1 0 |

Work starts at (0, 0). P0 wants (0, 1), which does not fit. P1 wants (1, 0), which does not fit. Both Finish flags stay false. Both processes are deadlocked. Each holds the unit the other is requesting.

If Available were (1, 0), P1’s request would fit. Work would become (1, 0) + (0, 1) = (1, 1). P0’s request (0, 1) would then fit. Neither process would be reported deadlocked.

### When to run the detector

| Policy | Cost |
| --- | --- |
| On every request | Deadlock is noticed immediately. The graph or the matrices are examined very often |
| Every few minutes, or when CPU utilization drops | Cheaper. A deadlock can exist until the next examination |
| When a process has waited longer than a threshold | The search is aimed at the symptom |

A detector that never runs is not a strategy. The system would leave the deadlocked processes waiting.

### Recovery by termination

| Choice | What happens | Cost |
| --- | --- | --- |
| Abort every process in the deadlocked set | The cycle certainly breaks | Every process in the set loses its work |
| Abort one process at a time | After each abort, run detection again, and stop when the cycle is gone | Fewer processes may die, but detection runs repeatedly |

The process chosen for abortion is the victim. Typical factors are the priority, how much CPU time would be lost, how many resources it holds, and how many processes would be freed. There is no single formula. The answer in an examination is the list of factors and the fact that detection is repeated if victims are removed one by one.

### Recovery by preemption

1. Select a victim process and a resource to take from it.
2. Roll the victim back to a safe saved state, or restart it, because the resource was taken in the middle of its work.
3. Give the resource to one of the waiting processes.
4. Run detection again if the cycle may still exist.

Rollback needs checkpoints. Without a saved state, preemption is only a disguised abort. A printer buffer can sometimes be rewound. A mutex in an arbitrary kernel section often cannot.

**Starvation of the victim.**  
If the same process is the cheapest victim on every detection, it is rolled back forever. The recovery policy must count how many times a process has been chosen and eventually pick someone else. That count is the same idea as aging.

### Comparison

| Method | Moment | Question it asks | Deadlock can exist for a while? |
| --- | --- | --- | --- |
| Prevention | Before any request pattern forms | Can I deny one necessary condition? | No, if the denial holds |
| Avoidance | At each request | Would the next state still be safe? | No, if maxima are honest and unsafe requests wait |
| Detection | After allocation | Are processes deadlocked now? | Yes, until the detector and the recovery run |

### Advantages and limitations

| Advantages | Limitations |
| --- | --- |
| Processes are not forced to declare a maximum | Deadlock is real until detection runs |
| There is no order of all resource types to maintain | Recovery destroys work or needs checkpoints |
| The wait-for graph is small and clear for single-instance resources | A cycle is the wrong final test when a type has several instances |
| Rare deadlocks do not pay the avoidance test on every request | A bad victim policy starves the victim |

### Common mistakes

- Drawing a wait-for edge from a process to a resource. In the wait-for graph both ends are processes.
- Declaring deadlock from a cycle when two instances of that resource exist. Use the matrix algorithm.
- Confusing Request in detection with Need in the Banker’s algorithm. Need is a future maximum remainder. Request is what the process is waiting for now.
- Aborting a victim and forgetting to run detection again when other processes may still be in a cycle.
- Always choosing the same victim.

### Important exam points

- Wait-for edge: waiter → holder. A cycle, for one instance of each resource, is deadlock.
- Multiple instances: Work and Request, then Work = Work + Allocation. Unfinished processes are deadlocked.
- The two-process example with Available (0, 0) is a deadlock. Available (1, 0) is not.
- Recovery: abort all, or abort one at a time, or preempt and roll back.
- Count victim selections so the same process is not rolled back forever.

### University exam questions

#### 2-mark questions

1. What is a wait-for graph?
2. What does a cycle mean if every resource has one instance?
3. Name two recovery methods.
4. Why must a victim not be selected on every recovery?

#### 4/5-mark questions

1. Convert a two-process resource-allocation cycle into a wait-for graph.
2. Explain the multiple-instance detection algorithm.
3. Compare termination and resource preemption.

#### 8/10-mark questions

1. Explain deadlock detection for single-instance and multiple-instance resources. Use a cycle and the two-process matrix example.
2. Explain recovery. Include victim selection, rollback, and starvation of the victim. Compare detection with avoidance.

### Practice problems

#### Easy

1. In a wait-for graph, what does A → B mean?
2. A wait-for graph of three processes has no cycle. Are they deadlocked under the single-instance rule?
3. Name a checkpoint’s role in preemption.
4. After aborting one process, why is detection run again?
5. Which matrix replaces Need when the algorithm is detection rather than avoidance?

#### Medium

1. A holds X and waits for Y. B holds Y and waits for X. Draw the wait-for edges.
2. Run the multiple-instance example with Available (0, 0). Which processes are deadlocked?
3. Change Available to (1, 0) and repeat.
4. List three reasonable victim criteria.
5. Explain victim starvation in two sentences.

#### Hard

1. Philosophers 0 through 4 each wait for the next philosopher. How long is the wait-for cycle?
2. Available is (0, 1), P0 Allocation (1, 0) Request (0, 1), P1 Allocation (0, 0) Request (0, 0). Who is deadlocked?
3. Why is “Allocation all zeros ⇒ Finish true” safe in detection?
4. A resource cannot be rolled back. Which recovery method remains?
5. Compare the Banker’s Need test with detection’s Request test in one sentence each.

**Answers**

- A → B means A is waiting for a resource that B holds.
- No cycle means no deadlock when each resource has one instance.
- The two-process wait-for edges are A → B and B → A.
- Available (0, 0): both P0 and P1 are deadlocked. Available (1, 0): P1’s request fits, then P0’s request fits. Nobody is deadlocked.
- Five philosophers form a cycle of length 5.
- P1 holds nothing, so Finish[P1] starts true. Work is (0, 1). P0’s request (0, 1) fits. Work becomes (1, 1) and P0 finishes. Nobody is deadlocked. P1 was not waiting.
- A process that holds nothing cannot be on a “holds and waits” cycle. Marking it finished lets the algorithm ignore it.
- If the resource cannot be taken back and restored, recovery has to terminate a process.
- The banker asks whether the remaining maximum could be granted. Detection asks whether the request the process is waiting for now can be granted.

### MCQs

**Q1. A wait-for graph contains:**

A. Only processes, with an edge from the waiter to the holder  
B. Only resources  
C. Request edges from a resource to a process  
D. The CPU ready queue  

**Answer:** A  

**Explanation:** Resource nodes are removed. Pi → Pk means Pi waits for something Pk holds.

**Q2. With one instance of each resource, deadlock exists if:**

A. The wait-for graph has a cycle  
B. The wait-for graph has at least two nodes and no edges  
C. Available is large  
D. A process is in the ready state  

**Answer:** A  

**Explanation:** The cycle is necessary and sufficient in the single-instance case.

**Q3. With several instances of a resource type, a cycle in a resource graph:**

A. Does not by itself prove deadlock  
B. Always proves deadlock  
C. Proves the state is safe  
D. Replaces the Request matrix  

**Answer:** A  

**Explanation:** A free instance, or an instance held outside the apparent cycle, may satisfy the request. The matrix algorithm is the test.

**Q4. In multiple-instance detection, a process that never receives Work is:**

A. Counted as deadlocked if its Finish flag stays false  
B. Always safe  
C. Removed from the system before the algorithm starts  
D. The banker  

**Answer:** A  

**Explanation:** Step 5 reports every process whose Finish flag remains false.

**Q5. Available (0, 0), P0 holding (1, 0) and requesting (0, 1), P1 holding (0, 1) and requesting (1, 0) means:**

A. Both processes are deadlocked  
B. Only P0 is deadlocked  
C. The state is safe in the Banker’s sense and both can finish now  
D. Neither process is waiting  

**Answer:** A  

**Explanation:** Neither request fits Work, so both Finish flags stay false.

**Q6. Aborting one victim at a time requires:**

A. Detection again after each abort  
B. Aborting every process in the system, including those not in the cycle  
C. A declared Max from every process  
D. That the victim keep its resources  

**Answer:** A  

**Explanation:** The first abort may leave another cycle. The detector checks.

**Q7. Resource preemption needs:**

A. A way to roll the victim back to a saved state  
B. The victim to continue from the middle of the lost resource with no repair  
C. A wait-for graph with no processes  
D. Prevention of mutual exclusion  

**Answer:** A  

**Explanation:** The work done with the resource is incomplete. A checkpoint, or a restart, is required.

**Q8. Selecting the same victim on every recovery causes:**

A. Starvation of that victim  
B. A guarantee that the victim finishes  
C. Prevention of circular wait  
D. Conversion of the system into a monitor  

**Answer:** A  

**Explanation:** The victim is rolled back again each time. A count of rollbacks must change the choice.

### Quick revision

#### Single instance

Wait-for edge: waiter → holder. A cycle is deadlock.

#### Several instances

```text
Work = Available
If Request[i] ≤ Work, then Work = Work + Allocation[i]
Anyone left unfinished is deadlocked
```

Example: each of two processes holds the unit the other wants, and Available is (0, 0). Both are deadlocked.

#### Recovery

Abort all processes in the set, or abort one and detect again, or preempt a resource and roll the victim back. Do not choose the same victim every time.
