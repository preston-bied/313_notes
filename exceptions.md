# Exceptions

## Exception Control Flow

An exception results in the transfer of control to the OS in response to some event

1. Current event (User Process)
2. Exception (to OS)
3. Exception processing by exception handler (OS)
4. Execution return (optional) (to User Process)
5. Next event

## Synchronous Exceptions

Synchronous exceptions can be triggered by executing an instruction

From within the user application there are 3 types of synchronous exceptions:
1. Faults
2. Aborts
3. Traps or System Calls

## Examples: Exceptions in Intel Processors

| Exception # | Exception                      |
| ----------- | ------------------------------ |
| 0           | Fault: Divide error            |
| 13          | Fault: Segmentation fault      |
| 14          | Fault: Page fault              |
| 18          | Abort: Machine check           |
| 32-127      | Interrupt or trap (OS-defined) |
| 128         | Trap                           |
| 129-255     | Interrupt or trap (OS-defined) |

## Faults

Faults are exceptions detected before the instruction executes, or during execution of the instruction

If detected during the instruction, the fault is reported with the machine restored to a state that permits the instruction to be *restarted*

1. Event (User Process)
2. Fault happened (to OS)
3. Fault handled (OS)
4. Return to same instruction (to User Process)
5. Event (User Process)

### Fault Example 1: Memory Reference

```c++
int a[1000];
main () {
    a[500] = 13;
}
```

- User writes to memory location
- That page of user memory is not mapped yet (because memory pages are mapped only when necessary)
- Page handler must load page into physical memory
- Returns to faulting instruction
- Successful on second try

1. Event (User Process)
2. Page fault (to OS)
3. Allocate page and its mapping in page table (OS)
4. Return (to User Process)
5. Event (User Process)

## Fault Example 2: Illegal Memory Reference

```c++
int a[1000];
main () {
    a[5000] = 13;
}
```

- User writes to memory location
- Address is not valid
- Page handler detects invalid address
- Sends `SIGSEGV*` signal to user process
- User process exits with "segmentation fault"

1. Event (User Process)
2. Segmentation fault (to OS)
3. Detect invalid address (OS)
4. Signal process

## Abort

Aborts are severe and unrecoverable errors

- Examples: parity error, machine check, divide by zero
- Aborts current program and hands control over to the OS
- This is the way for the OS to put essential error checking
- Also, an excellent way to make OS resilient when applications are failing or crashing

1. Fatal hardware error occurs (Application Program)
2. Control passes to handler (tp Exception Handler)
3. Handler runs (Exception Gandler)
4. Handler aborts execution (Exception Handler)

## Traps or System Calls

A trap is a synchronous interrupt *intentionally* triggered by an exception in a user process to perform an action

Exception conditions like invalid memory access, division by zero, or a breakpoint can trigger a trap in an OS

Examples:
- Application program requests more heap memory
- Application program requests I/O

Traps provide a function-call-like interface between application and OS

1. Application program traps (Application Program)
2. Control passes to handler (to Exception Handler)
3. Handler runs (Exception Handler)
4. Handler returns control to *next* instruction (to Application Program)

### Types of System Calls

**Process Control**
- load
- execute
- end, abort
- create process
- terminate process
- get/set process attributes
- wait for time, wiat event, signal event
- allocate, free memory

**File Management**
- create file, delete file
- open, close
- read, write, reposition
- get/set file attributes

**Device Management**
- request device, release device
- read, write, reposition
- get/set device attributes
- logically attach or detach devices

**Information Maintenance**
- get/set time or date
- get/set system data
- get/set process, file, or device attributes

**Communication**
- create, delete communication connection
- send, receive messages
- transfer status information
- attach or detach remote devices

### System Calls: Example

```c++
#include <unistd.h>

int main(void) {
    write(1, "Hello, world\n", 13);
    return 0;
}
```

**Making System Calls in 32-but Linux:**
1. Load system call number in register `eax`
2. Load arguments to system call in registers `ebx`, `ecx`, `edx`, `esi`, `edi`, `ebp`
3. Invoke software interrupt: `int 0x80`

Returned values are stored in `eax`

## Traps in Intel Processors

To execute a trap, application program should:
- Place number in `eax` register indicating desired functionality
- Place parameters in `ebx`, `ecx`, `edx` registers
- Execute assembly language instruction `int 128`

```
movl $45, %eax     // in Linux, 45 indicates request for more heap memory
movl $1024, %ebx   // request is for 1024 bytes
int  $128          // causes trap
```

## Summary of Exception Classes

| Class           | Cause                                       | Asynch/Sync | Return Behavior                       |
| --------------- | ------------------------------------------- | ----------- | ------------------------------------- |
| Interrupt       | Signal from I/O device                      | Ansynch     | Return to next instruction            | 
| Fault           | Unintentional and (maybe) recoverable error | Sync        | (Maybe) return to current instruction |
| Abort           | Unintentional and non-recoverable error     | Sync        | Do not return                         |
| Trap / Sys Call | Intential from within user application      | Sync        | Return to next instruction            |