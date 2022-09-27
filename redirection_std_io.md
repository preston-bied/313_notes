# UNIX I/O File Descriptors, Redirection, and Standard I/O

## Why File Descriptors?

Why do we need to open and close sessions when we read from or write to files?

Because the system must internally store (cache) a lot of information about the file as we access it.
- Where is the file stored on which storage device?
- Where are we as we sequentially traverse the file?
- Which parts of the file are cached and where?
- Who is accessing the file right now?

## File Descriptors and `fork()`

With `fork()`, the child inherits content of parent's address space, including the file descriptor table.

Suppose the disk file `foobar.txt` consists of the six ASCII characters "foobar".

What is the output of the following program?

```c++
int main() {
    char c;
    int fd = open("foobar.txt", O_RDONLY);
    if (fork() == 0) {
        read(fd, &c, 1);
        return 0;
    } else {
        wait(0);
        read(fd, &c, 1);
        cout << "c=" << c << endl;
    }
}
```

Answer: The child inherits the parent's descriptor table and all processes share the same file table. Thus, the descriptor `fd` in both the parent and child points to the same open file table entry. When the child reads the first byte of the file, the file position increments by 1. Thus, the parent reads the second byte and the output is "c=o".

## I/O Redirection

Question: How does a shell implement I/O redirection?

`unix> ls > foo.txt`

Answer: By calling the `dup2(oldfd, newfd)` function which copies (per-process) the descriptor table entry `oldfd` to entry `newfd`

### I/O Redirection: Example

Assuming that the disk file `foobar.txt` consists of six ASCII characters "foobar", what is the output?

```c++
int main() {
    int fd1 = open("foobar.txt", O_RDONLY);
    int fd2 = open("foobar.txt", O_RDONLY);
    char c;
    read(fd2, &c, 1); // read a char fd2
    dup2(fd2, fd1); // duplicate
    read(fd1, &c, 1); // read a char fd1
}
```

Because we are *redirecting* `fd1` to `fd2`, the output is "c=o".

## Standard I/O Streams

Standard I/O models open files as *streams*
- Abstraction for a file descriptor and a buffer in memory.

C programs begin life with three open streams (defined in `stdio.h`)
- **stdin** (standard input)
- **stdout** (standard output)
- **stderr** (standard error)

## Pros and Cons of UNIX I/O

**Pros**
- It is the most general and lowest overhead form of I/O
    - All other I/O packages are implemented using UNIX I/O functions on a UNIX system
- It provides functions for accessing file metadata

**Cons**
- Efficient data access requires some form of buffering, which is:
    - Device dependent (e.g., a whole track of a disk can be read at once)
    - Tricky and error prone
    - Both of these issues are addressed by standard I/O packages

## Pros and Cons of Standard I/O

**Pros**
- Portable code (same code works on Windows and UNIX)
- Buffering increases efficiency by decreasing the number of `read` and `write` system calls
- Less burden on the programmer

**Cons**
- Provides no function for accessing file metadata
- Standard I/O is not appropriate for input and output on network sockets

## Choosing I/O Functions

General rule: use the highest-level I/O functions you can
- Many C programmers are able to do all of their work using the standard I/O functions

When to use standard I/O:
- When working with disk or terminal files

When to use raw UNIX I/O:
- Inside signal handlers, because UNIX I/O is async-signal-safe
- In rare cases when you need absolute highest performance


