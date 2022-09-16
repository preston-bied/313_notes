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