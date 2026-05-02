# Assignment 3 - Complete Documentation

**Student Name**: [turki fahad alanzy]  
**Student ID**: [444050787]  
**Date Submitted**: [2/5/2026]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ yes] Link is accessible (tested in incognito mode)
- [ yes] Video is 3-5 minutes long
- [yes] Video shows code walkthrough and commits
- [yes] Video has clear audio
- [ yes] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [May 2, 2026, 1:30 AM]
**What I implemented**: 
GitHub repository setup and student identification.
**Challenges encountered**: 
Connecting the local VS Code environment to the remote repository.
**How I solved it**: 
Used the Git Credential Manager for browser-based authentication.
**Testing approach**: 
Verified the "Sync Changes" button appeared in VS Code.
**Time spent**: 
30 mins
---

### Entry 2 - [May 2, 2026, 02:30 AM]
**What I implemented**: 
Initializing synchronization tools (ReentrantLock and Semaphore).
**Challenges encountered**: 
Missing imports for concurrent libraries.
**How I solved it**: 
import java.util.concurrent.locks.ReentrantLock; and import java.util.concurrent.Semaphore;.
**Testing approach**: 
hecked for red syntax error lines in the editor.
**Time spent**: 
20 mins
---

### Entry 3 - [May 2, 2026, 02:50 AM]
**What I implemented**: 
Added synchronized access methods for shared counters and the execution log, centralizing all critical updates inside ReentrantLock-protected helper methods.
**Challenges encountered**: 
Ensuring that shared state updates did not interfere with each other while keeping the scheduler responsive and avoiding deadlock.
**How I solved it**: 
Encapsulated each shared update inside lock.lock() / try-finally blocks and kept critical sections short.
**Testing approach**: 
Ran the scheduler multiple times to verify consistent process counts, waiting time totals, and stable execution log behavior.
**Time spent**: 
30 mins

---

### Entry [May 2, 2026, 03:00 AM]
**What I implemented**: 
Protecting shared counters (contextSwitchCount, completedProcessCount) and the execution log.
**Challenges encountered**: 
Potential race conditions where multiple threads update the same counter simultaneously.
**How I solved it**: 
Implemented lock.lock() with try-finally blocks to ensure the lock is always released.
**Testing approach**: 
Manual code review to ensure all shared increments were inside critical sections.
**Time spent**: 
40 mins
---

### Entry 5 - [[May 2, 2026, 03:00 AM]]
**What I implemented**: 
execution control using a Semaphore.
**Challenges encountered**: 
InterruptedException when calling acquire().
**How I solved it**: 
the acquire() call in a try-catch block and ensured release() is called after execution.
**Testing approach**: 
Ran the simulation to ensure only one process executes at a time.
**Time spent**: 
30 mins
---

## Part 2: Technical Questions (1 mark)
 
### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:
: Two major race conditions exist: first, contextSwitchCount++ is not atomic; two threads could read the same value and increment it, leading to a lost update. Second, executionLog.add() involves modifying an ArrayList, which is not thread-safe. Concurrent access can cause data corruption or a ConcurrentModificationException.


---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:
 A ReentrantLock provides mutual exclusion for critical sections (only one owner). I used it to protect shared variables like counters. A Semaphore manages a set of permits; I used it with one permit to act as a gatekeeper for the CPU, ensuring only one process runs at a time.
---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

Deadlock is a state where threads are stuck waiting for each other to release resources. I prevented it by using try-finally blocks, ensuring lock.unlock() or semaphore.release() is called regardless of any errors. I also used a single lock to avoid complex lock-ordering cycles.

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:
I used ONE lock for all three counters and the shared execution log, which is a coarse-grained locking approach. This choice keeps the code simple and avoids complex lock ordering, since all shared updates are guarded by the same `ReentrantLock`. The trade-off is that it reduces concurrency slightly, because threads that only need to update one counter still serialize on the same lock. Fine-grained locking with separate locks for each counter could improve concurrency if the counters were updated independently in high-volume parallel code, but it would also add complexity and increase the risk of deadlock if lock ordering is not carefully managed. In this scheduler, the critical sections are short and the shared state is small, so the coarse-grained lock is the better practical choice. If the three counters were truly independent and the program had many simultaneous writers, separate locks would provide better concurrency by allowing different counters to be updated in parallel.

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: 
contextSwitchCount, completedProcessCount, totalWaitingTime.
**Why they need protection**: 
These variables are shared by all process threads and are updated concurrently during execution. Without protection, increments and addition operations can interleave, producing lost updates or incorrect totals.
**Synchronization mechanism used**: 
ReentrantLock.
**Code snippet**:
```java
SharedResources.lock.lock();
try {
    contextSwitchCount++;
} finally {
    SharedResources.lock.unlock();
}
```

**Justification**: 
The `ReentrantLock` ensures atomic updates to the shared counters and prevents race conditions when multiple threads increment or add to the same shared values.

