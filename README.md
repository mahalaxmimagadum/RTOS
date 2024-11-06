##Real-Time Embedded Systems

Overview of Multitasking and Time-Slicing System
The system we have outlined here supports multitasking, enabling several tasks to run simultaneously on a single processor. Here we make use of Task Control Block (TCB) list, where each TCB corresponds to an individual task within the operating system. Multitasking and Time-Slicing principles in the runtime system ensures that the tasks share the processor’s time fairly. The scheduler, which is a key component, cycles through tasks in a Round Robin fashion, allowing each task a designated time. 
In a runtime system, each task is represented by a TCB which stores all the essential information such as Status Register (SR), Program Counter (PC), Address Register (A0-A7), Data Registers (D0-D7) and a pointer.
 
Mutexes and system calls namely ‘syscr’, ‘sysdel’, ‘syswaitmx’, ‘syssiggmx’, ‘syswttm’, ‘sysinmx’ are illustrated for synchronization mechanisms which contribute to the seamless execution of tasks and also providing responsive runtime system environment. 

Implementation of Multitasking and Time-Slicing System
Implementation of multitasking and Time-Slicing in this system is realized through the utilization of Task Control Block (TCB) list making sure multiple tasks can happen at once with the help of mutex ( mutual exclusion) which helps in the synchronization. It has got a handle on task creation and deletion, making sure each task has its own set of instructions to follow. The scheduler keeps the order in which the tasks are executed by updating the ready task list. Each of the user functions that was implemented in this RTS has been explained in the coming sections. 

System Call Routing in the RunTime System
In operating system the system call routine is responsible for determining the action to take based on the value of the ‘id’ parameter. This ‘id’ has been initialized in the RTS variables in the run time system outline program. Each unique ‘id’ corresponds to a specific system call, directing to the program flow to the appropriate subroutine.
Now the value of the ‘id’ is transferred to the register d0, then the value in the register d0 is compared with 1 using ‘cmp.l’ function. If equal, the ‘id’ branches to the “Create_Task” subroutine indicating the system call for creating a new task. (in the first case) 
                         move.l  id,d0
                         cmp.l   #1,d0
                          beq     Create_task

Subsequently the system performs comparisons to check for other possible system call values. If a match is found, the program branches to the corresponding subroutine. Default branch is also set for the system when the previous comparisons does not match, the program unconditionally branches to the ‘sched’ (scheduler) subroutine.


Memory Layout of the User Task
The memory layout of this runtime system is structured to accommodate Task Control Block (TCB) that is number of TCB records (‘ntcblst’), length of the TCB (‘tcblen’) and User Task Memory. In the user task memory ‘usrcode’ represents the starting address of the user task code which is initialized to $1000 which indicates the location in the memory where the instructions of the user tasks are stored. User task’s stack (‘usrstk’) is initialized to $8000. The stack is a region of memory used for managing function calls. The length of the TCB is initialized to $60 (tcblen     equ     $60) which holds the essential information about the task and also extra space is provided for the buffer.

Explanation of the startup behaviour
The startup behaviour for the designed runtime system involves execution of reset routine (‘res’) in response to the system reset. Reset typically refers to the initialization and preparation of the system for execution. This includes setting initial values, configuring data structures, and readying tasks or processes for execution. 
In the code to begin with, the status register(‘sr’) is set to a specific value, potentially configuring interrupt levels and other system-specific settings, initializes register ‘d0’ to 0 and system time variable(‘time’) to 0. Next it initializes the TCBs by iterating through the TCB list, marks each TCB as unused(‘tcbused(a0)’), advances to the next TCB in the list and repeats until all TCBs are initialized. The address of the user code(‘usrcode’) is loaded into the register ‘d0’. Pc is set, ‘#$00002000, d0’ configures the status register in the TCB sets the status register in TCB and marks the TCB as used and links the TCB to the ready task list. ‘and     #%1111100011111111, sr’ clears certain bits in the status register, potentially adjusting the interrupt levels. ‘jmp usrcode’ jumps the user code(‘usrcode’) to initiate normal system operation. During the initial execution the interrupt vectors are also involved (‘fltint’ and ‘flsint’) to configure timmer interrupts and system call traps respectively. The dispatcher(‘disp’) is called during the timer interrupt handling (‘fltint’) which then finds out which is the next task from the ready list and restores its content by switching the execution from the current task to the next task. During the initial startup behaviour, the runtime system does not create additional tasks automatically. 
Description of User Functions and Parameters

•	Create Task
The create task routine is responsible for establishing a new task within the task environment. It involves allowing necessary resources such as memory and defining the execution parameters for the task which include start address of the new task and the address of the top of the stack. 
In the beginning, task ID is loaded into a register ‘d0’. Register ‘d1’ is set to 0. The code then searches for an unused Task Control Block(TCB) in a TCB list(‘tcblist’) by iterating though the list until an unused TCB is found. Having found an unused TCB, it sets the program counter(‘tcbpc’), status register(‘tcbsr’), and stack pointer(‘tcba7’) in the TCB. Thus marking the TCB as used(‘tcbused’). TCB linked list is updated, linking the new TCB to the ready task list. Lastly, branches to the scheduler(‘sched’) to handle task scheduling.
                         move.l  #1, d3
                         move.l  d3, tcbused(a0)
                         move.l  rdytcb,a1

