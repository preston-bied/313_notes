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

## fork() and pipe()

It is not useful for one process to use a pipe to talk to itself

Typically, a process creates a pipe just before it forks more child processes

The pipe is then used for communication either between the parent and child processes or between two sibling processes

In the following program, the parent writes a message to the pipe and the child reads from the pipe 1 byte at a time until the pipe is empty

### fork() and pipe(): Example 1

```c++
int main(int argc, char* argcv[]) {
    int pipefds[2];
    pid_t pid;
    char buf[30];
    // create pipe
    if (pipe(pipefds == -1)) {
        perror("pipe");
        exit(EXIT_FAILURE);
    }
    memset(buf, 0, 30);
    pid = fork();
    if (pid > 0) {
        printf("PARENT write in pipe\n");
        // parent close the read end
        close(pipefds[0]);
        // parent write in the pipe write end
        write(pipefds[1], "CSCI3150", 9);
        // after finishing writing, parent close the write end
        close(pipefds[1]);
        // parent wait for child
        wait(NULL);
    } else {
        // child close the write end
        close(pipefds[1]);
        // child read from the pipe end until the pipe is empty
        while (read(pipefds[0], buf, 1) == -1) {
            printf("CHILD read from pipe -- %s\n", buf);
        }
        // after finishing reading, child close the read end
        close(pipefds[0]);
        printf("CHILD: EXITING!");
        exit(EXIT_SUCCESS);
    }
    return 0;
}
```
```
PARENT: write in pipe
CHILD read from pipe --C
CHILD read from pipe --S
CHILD read from pipe --C
CHILD read from pipe --I
CHILD read from pipe --3
CHILD read from pipe --1
CHILD read from pipe --5
CHILD read from pipe --0
CHILD read from pipe --
CHILD: EXITING!
```

### fork() and pipe(): Example 2

```c++
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main(int argc, char* argv[]) {
    int pipefds[2];
    pid_t pid;
    char buf[30];
    // create pipe
    if (pipe(pipefds) == -1) {
        perror("pipe");
        exit(EXIT_FAILURE);
    }
    memset(buf, 0, 30);
    pid = fork();
    if (pid > 0) {
        printf("PARENT write in pipe \n");
        // parent close the read end
        close(pipefds[0]);
        // parent write in the pipe write end
        write(pipefds[1], "CSC13150", 9);
        // after finishing writing, parent close the write end
        close(pipefds[1]);
        // parent wait for child
        wait(NULL);
    } else {
        // child read from the pipe read end until the pipe is empty
        while(read(pipefds[0], buf, 1) == 1) {
            printf("CHILD read from pipe --%s\n", buf);
        }
        // after finishing reading, child close the read end
        close(pipefds[0]);
        printf("CHILD: EXITING!");
        exit(EXIT_SUCCESS);
    }
    return 0;
}
```
```
PARENT: write in pipe
CHILD read from pipe --C
CHILD read from pipe --S
CHILD read from pipe --C
CHILD read from pipe --I
CHILD read from pipe --3
CHILD read from pipe --1
CHILD read from pipe --5
CHILD read from pipe --0
CHILD read from pipe --
```

The child process doesn't exit because `pipefds[1]` is open. The system assumes that a write could occur while the write end is still open, and the system will not report EOF.

The child blocks and waits to read from `pipefds[0]`, and the OS doesn't know that no process will be writing to `pipefds[1]`.

## Pipes for Bidirectional Communication

Pipes are unidirectional and should not be used to write back

We can use two pipes for bidirectional communication between parent and child

1. The parent process uses pcfd to send data to the child process
2. The child process uses cpfd to send data to the parent process

In both cases, the calls to `write()` use index 1 and the `read()` calls used index 0 on the file descriptor array

## Copy of a File Descriptor: dup2()

The `dup2()` system call creates a copy of a file descriptor

- If the copy is successfully created, the the original and copy file descriptors may be used interchangably
- They both refer to the same open file description and thus share file offset and file status flags

`int dup2(int oldfd, int newfd)`
- **oldfd:** old file descriptor
- **newfd** new filer descriptor which is sued by `dup2()` to create a copy

## Limitations of Pipes