---

### Critical Section #2: Execution Log

**What resource**: 
ArrayList<String> executionLog.
**Why it needs protection**: 
`ArrayList` is not thread-safe, so concurrent `add()` operations can corrupt its internal structure or throw exceptions.
**Synchronization mechanism used**: 
ReentrantLock.

**Code snippet**:
```java
public static void logExecution(String message) {
    lock.lock();
    try {
        executionLog.add(message);
    } finally {
        lock.unlock();
    }
}
```

**Justification**: 
Locking before modifying the execution log prevents concurrent writes and preserves a consistent, correct log history for the scheduler.

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 
To limit concurrent process execution to simulate a single CPU.
**Number of permits and why**: 
1 permit, because only one process should occupy the CPU at any given time.
**Where implemented**: 

**Code snippet**:
```java
// SharedResources.cpuSemaphore.release();
```

**Effect on program behavior**: 

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
# Commands used (run the program at least 5 times)
```

**Results**: 
Every run produced identical final values for total waiting time and completed processes. The execution log maintained a logical order without interleaved text.

**Why synchronization is necessary**: 
(Explain what race conditions COULD occur without synchronization, even if you didn't observe them. Explain which shared resources need protection and why.)
Without synchronization, a "Race Condition" could occur on the contextSwitchCount. For instance, if two threads read the counter at value 5, both might increment it to 6 and write it back, losing one increment. Resources like the executionLog (ArrayList) also need protection because they are not thread-safe

**Conclusion**: 
The implementation of ReentrantLock successfully eliminated data inconsistency.
---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: 
: Running the simulation with 10+ processes to increase thread contention on the executionLog list.
**Results**: 
No exceptions were thrown.
**What this proves**: 
This proves that the lock.lock() and lock.unlock() mechanism is correctly managing access to the shared ArrayList, preventing one thread from modifying the list while another is trying to read or add to it.
---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)
Verifying correct final values for scheduling metrics.
**Expected values**: 
Total Completed Processes = 10 (Total processes in queue).
**Actual values**: 
Total Completed Processes = 10.
**Analysis**: 
: The values matched perfectly, indicating that every process was accounted for and no increments were lost due to race conditions.
---

### Test 4: Different Scenarios
**Scenario tested**: [e.g., different time quantum, more processes, etc.]
Increasing the number of processes to 20 and reducing the Time Quantum.
**Purpose**: 
To see if the Semaphore effectively manages a high volume of process requests for the CPU.
**Results**: 
s: The simulation remained stable. The output showed clear STARTING and FINISHED blocks for each quantum proving the Semaphore(1) is acting as a strict gatekeeper
**What I learned**: 
Synchronization becomes even more critical as the number of threads increases and the time intervals between context switches decrease.
---

## Part 5: Reflection and Learning

### What I learned about synchronization:

I learned that multithreading is a powerful tool but can be highly risky if shared data is not handled with extreme caution. I discovered that even simple operations like ++ are not atomic at the low level which necessitates the use of Locks to maintain data integrity. I also gained a clear understanding of the fundamental difference between a ReentrantLoc used for protecting shared variables, and a Semaphore used to regulate access to limited resources like the CPU. Implementing try-finally blocks was a crucial lesson in preventing deadlocks during execution errors. Connecting these theoretical OS concepts with practical Java implementation helped me truly grasp how real-world processes are managed within a computer system

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 
Banking & ATM Systems: Synchronization is critical to ensure that simultaneous withdrawals from different locations do not allow a user to exceed their available balance due to a race condition in the balance update.
**Example 2**: 
Airline Reservation Systems: To ensure a single seat is not sold to multiple passengers who might attempt to complete their payment at the exact same millisecond.
---

### How I would explain synchronization to others:

[Explain to someone who just finished Assignment 1 - use simple terms and analogies]
Imagine the CPU is a single "public restroom" and the processes are people waiting in line. The Semaphore is the "key"; no one can enter unless they hold the key, and once they finish, they hand it to the next person. A Lock, on the other hand, is like a "shared safety deposit box"; if you want to put something in or take it out, you must lock it while you're using it so no one else can change the contents until you are done.
---

## Part 6: GitHub Repository Information

**Repository URL**: 
https://github.com/Turki-fahad-444050787/OS-Assignment3-Starter
**Number of commits**: 
5
**Commit messages**: 
1. 1/ Student ID
2. Initialize ReentrantLock and Semaphore
3. Protect shared counters
4. ogExecution + addWaitingTime 
5.Implement Semaphore to control CPU access

---

## Summary

**Total time spent on assignment**: 
6h
**Key takeaways**: 
1. 
2. 
3. 

**Most challenging aspect**: 
  short of time 
**What I'm most proud of**: 
in short time h finshed i am so happy
---

**End of Documentation**
