# Processes

## What is a Process?

A process is an instance of a program in execution

Every time a user runs a program by typing the name of an executable object file to the shell, the shell creates a new process and then runs the executable object file in the context of the new process

Context consists of
- Process ID
- Address space (code, data, heap, stack)
- Processor state (registers)

A process provides each program with two key abstractions:
- *Logical control flow:* each program seems to have exclusive use of the CPU
- *Private address space:* each program seems to have exclusive use of main memory

Maintained by:
- Process executions interleaved (multitasked)
- Main memory address spaces managed (virtual memory system)

### Multiprocessing

- Single processor executes multiple processes concurrently
- The CPU runs one process at a time
- Address spaces managed by virtual memory system
- Execution context (register values, stack, ...) for other processes saved in memory

## Processes in Linux

`$ ps`

**PID:** the unique process ID
**TTY:** the terminal type you're logged into
**TIME:** the total amount of CPU usage
**CMD:** the name of the commands that launched the process

`$ ps aux`

**ps:** the process status command
**a:** displays information about other users' processes as well as your own
**u:** displays the processes belonging to the specified usernames
**x** includes processes that do not have a controlling terminal