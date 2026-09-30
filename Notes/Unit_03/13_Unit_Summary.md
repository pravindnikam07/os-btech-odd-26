**Navigation:** [Unit index](00_Index.md) · [Previous: 3.12 Deadlock detection and recovery](12_Deadlock_Detection_and_Recovery.md) · Next: —

---

**Subject:** Principles of Operating Systems (03010803PC05)  
**Course / Semester:** BTech, Semester 3  
**Unit:** 3 — Process Synchronization and Deadlocks

# Unit summary

Unit 3 has two halves. The first half stops concurrent processes from corrupting shared data: critical sections, hardware instructions, Peterson’s solution, semaphores, the three classical problems, monitors, mutexes, spinlocks, and lock-free updates. The second half stops a set of processes from waiting forever for each other: the four necessary conditions, prevention, the Banker’s algorithm, detection, and recovery.

## Topic table

| Topic | Key concept | Rule or formula | Point to remember |
| --- | --- | --- | --- |
| Critical section | Code that uses shared data | Mutual exclusion, progress, bounded waiting | Two increments of 5 can finish at 6 |
| Test-and-set | Atomic “read and set true” | Enter if the old value was false | The simple spin has no bounded waiting |
| Compare-and-swap | Store only if the word still equals the expected value | `while (CAS(&lock, 0, 1) != 0)` | Also the primitive behind lock-free code |
| Peterson | Two-process software solution | `flag[i] = true; turn = j; while (flag[j] && turn == j)` | The last writer of `turn` waits |
| Semaphore | Integer changed only by P and V | Blocking: sleep if S < 0 after the decrement | Binary mutex starts at 1; counting starts at the pool size |
| Producer-consumer | n buffer slots | `empty = n`, `full = 0`, `mutex = 1` | `wait(empty)` before `wait(mutex)` |
| Readers-writers | Many readers or one writer | First reader locks `wrt`; last reader unlocks it | Writers can starve |
| Dining philosophers | Five chopsticks | Even and odd pick up in opposite orders | All-left-then-right deadlocks |
| Monitor | One process inside | Condition `wait` releases the monitor | Mesa semantics need `while`; a condition signal is not stored |
| Mutex | Lock with an owner | Only the owner unlocks | Not the same as a binary semaphore |
| Spinlock | Busy-wait lock | Short critical sections on multicore machines | Not lock-free, and dangerous on one CPU if the holder is preempted |
| Lock-free update | CAS retry, no lock held | Some thread completes even if another stalls | ABA: the bits return to A |
| Deadlock | Closed set of waits | All four conditions are necessary | Not the same as starvation |
| Prevention | Deny one condition | Increasing resource numbers deny circular wait | The Banker’s algorithm is not prevention |
| Banker | Grant only a safe state | Need = Max − Allocation | Sequence ⟨P1, P3, P4, P0, P2⟩; refuse the later P0 (0, 2, 0) |
| Detection | Is there a deadlock now? | Single instance: wait-for cycle. Several instances: Request against Work | Recovery must not roll back the same victim forever |

## Important definitions

- Critical section, race condition, mutual exclusion, progress, bounded waiting.
- Test-and-set, compare-and-swap, semaphore, binary semaphore, counting semaphore.
- Monitor, condition variable, mutex, spinlock, lock-free.
- Deadlock, safe state, unsafe state.
- Prevention, avoidance, detection, recovery.

## Important formulas and initial values

```text
TAS:  return the old boolean and set true
CAS:  if value == expected, value = new; return the old value

Peterson entry:  flag[i] = true; turn = j; while (flag[j] and turn == j)

Semaphore wait:    S = S − 1; if (S < 0) sleep
Semaphore signal:  S = S + 1; if (S ≤ 0) wake one

Buffer:  mutex = 1, empty = n, full = 0
Readers: mutex = 1, wrt = 1, readcount = 0

Need = Max − Allocation
Work starts as Available
If Need[i] ≤ Work, Work = Work + Allocation[i]
```

## The Banker’s snapshot

Available = (3, 3, 2).

| Process | Allocation | Max | Need |
| --- | --- | --- | --- |
| P0 | 0 1 0 | 7 5 3 | 7 4 3 |
| P1 | 2 0 0 | 3 2 2 | 1 2 2 |
| P2 | 3 0 2 | 9 0 2 | 6 0 0 |
| P3 | 2 1 1 | 2 2 2 | 0 1 1 |
| P4 | 0 0 2 | 4 3 3 | 4 3 1 |

Safe sequence: ⟨P1, P3, P4, P0, P2⟩.  
P1’s request (1, 0, 2) is granted. Available becomes (2, 3, 0).  
P0’s later request (0, 2, 0) is refused.

## Important diagrams

- Lost update of a shared counter.
- Peterson’s two flags and `turn`.
- Producer and consumer order of waits.
- Five philosophers and one chopstick between each pair.
- Monitor with a condition queue.
- Two-lock cycle, then the same cycle as a wait-for graph.
- Safety table with Work growing after each process.

## Important comparisons

- Progress and bounded waiting.
- Test-and-set and compare-and-swap.
- Busy-wait semaphore and blocking semaphore.
- Binary semaphore and mutex.
- Mutex and spinlock.
- Hoare signal and Mesa signal.
- Lock-free and wait-free; spinlock and lock-free.
- Deadlock and starvation.
- Prevention, avoidance, and detection.
- Safe, unsafe, and deadlocked.

## Examination priorities

- The three critical-section requirements, with a race trace.
- Peterson’s four lines and the reason the last writer of `turn` waits.
- Semaphore P and V, and the negative value in the blocking form.
- The producer-consumer program and the deadlock if mutex is taken first.
- Readers-writers starvation, and the philosophers’ cycle plus one cure.
- A monitor, a condition variable, and `while` under Mesa semantics.
- Banker’s Need table, one safe sequence with Work shown, and one refused request.
- Wait-for cycle, and recovery without starving the victim.
