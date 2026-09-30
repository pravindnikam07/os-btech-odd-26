**Navigation:** [Unit index](00_Index.md) · [Previous: 2.9 Multicore scheduling](09_Multicore_Scheduling.md) · [Next: 2.11 Linux CFS](11_Linux_CFS.md)

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 2 — Processes, Threads and CPU Scheduling

## Topic 2.10 — Real-Time Scheduling: Rate Monotonic and EDF

### Learning Outcomes

- State what a periodic real-time task is.
- Apply Rate Monotonic priority assignment and its utilization test.
- Apply Earliest Deadline First.
- Use one task set to show a Rate Monotonic deadline miss that EDF avoids.

### Prerequisites

Preemptive priority scheduling (Topic 2.7) and the distinction between hard and soft real-time systems (Unit 1).

### Introduction

A real-time task is correct only if its result arrives by a deadline. Many such tasks are periodic: a sensor is read every T time units and the computation must finish before the next reading, or before a stated deadline. Two classic preemptive algorithms are Rate Monotonic (RM) and Earliest Deadline First (EDF). RM fixes the priorities before execution. EDF changes the effective priority as deadlines approach.

The tests below are for periodic tasks whose relative deadline equals the period. A job released at time r must finish by time r + T.

### Definition

**Definition:**  
Rate Monotonic scheduling assigns a fixed priority by period: the shorter the period, the higher the priority.

**Definition:**  
Earliest Deadline First scheduling always runs the ready job whose absolute deadline is nearest.

### Task quantities

| Symbol | Meaning |
| --- | --- |
| Cᵢ | Worst-case execution time of one job of task i |
| Tᵢ | Period of task i |
| Uᵢ = Cᵢ / Tᵢ | Processor utilization of task i |
| U = Σ Cᵢ / Tᵢ | Total utilization |
| Release time | Time at which a job becomes ready |
| Absolute deadline | Release time plus the relative deadline. Here the relative deadline is Tᵢ |

A job that has not finished at its absolute deadline has missed the deadline.

### Rate Monotonic

```text
Tasks sorted by period
        ↓
Shorter period → higher fixed priority
        ↓
At every release and completion, run the ready job
with the highest RM priority
        ↓
A job may preempt a longer-period job
```

```text
Algorithm: Rate Monotonic

Step 1: Order tasks so that T1 ≤ T2 ≤ … ≤ Tn.
Step 2: Give task 1 the highest priority and task n the lowest. These priorities never change.
Step 3: When several jobs are ready, run the one whose task has the shorter period.
Step 4: A newly released job preempts the running job if its period is shorter.
Step 5: A schedule is acceptable only if every job completes by its deadline.
```

**Utilization test.**  
For n periodic tasks with deadline equal to period, a sufficient schedulability test is

```text
U = C1/T1 + C2/T2 + … + Cn/Tn  ≤  n (2^(1/n) − 1)
```

| n | Bound n (2^(1/n) − 1) |
| ---: | ---: |
| 1 | 1.000 |
| 2 | 0.828 |
| 3 | 0.780 |
| Large n | approaches ln 2 ≈ 0.693 |

The bound for two tasks is computed as follows. 2^(1/2) = √2 ≈ 1.414. Then 1.414 − 1 = 0.414, and 2 × 0.414 = 0.828.

If the inequality holds, RM will meet every deadline. If the utilization is above the bound, the test does not decide. The task set may still be schedulable, and it must be checked by drawing the schedule or by an exact test. If utilization is above 1, no algorithm can schedule the tasks on one processor, because the jobs demand more CPU time than the processor has.

Among fixed-priority rules for these periodic tasks, RM is optimal: if any fixed-priority assignment can meet the deadlines, the rate-monotonic assignment can too.

### Example — RM meets the deadlines

| Task | C | T | C/T |
| --- | ---: | ---: | ---: |
| T1 | 20 | 50 | 0.40 |
| T2 | 35 | 100 | 0.35 |

U = 0.75. The two-task bound is 0.828. Because 0.75 ≤ 0.828, RM is guaranteed to succeed.

T1 has the shorter period, so T1 has the higher priority.

```text
|   T1   |        T2         |   T1   | T2 |
0        20                  50       70   75
```

- The first T1 job is released at 0, runs 0–20, and meets deadline 50.
- T2 runs 20–50 and is preempted by the next T1 job, with 5 time units still left.
- T1 runs 50–70 and meets deadline 100.
- T2 finishes the remaining 5 units during 70–75 and meets deadline 100.

### Example — utilization above the RM bound

| Task | C | T | C/T |
| --- | ---: | ---: | ---: |
| A | 2 | 5 | 0.400 |
| B | 4 | 7 | 0.571 |

U = 2/5 + 4/7 = 14/35 + 20/35 = 34/35 ≈ 0.971.

This is less than or equal to 1, so a one-processor schedule is not impossible. It is greater than 0.828, so the RM test does not guarantee success. The schedule shows an actual miss.

A has the shorter period and the higher RM priority.

```text
| A |   B   | A |
0   2       5   7
```

