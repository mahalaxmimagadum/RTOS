## READ FILE README FOR MORE CLARIFICATION 

 Reset:
Purpose: Initializes the system into a default, ready state.
Operation: When the system powers up or restarts, the reset function sets up the core values, such as initializing the mutex to an unlocked state, setting the system time to zero, and clearing all task entries.
Outcome: The system is now ready to accept and manage tasks.

2. Create Task:
Purpose: Allows the system to add a new task to be executed.
Operation: The system searches for an available Task Control Block (TCB) in its list. When it finds one, it initializes it with default values, including setting the task’s starting address (program counter) and an initial status.
Outcome: The new task is added to the ready queue, awaiting execution.

4. Scheduler:
Purpose: Manages which task should run next.
Operation: The scheduler selects the next task from the ready queue in a round-robin fashion, moving each task from the ready state to the running state in sequence.
Outcome: Ensures that each task gets a turn to execute, maintaining an orderly task progression.

6. Dispatcher:
Purpose: Prepares the environment for the selected task to run.
Operation: The dispatcher retrieves the saved state of the next task from its TCB (e.g., register values and program counter) and restores these to the system. It then starts execution from the task’s last saved program counter.
Outcome: The selected task resumes from where it left off.

8. Interrupts:
Purpose: Temporarily pause the current task to handle high-priority events, like system calls or timer events.
Types:
Timer Interrupts: Triggered periodically to keep track of time-dependent operations, such as managing task wait times.
Software Interrupts: Triggered by system calls to request operations like creating or deleting tasks.
Operation: When an interrupt occurs, the system saves the current task’s state in its TCB (including registers and program counter) to resume later.
Outcome: Allows the system to handle urgent tasks without losing the current task's progress.

10. Wait Time:
Purpose: Allows a task to delay its execution for a specified period.
Operation: When a task requests a wait, it’s moved to a wait queue, and a countdown timer is set. As time progresses, the scheduler or timer interrupt checks each waiting task’s timer and moves it back to the ready queue once the wait time has elapsed.
Outcome: The task will resume execution after its wait time expires.

12. Mutex (Mutual Exclusion):
Purpose: Ensures that only one task can access a shared resource at any given time.
Operation:
Wait on Mutex: If a task needs exclusive access to a resource, it waits for the mutex to be available. If the mutex is already taken, the task is added to a wait queue.
Signal Mutex: When a task finishes using the resource, it signals the mutex to release it, allowing the next waiting task to take over.
Outcome: Prevents conflicts and ensures safe access to shared resources by allowing only one task to hold the mutex at a time.

14. Delete Task:
Purpose: Removes a specified task from the system when it’s no longer needed.
Operation: The system finds the task in its TCB list and marks it as "unused," effectively removing it from the ready queue.
Outcome: Frees up resources, making the TCB available for new tasks.
