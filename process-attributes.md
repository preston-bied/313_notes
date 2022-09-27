# Process Attributes

## Program Execution Flow

1. Load instruction and data segments of executable file into memory
2. Create stack and heap
3. Transfer control to it
4. Provide services to it (network, file connections, I/O, etc.)

## Overview of Memory Regions

- **Stack:** This section contains the temporary data such as method/function parameters, return addresses, and local variables
- **Heap:** This section is dynamically allocated memory to a process during its run time.
- **Text:** This section includes the current activity represented by the value of the program counter and the contents of the processor's registers
- **Data:** This section contains the global and static variables

## The Life Cycle of a Process

1. **Start:** This is the initial state when a process is first started/created.
2. **Ready:** The process is waiting to be assigned to a processor. Ready processes are waitinf to have the processor allocated to them by the OS so that they can run. Processes may come into this state after the start state or while running and interrupted by the scheduler to assign the CPU to some other process.
3. **Running:** Once the process as been assigned to a processor by the *OS scheduler*, the process state is set to running and the processor executes its instructions.
4. **Waiting:** A process moves into the waiting state if it needs to wait for a resource, such as waiting for user input or waiting for a file to become available.
5. **Terminated/Exited:** Once the process finishes its execution or it is terminated by the OS, it is moved to the terminated state where it waits to be removed from the main memory.

## How the OS Represents a Process in the Kernel

At any time, there are many processes in the system, each in its particular state.

The OS data structure representing each process is called the *Process Control Block (PCB)*.

- The PCB contains all of the info about a process
- The PCB is also where the OS keeps all of a process's hardware execution state (PC, SP, registers, etc.) when a process is not running
- This state is everything that is needed to restore the hardware to the same configuration it was in when the process was switched out of the hardware

## Process Control Block (PCB)

- **Process State:** The current state of the process, i.e., whether it is ready, running, waiting, etc.
- **Process Privileges:** This is required to allow/disallow access to system resources
- **Process ID:** Unique indentification for each of the processes in the OS
- **Pointer:** A pointer to the parent process
- **Program Counter:** A pointer to the address of the next instruction to be executed for the process
- **CPU Registers:** Various CPU registers where the process needs to be stored for execution for the running state
- **CPU Scheduling Information:** Process priority and other scheduling information which is required to schedule the process
- **Memory Management Information:** This includes the information of the page table, memory limits, and segment table depending on the memory used by the OS
- **Accounting Information:** This includes the amount of CPU used for process execution, times limits, execution ID, etc.
- **I/O Status Information:** This includes a list of I/O devices allocated to the process

## OS's Internal Tables

An OS keeps a lot of information in the main memory and much of this info is about:
- Resources (e.g., devices, memory state)
- Running programs/processes

Note that program data is not the same as the PCB.

The PCB is a process's metadata.

Program data (variables, allocated memory) and code are kept in the process image (i.e., address space), which is seperate and usually bigger in footprint.

## Context Switch: PCB and Hardware States

1. When a process is running, its hardware state (PC, SP, registers, etc.) is in the CPU. The hardware registers contain the current values.
2. When the OS stops running a process, it saves the current values of the registers into the process's PCB.
3. When the OS is ready to start executing a new process, it loads the hardware registers from the values stored in that process's PCB.

The process of changing the CPU hardware state from one process to another is called a *context switch*.

## States of a Process

**User view:** A process is executing *continuously*

**In reality:** Several processes *compete* for the CPU and other resources

A process may be:
- **Running:** It holds the CPU and is executing instructions
- **Ready:** It is waiting to get back on the CPU
- **Blocked:** It is waiting for some I/O event to occur

## Process Scheduling Queues

- **Job Queue:** Set of all processes in the system
- **Ready Queue:** Set of all processes residing in main memory that are ready and waiting to execute
- **Device Queues:** Set of processes waiting for an I/O device

Processes migrate among the various queues.

## Schedulers

**Long-Term Scheduler (Job Scheduler)**
- Selects which processes should be brought into the ready queue
- Controls degree of multiprogramming
- Must select a good process mix of I/O-bound and CPU-bound processes

**Short-Term Scheduler (CPU Scheduler)**
- Selects which process should be executed next and allocates CPU
- Executes at least every 100ms, therefore must be very fast

**Medium-Term Scheduler (Swapper)**
- In some OS's
- Sometimes good to temporarily remove processes from memory (suspended)
