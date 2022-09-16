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