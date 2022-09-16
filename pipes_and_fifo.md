# Inter-process Communication: Pipes and FIFO

## WHat is a Pipe in Linux?

1. `write(fd1, buf2, n2)` into free space
2. `read(fd2, buf2, n2)` from data

A pipe is a construct in the Unix OS that allows two processes to communicate

One end of the pipe is the read end and the other is the write end

A pipe is create via a `pipe()` **system call**

`int pipe(int fd[2])`

The function returns a pair of **file descriptors**
- `fd[0]` is the **read** end
- `fd[1]` is the **write** end

Data is received in the order it is sent

## How to Create and Use a Pipe

```c++
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char* argv[]) {
    int fd[2];
    char buf[30];
    // create pipe
    if (pipe(fd) == -1) {
        perror("pipe");
        exit(EXIT_FAILURE);
    }
    // write to pipe
    write(fd[1], "CSCE 313", 9);
    // read from pipe
    read(fd[0], buf, 9);
    printf("read \"%s\"\n", buf);
    return 0;
}
```

`pipe()` takes an array of two `int`s as an argument

Return values of `pipe()`:
- -1: pipe failure
- 0: pope success

`pipe()` fills in the array with two file descriptors

When any bytes are written to `fd[1]` the OS makes them available for reading from `fd[0]`

## Linux Pipe Command

Unix shells use abstraction of a job to represent the processes that are created as a result of evaluating a single command line

`linux> ls | sort`

This command creates a foreground job consisting of two processes connected by a Unix pipe: one running the `ls` program and the other running the `sort` program

## UNIX fork()

`fork()` is a **system call** that creates a new process by duplicating the calling process

The process calling `fork()` is the **parent** process

The new process is the **child** process

`pid_t fork(void);`

Return values of `fork()`:
- `< 0` : **fork failure**, child process not created
- `= 0 `: **fork success**, value 0 returned to the child process
- `> 0` : **fork success**, process ID of the child process returned to the parent process

### UNIX fork() Example

```c++
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
    fork();
    printf("Welcome to CSCE 313!\n");
    return 0;
}
```
```
Welcome to CSCE 313!
Welcome to CSCE 313!
```
The number of times the message prints is equal to the number of processes created: **2<sup>n</sup>**