- A runs 0–2 and meets deadline 5.
- B runs 2–5 and then the next A job, released at 5, preempts it. B has used 3 of its 4 units.
- A runs 5–7 and meets deadline 10.
- At time 7, B’s deadline is 7 and 1 unit of B is still unfinished. B misses its deadline.

### Earliest Deadline First

```text
Each ready job has an absolute deadline
        ↓
Run the job with the nearest deadline
        ↓
A newly released job preempts the running job
if its deadline is earlier
        ↓
Priorities change as time passes and new jobs are released
```

```text
Algorithm: EDF

Step 1: When a job is released at time r, set its absolute deadline to r + T.
Step 2: Among ready jobs, run the one with the smallest absolute deadline.
Step 3: If a new job’s deadline is earlier than the running job’s deadline, preempt.
Step 4: If two deadlines are equal, either job may run. Both choices are valid EDF decisions.
Step 5: Every job must finish by its absolute deadline.
```

**Utilization test.**  
For periodic tasks on one processor, with relative deadline equal to period, EDF meets every deadline if and only if

```text
U = Σ (Ci / Ti)  ≤  1
```

EDF can therefore accept task sets that lie above the Rate Monotonic bound, up to full utilization. The cost is that the priority of a job is not fixed: the scheduler must compare deadlines at run time.

### Example — EDF on the task set that RM missed

The same tasks are A (C = 2, T = 5) and B (C = 4, T = 7). U ≈ 0.971 ≤ 1, so EDF can meet the deadlines.

At time 0, A’s deadline is 5 and B’s deadline is 7. A runs first.

```text
| A |    B    | A |
0   2         6 8
```

- A runs 0–2 and meets deadline 5.
- B’s deadline 7 is earlier than the deadline 10 of the A job released at time 5, so B continues through time 5 and finishes at time 6. The deadline at 7 is met.
- A then runs 6–8 and meets deadline 10.

The first B job, which RM had not finished at time 7, finishes at time 6 under EDF.

### Comparison

| Parameter | Rate Monotonic | Earliest Deadline First |
| --- | --- | --- |
| Priority | Fixed, from the period | Dynamic, from the absolute deadline |
| Shorter period | Higher priority | Helpful only because it creates earlier deadlines |
| Test, deadline = period | U ≤ n(2^(1/n) − 1) is sufficient | U ≤ 1 is necessary and sufficient on one processor |
| Utilization that can be guaranteed | At most about 0.693 for a large number of tasks | Up to 1 |
| A set with U = 0.971 and two tasks | The bound does not guarantee it; the example misses | Schedulable |
| Run-time work | Priorities are assigned once | Deadlines are compared as jobs are released |
| Preemption | A shorter-period job preempts a longer-period job | An earlier deadline preempts a later deadline |

### Advantages and limitations

| Algorithm | Advantages | Limitations |
| --- | --- | --- |
| RM | Simple fixed priorities; a clear sufficient test; optimal among fixed-priority rules | The sufficient bound is below 1, so some feasible sets fail the test or miss deadlines |
| EDF | Uses the processor up to utilization 1 for deadline = period; meets deadlines on sets RM can miss | Deadline comparisons are more work; behaviour under a brief overload is less predictable than a fixed-priority scheme |

Neither algorithm removes the need for the execution times and periods to be known. If the real execution exceeds Cᵢ, the proof no longer applies.

### Common mistakes

- Giving the higher RM priority to the longer period. The shorter period is the higher priority.
- Treating the RM bound as necessary. It is sufficient. Failure of the inequality does not by itself prove a deadline miss; the 0.971 example needed a timeline to show the miss.
- Using U ≤ 0.693 as the test for two tasks. The two-task bound is approximately 0.828. The value 0.693 is the limit for many tasks.
- Comparing EDF by period instead of by absolute deadline.
- Applying the U ≤ 1 test when the deadline is shorter than the period. These notes use that test only for deadline equal to period.

### Important exam points

- RM: shorter period, higher fixed priority.
- Bound for two tasks ≈ 0.828. For a large number of tasks the bound approaches ln 2 ≈ 0.693.
- Worked success: C/T = 0.40 and 0.35, U = 0.75 ≤ 0.828.
- Worked contrast: U = 34/35 ≈ 0.971. RM misses at time 7. EDF finishes that job at time 6.
- EDF condition: U ≤ 1 when the relative deadline equals the period.

### University exam questions

#### 2-mark questions

1. State the Rate Monotonic priority rule.
2. State the EDF selection rule.
3. Write the RM utilization bound for n tasks.
4. Write the EDF schedulability condition used when the deadline equals the period.

#### 4/5-mark questions

1. Explain Rate Monotonic scheduling with the task set (20, 50) and (35, 100).
2. Explain why U ≤ n(2^(1/n) − 1) is a sufficient test and not a necessary one.
3. Differentiate RM and EDF.

#### 8/10-mark questions

1. For tasks A (C = 2, T = 5) and B (C = 4, T = 7), compute the utilization, test the RM bound, and draw RM and EDF schedules for the first deadlines.
2. Explain RM and EDF, including both utilization tests, and compare fixed and dynamic priority.

### Practice problems

