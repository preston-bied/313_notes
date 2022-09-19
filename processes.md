# Processes

## What is a Process?

A process is an instance of a program in execution

Every time a user runs a program by typing the name of an executable object file to the shell, the shell creates a new process and then runs the executable object file in the context of the new process

Context consists of
- Process ID
- Address space (code, data, heap, stack)
- Processor state (registers)

A process provides each program with two key abstractions:
- **Logical control flow:** each program seems to have exclusive use of the CPU
- **Private address space:** each program seems to have exclusive use of main memory

Maintained by:
- Process executions interleaved (multitasked)
- Main memory address spaces managed (virtual memory system)

### Multiprocessing

- Single processor executes multiple processes concurrently
- The CPU runs one process at a time
- Address spaces managed by virtual memory system
- Execution context (register values, stack, ...) for other processes saved in memory

### Context Switch

A context switch is an operation that reassigns the CPU from one process or task to another

The execution of the process that is present in the running state is suspended by the kernel and another process that is present in the ready state is executed by the CPU

## Processes in Linux

`$ ps`

- **PID:** the unique process ID
- **TTY:** the terminal type you're logged into
- **TIME:** the total amount of CPU usage
- **CMD:** the name of the commands that launched the process

`$ ps aux`

- **ps:** the process status command
- **a:** displays information about other users' processes as well as your own
- **u:** displays the processes belonging to the specified usernames
- **x** includes processes that do not have a controlling terminal

## When is a Process Created?

Processes can be created in two ways:
- **System initialization:** one or more processes created when the OS starts up
- **Execution of a process creation system call:** something explicitly asks for a new processes

System calls can come from:
- User request to create a new process (system call executed from user shell)
- Already running processes
    - User programs
    - System daemons

## Creating New Processes and Programs

fork-exec model (Linux):
- `fork()` creates a copy of the current process
- `exec()` replaces the current process' code and address space with the code for a different program
    - Family: `execv`, `execl`. `execve`, `execle`, `execvp`, `execlp`
- `fork()` and `execve()` are *system calls*

Other system calls for process management:
- `getpid()`
- `exit()`
- `wait()`, `waitpid()`