Parameters: Start address of the new task (‘#usrcode’) represents the start address of the user task code which is         stored in the Program Counter.
Top-of-stack represents the memory location that points to the top stack of the new task.
                        move.l  d1, tcbpc(a0)
                        move.l  #$00002000, tcbsr(a0)
                        move.l #$01000000,tcba7(a0)


•	Delete Task
The Delete task routine has to terminate the task and remove the TCB from the list and mark as unused and also return the memory allocated to the task to the system.
Firstly, the code Initializes the pointer ‘a0’ to the beginning of the TCB list (‘#$12000’) and ‘d1’ to the total number of TBC’s. The search for the task in the TCB list begins, it iterates through the tcb list to find a tcb marked as in use (‘tcbused(a0) ==1’). If the match is found it proceeds to the “Task_Found”. In the “Task_Found”, it updates the TCB linked list to bypass the TCB being deleted. Connects the previous                                               
                         cmp.l   #1,tcbused(a0)
                         beq     Task_Found
                         add.l   #tcblen, a0
                         sub.l   #1, d1

TCB(‘tcbnext(a0)’) to the next TCB(‘a1’), marks the TCB as unused(‘tcbused(a0)’ set to 0), updates the previous TCBs next pointer to skip the deleted TCB(‘(a1)’ set to ‘a0’), loops until the task is deleted and once the task is deleted, returns from the subroutine. If the task is not found the code returns from the subroutine without making any changes. 


•	Wait Mutex
Wait Mutex operation is typically used for synchronization to control access to shared resources among multiple tasks. A mutex (mutual extension) ensures that only one task can access a critical section of the code or a shared resource at a time. The “wait mutex” operation involves a task attempting to acquire a mutex. If the mutex is available, the task continues its execution, it enters a wait state until the mutex becomes available. 
In the code firstly, the address of the mutex is loaded into ‘d0’.  Now the mutex availability is checked by comparing the values of the mutex with 1, indicating availability (‘cmp.l #1, d0) and branches to “MutexAvailable”, if the mutex is available. If the mutex is not available (Wait State), links the task (pointed to by register ‘a0’) to the wait task queue (‘wttcb’), updates the wait task queue head and branches to “EndWaitMutex”. Once the mutex is available, mutex is set to 0, indicating it is now in use and links the task to the ready task queue (‘rdytcb tcbnext(a0)’) and then updates the ready task queue head and finally returns from the subroutine. 
                          move.l  wtlsln,d0
                          cmp     #0,d0
                         move.l  tcbnext(a2),tcbnext(a1)
                          move.l  wttcb, tcbnext(a0)


•	Signal Mutex
Signal Mutex routine is a significant element within the runtime system’s synchronization strategy, specifically designed for signalling a mutex and managing the tasks that seek access to shared resources. When a task finishes using a shared variable, it signals the mutex, making it available for other tasks. This process avoids conflicts and establishes order in accessing shared areas.
Firstly, in the code ‘syssiggmx’ routine grabs the address of the waiting task list, known as ‘wttcb’, storing it in the register a1. Following this, it assesses whether the waiting task list is empty of tasks by comparing the contents of d1 to 0. If it turns out that there are no tasks in the waiting list, the routine sets the ‘mutex’ flag to 1. Signalling the availability of the shared variable. If there is a task waiting, it shifts the focus to the next waiting task, updating the ready task list (‘rdytcb’) to exclude the recently signaled task and the waiting task is linked back to the ready task. Ultimately the “EndSignalMutex” exits the subroutine.
                          move.l  a1,tcbnext(a0)
                          move.l  wtlsln,d0
                          sub     #1,d0
                          move.l  d0,wtlsln


•	Initialise Mutex
The system call initialise mutex “sysinmx” is designed to initialize a mutex. It sets the mutex to an initial state where the shared variables are available by clearing the wait task list. The mutex is set to the value 0 or 1 referring to the availability of the shared variable. We load the contents of the register ‘d1’ into the mutex variable. ‘d1’ contains the value representing the initial state of the mutex. We the load the current value of the mutex into the register ‘d0’. Then the value of the mutex ‘d0’ is compared with ‘1’, if equal the mutex is already acquired by another task. 
Parameters: The parameters for initializing the mutex is a binary value, typically either 0 or 1. This parameter signifies the initial state of the mutex, where 0 often represents unavailable state and 1 represents available state.


•	Wait Time
Wait Time is a routine which pauses the execution of a task for a specified duration. During the waiting period the task is moved to waiting list, which introduces delays between certain operations. Wait time plays a pivotal role in multitasking environment, offering a mechanism to introduce controlled delays in task execution for precise timing and synchronization.
For the wait time system call we have designed a separate run time system where we have included the wait time program in the first-level interrupt handler. The purpose of this section is to manage the wait queue, ensuring that the tasks waiting for a specific duration (wait time_ are appropriately handled. Tasks whose wait time have expired are moved back to the ready queue for potential execution. In the code firstly, we retrieve the next task from the head of the wait queue (‘wttcb’). Then we load the wait time of the current task and decrement it by 1. If the wait time has expired, we then move to the current task from the wait queue to the ready queue (‘rdytcb’). The head of the wait queue is then updated (‘wttcb’). Linking the task and updating the ready queue is done hand in hand.
DecrementWaitTimes:       move.l  tcbwtim(a0),d1
                          cmp     #0,d1
                          beq     NextTask
                          sub.l   #1, d1 
In the actual wait time system call program the current task’s TCB is retrieved from the head of the (‘rdytcb’). Then the wait time is set by coping the value from the register D1 to the (‘tcbwtim’). We then continue to move through the ready queue until we find the task with the shortest wait time. Later linking the tasks to the ready queue, ensuring that they are in the correct order based on wait times. Finally, the current task in the wait queue is moved to the end of the wait queue, and the head of the wait queue is updated. 
Parameters: Wait time specifies the number of time intervals that the requesting task should wait. The value specifies the duration the task remains in a waiting state before being transferred back to the ready list. 

Execution Time Analysis
The Execution time analysis section provides a detailed examination of the estimated execution times for key functions within runtime system. Understanding the time complexity of these functions is crucial for assessing the overall performance and responsiveness of the system.

CREATE TASK
Creating the task from the task list, which might vary based on the number of tasks.                                                       Could be in the range of 45-60 Instructions.

DELETE TASK
Deleting a task from the task list. Varies based on the number of tasks,                                                                     Could be in the range of 75-85 Instructions.

WAIT MUTEX
This deals with the mutex initialization ensuring access to the shared variable.                                                      Could be in the range of 70-85 Instructions.

SIGNAL MUTEX
This system call deals with signalling of the mutex and update the mutex availability.                                                Could be in the range of 75-95 Instructions.

INITIALIZATION MUTEX
Initialize mutex system call is responsible for setting up a mutex, initializing its state, and making it ready for use by tasks. Could be in the range of 55-75 Instructions.

WAIT TIME
This part of the analysis focuses on the system call for waiting on a timer.                                                             Could be in the range of 85-95 Instructions.

Task Scheduling
When a new task is created, it is added to the end of the ready queue. The process of adding a new task to the ready queue involves locating an unused TCB in the TCB list, setting up its parameters, marking it as in use (‘tcbused’) and updating the linked list to include the newly created task. Regarding the scenario where two tasks are waiting for the same time and become ready together, the scheduling system will determine the order in which they are placed in the ready queue. Typically, the task that has been waiting longer (arrived first) might be given priority. This can also be termed as Round-Robin scheduling.
Scheduler
The scheduler is responsible for determining the order and the duration of the task execution. Its primary role is to manage the allocation of the CPU time among various tasks ensuring efficient utilization of system resources. It also coordinates the execution of multiple tasks concurrently, higher-priority tasks are given preferences in the execution. At the start the scheduler loads the address of the currently running task’s tcb into the register a0 from (‘rdytcb’). Next the address of the next task in line is collected from (‘tcbnext(a0)’) of the current task’s TCB. With the help of which we can determine the task that will be executed next. Finally, the (‘rdytcb’) is updated with the address of the next task, indicating that it is now the task ready for execution.
Scheduling Strategy
Our runtime system does not incorporate prioritized scheduling. Tasks within the system are treated equally, since the scheduling strategy is round-robin, all tasks that are ready have an equal chance of receiving CPU time. The actual run time of tasks may vary due to differences in their execution requirements. Tasks with longer execution times may receive less frequent opportunities to execute leading to delays.

Timer Interrupt Period and Task Switching
Timer Interrupt Period:
The timer interrupt period is defined by the duration between successive timer interrupts. In this runtime system, the timer interrupt occurs at the frequency determined by the hardware or software timer.
Timer Interrupt Period= 1000ms
Task Switching Duration:
Task switching refers to the time taken to switch from one task to another. Estimated task switching duration is 50 instructions.
Assuming task switching duration = 50 instructions and average instruction time = 1000ms
Proportion of time spent in RTS = 50μs / 1000000μs = 0.00005 or 0.005%
Available Time for user tasks 
Time available for user tasks = 1000ms - 50μs = 999950 μs.
Proportion of time available for user tasks = 999950 μs / 1000000 μs = 0.99995 or 99.995%
End result would be approximately 0.005% of the adjusted timer interrupt period is spent in the RTS. Around 99.995% of the adjusted timer interrupt period is available for executing user tasks.##