#### Easy

1. Task P has period 10 and task Q has period 30. Which has the higher RM priority?
2. A job is released at time 40 and its period is 15. What is its absolute deadline if the deadline equals the period?
3. Is U = 0.75 acceptable under the two-task RM bound?
4. Is U = 1.2 schedulable on one processor by EDF?
5. Which algorithm changes priority as deadlines change?

#### Medium

1. Compute the RM bound for n = 2 from 2(√2 − 1).
2. Tasks have utilizations 0.3 and 0.4. Does the RM sufficient test accept them?
3. Draw the RM chart for T1 (C = 20, T = 50) and T2 (C = 35, T = 100) up to time 75.
4. At time 5 in the A/B example, why does EDF continue B instead of switching to A?
5. State one task set in this topic that RM misses and EDF does not.

#### Hard

1. Three tasks have utilization 0.2 each. Compare their total utilization with the three-task RM bound 0.780 and with the EDF limit.
2. Explain a task set with U = 0.90 and n = 2. What can the RM bound say before a timeline is drawn? What can EDF say?
3. A job of B has used 3 units by time 5 and needs 4. Its deadline is 7. Show the miss under RM.
4. Why is RM called optimal among fixed-priority algorithms?
5. Two jobs have the same absolute deadline. What does EDF allow?

**Answers**

- Q has period 30, so P, with period 10, has the higher RM priority.
- Absolute deadline = 40 + 15 = 55.
- U = 0.75 ≤ 0.828, so the RM test accepts it.
- U = 1.2 > 1, so EDF cannot schedule it on one processor.
- Utilizations 0.3 and 0.4 give U = 0.70 ≤ 0.828, so RM accepts them.
- Three tasks at 0.2 each give U = 0.60, which is within 0.780 and within 1.
- For U = 0.90 and n = 2, RM’s sufficient test does not accept the set, and it does not prove a miss. EDF accepts it because 0.90 ≤ 1.
- EDF may run either job when the absolute deadlines are equal.

### MCQs

**Q1. Under Rate Monotonic scheduling, the highest priority belongs to the task with:**

A. The shortest period  
B. The longest period  
C. The largest process identifier  
D. The largest execution time, regardless of period  

**Answer:** A  

**Explanation:** The rate is 1/T. A shorter period is a higher rate and receives the higher fixed priority.

**Q2. The RM utilization test is:**

A. A sufficient condition  
B. A necessary and sufficient condition for every n  
C. Used only for non-preemptive FCFS  
D. U ≤ 2 for one processor  

**Answer:** A  

**Explanation:** If the bound holds, the set is schedulable. If the bound fails, the set may or may not be schedulable.

**Q3. For two tasks, the RM bound is approximately:**

A. 0.828  
B. 0.500  
C. 2.000  
D. 0.100  

**Answer:** A  

**Explanation:** n(2^(1/n) − 1) for n = 2 is 2(√2 − 1) ≈ 0.828.

**Q4. EDF runs the ready job with:**

A. The earliest absolute deadline  
B. The longest period  
C. The smallest process name  
D. The largest remaining burst only  

**Answer:** A  

**Explanation:** The decision uses absolute deadlines, so the priority of a task changes from job to job.

**Q5. For periodic tasks whose deadline equals the period, EDF on one processor requires:**

A. Total utilization at most 1  
B. Total utilization at most 0.693 only  
C. Every period to be equal  
D. Non-preemptive execution  

**Answer:** A  

**Explanation:** U ≤ 1 is the schedulability condition for this case. The value 0.693 is the limiting RM bound, not the EDF bound.

**Q6. For tasks A (2, 5) and B (4, 7), the utilization is:**

A. 34/35  
B. 6/35  
C. 1.20  
D. 0.50  

**Answer:** A  

**Explanation:** 2/5 + 4/7 = 14/35 + 20/35 = 34/35 ≈ 0.971.

**Q7. In that task set, RM misses a deadline because:**

A. At time 7, B still has one unit of work and its deadline has arrived  
B. A has the longer period  
C. Utilization is greater than 1  
D. EDF forbids preemption  

**Answer:** A  

**Explanation:** B runs only during 2–5 before A preempts it. One unit remains at the deadline.

**Q8. On that same task set, EDF finishes the first B job at time:**

A. 6  
B. 7, with a miss  
C. 2  
D. 20  

**Answer:** A  

**Explanation:** B runs during 2–6 because its deadline 7 is earlier than A’s next deadline 10. The job meets the deadline.

### Quick revision

#### Rules

- RM: shorter period, higher fixed priority.
- EDF: earliest absolute deadline. Equal deadlines may be broken either way.

#### Tests, when deadline = period

```text
RM:  U ≤ n (2^(1/n) − 1)     sufficient
EDF: U ≤ 1                    necessary and sufficient on one processor
```

Two-task RM bound ≈ 0.828. Many-task limit ≈ 0.693.

#### Numbers to remember

- (20/50) + (35/100) = 0.75. RM succeeds. T2 finishes the first job at time 75.
- (2/5) + (4/7) = 34/35. RM misses at time 7. EDF completes that job at time 6.