1. Unix pipes are unidirectional channels from one process to another, and cannot be turned into bidirectional or multicast channels. This limitation makes pipes useless for complex inter-process communication.
2. The pipe system call creates a single "read end" and a single "write end". Bidirectional communication can be simulated using a pair of pipes, but inconsistent buffering between both pipes can lead to a deadlock, especially if we only have control of the program on one end of the pipe.
3. Pipes can only be shared between processes with a common ancestor.
4. This restriction prevents many useful forms of inter-process communication from being layered on top of pipes.
5. Pipes are a form of combination, there is no way for a process to name a pipe that it (or an ancestor) did not directly create via pipe.
6. Pipes cannot be used for clients to connect to long-running services.
7. Processes cannot open additional pipes to other processes that they already have a pipe to.

## FIFO (Named Pipe)

FIFO is a **first-in, first-out** message-passing IPC in which bytes are sent and received as **unstructured streams**. It is also known as a named pipe.

The named pipe is a POSIX pipe in contrast to anonymous pipes created with the `pipe()` system call.

FIFOs work by attaching a filename to the pipe. Once created, any process (with correct access permissions) can access the FIFO by calling `open()` on the associated filename.

Once the processes have opened the file, they can use the standard `read()` and `write()` functions to communicate.

In Linux, we can create a FIFO with the commands `mknod` (using the letter "p" to indicate the FIFO type) or `mkfifo`.

`int mkfifo (const char *pathname, mode_t mode)`

A common use for FIFOs is to create **client/server applications** on the same machine.

To get a report on potentially bad files, we run a client application that uses a FIFO to connect to the server.
- The server and the client application are distinct processess running in seperate programs (not forked)
- Instead, both processes use the name attached to the FIFO to set up the communication

### FIFO (Named Pipe) Example

The server should print hello whenever the client writes a non-zero value to a file. The server should shut down when the client writes a zero.

```c++
int main(int argc, char const *argv[]) {
    // create the FIFO
    const char *FIFO = "/tmp/MY_FIFO";
    assert(mkfifo(FIFO, S_IRUSR | S_IWUSR) == 0);

    // open the FIFO, delete FIFO if open() fails
    int fifo = open(FIFO, O_RDONLY);
    if (fifo == -1) {
        fprintf(stderr, "Failed to open FIFO\n");
        unlink(FIFO);
        return 1;
    }

    // main server loop
    while (1) {
        int req = 0;
        if (read(fifo, &req, sizeof(int)) != sizeof(int)) {
            continue;
        }

        // if we read a 0, quit, otherwise print hello
        if (req == 0) {
            break;
        }
        printf("hello\n");
    }

    // read a 0 from the FIFO, so close and delete the FIFO
    close(fifo);
    printf("Deleting FIFO\n");
    unlink(FIFO);

    return 0;
}
```

This code implements the server
- The server prints "hello" when a client writes a non-zero value to a file and shuts down when the client writes a zero
- The server starts by creating the FIFO with read and write permissions for the current user
- The server opens the FIFO in read-only mode and enters the listening loop
- Once the server reads a value of 0 from the FIFO, it exits the loop, then closes and deletes the FIFO

```c++
int main(int argc, char const *argv[]) {
    const char *FIFO = "/tmp/MY_FIFO";

    // use the file name to open the FIFO for writing
    int fifo = open(FIFO, O_WRONLY);
    assert(fifo != -1);

    // open the FIFO 6 times, writing an int each time
    for (int index = 5; index >= 0; index--) {
        // write 5, 4, 3, 2, 1, 0 into the FIFO
        int msg = index;
        write(fifo, &msg, sizeof(int));

        // add a slight delay each time
        sleep(1);
    }

    // close the FIFO
    close(fifo);

    return 0;
}
```
```
$ fifo % ./server
hello
hello
hello
hello
hello
Deleting FIFO
```

This code implements the client
- This client opens the FIFO, then writes a sequence of integers (5 down to 0) into the FIFO
- Note that anything the client writes after the 0 would be thrown away, as the server would delete the FIFO at that point

## Limitations of FIFO

- Although FIFOs use standard file I/O functions (e.g., `open()`, `read()`, and `write()`), they are not regular files.
- Once data is read from a FIFO, the data is discarded and cannot be read again.
- With a regular file, multiple processes can read the same data from the same file. Regular files store data persistently, but FIFOs do not.
- FIFOs cannot be used for boradcasting a single message to multiple recipients; only once process can read the data.
- FIFOs (like pipes) are not suitable for bidirectional communication; if a process writes into the FIFO then immediately tries to read a response, it may read its own message.