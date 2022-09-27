# UNIX File I/O

## I/O

I/O is the process of copying data between the main memory and external devices (disk drives, terminals, networks, etc.).

Language run-time systems provide higher-level facilities for performing I/O (e.g., `printf`/`cout`, `scanf`/`cin`).
- On UNIX systems, these higher-level I/O functions eventually call *System-Level UNIX I/O functions* provided by the Kernel (e.g., `printf()` internally calls `write()`)
- The same applies to Windows and other OS's
- `printf()` calls native Windows function `WriteFile()`

## UNIX Files

A UNIX file is a sequence of m bytes B<sub>0</sub>, B<sub>1</sub>, ..., B<sub>k</sub>, ..., B<sub>m-1</sub>.

All I/O devices (networks, disks, terminals) are represented as files:
- `/dev/sda2` (`/usr` disk partition; a is the order and #2 is the partition)
- `/dev/tty2` (terminal)

Even the kernel is represented as a file:
- `/dev/kmem` (kernel memory image)
- `/proc` (kernel data structures)

All I/O is performed by reading and writing files.

## Common UNIX File Types

- **Regular Files:** A file containing user/app data (binary, text, etc.)
- **Directory Files:** A file that contains the names and locations of other files
- **Character Special and Block Special Files:** Terminals (character special) and disks (block special)
- **FIFO (Named Pipe):** A file type used for inter-process communication
- **Socket:** A file type used for network communication between processes

## UNIX I/O

**Opening and closnig files:**
- `open()` and `close()`

**Reading and writing a file:** 
- `read()` and `write()`

**Current file position:**
- Kernel maintains a file position (initially 0) for each open file (aka file pointer)
- Indicates next (byte) offset into file to read or write
- Reading and writing automatically *advance* the file pointer
- `lseek()` can be used to change the file position *manually* at programmer's wish

## Opening Files

Opening a file informs the kernel that we are getting ready to access that file.

```c++
int fd = open("/etc/hosts", O_RDONLY); // fd = 3
if (fd < 0) {
    perror("open");
    exit(1);
}
```

- Returns an indentifying integer *file descriptor*
- `fd == -1` indicates that an error occured
- Kernel keeps track of all information about the open file (e.g., current file position, permissions, etc.).

Each process created by a UNIX shell begins with three open files associated with the terminal:
- **0:** standard input
- **1:** standard output
- **2:** standard error

On succes, open returns the smallest available fd value.

## Closing Files

Closing a file informs the kernel that we are finished accessing that file.

```c++
if ((retval = close(fd)) < 0) {
    perror("close");
    exit(1);
}
```

The kernel responds by:
- Freeing some data structures
- Restoring the descriptor to the pool of available descriptors

When a process terminates for any reason, the kernel performs `close()` on all open files

## Creating a File

To create a file, we call `open()` with `flag` set to write and `mode` specifying appropriate *permissions*

`int open(const char *pathname, int flags, mode_t mode);`

We will commonly use one of these three flags: `O_RDONLY`, `O_WRONLY`, or `O_RDWR`, for read, write, or both, respectively.

The following two things specify permissions of a file:
- **Types of Access**
    - Read (R)
    - Write (W)
    - Execute (X)
- **Types of Users**
    - User/Owner (U)
    - Group (G)
    - Others (O)

For each user, there is a 3-digit octet *RWX* that determines what level of permissions they have.

For instance, if User/Owner has R, W, and X permission, they get 7 (111 in binary). If the group has only R and W permissions, everybody in that group get 6 (110) in binary. For others, no permission would give 0. Together, the permission for the file should be 760.

```c++
int main() {
    int fd = open("test.txt", O_WRONLY|O_CREAT|O_TRUNC, S_IRWXU)
    if (fd < 0) {
        perror("Cannot create file");
        exit(0);
    }
    cout << "File created successfully" << endl;
}
```

You cannot just directly put `700` for permission because 700 in decimal is not the same as 700 in octet. You can write `0700` because that represents the octal number.

## Reading Files

Reading a file copies bytes from the current file position to memory and then updates the file position.

```c++
char buf[512];
int fd = open(filename, O_RDONLY);
int nbytes;
// read up to 512 bytes from file fd
if ((nbytes = read(fd, buf, sizeof(buf))) < 0) {
    perror("read");
    exit(1);
}
```

Returns number of bytes read from file `fd` into `buf`
- Return type `size_t` is signed integer
- `nbytes < 0` indicates that an error occured
- *Short counts* (`nbytes < sizeof(buf)`) are possible and are not errors (e.g., EOF when the file does not have `nbytes` of data starting from the file pointer, reading text from terminal where user gave `< nbytes`, etc.).

### Reading Files: Example

Output of the following program reading a file containing "foobar":

```c++
char buf[2];
int fd = open(filename, O_RDONLY);
int nbytes = read(fd, buf, sizeof(buf)); // buf = ?
nbytes = read(fd, buf, sizeof(buf)); // buf = ?
```

Answer: "fo" and "ob"

## Writing Files

Writing a file *copies* bytes from memory to the current file position, and then *updates* the current file position.

```c++
char buff[512];
int nbytes; // number of bytes written
int fd = open(filename, O_WRONLY|O_CREAT, 0700);
// then write up to 512 bytes from buf to file fd
if ((nbytes = write(fd, buf, sizeof(buf))) < 0) {
    perror("write");
    exit(1);
}
```

Returns number of bytes written from `buf` to file `fd`
- `nbytes < 0` indicates that an error occurred
- As with reads, short counts are possible and are not errors

### Writing Files: Example

What would be the content of the file after executing the following program?

```c++
char buf[] = {'a', 'b'};
int fd; // file descriptor
// open the file fd
// then write up to 2 bytes from buf to file fd
write(fd, buf, sizeof(buf)); // file = ?
write(fd, buf, sizeof(buf)); // file = ?
```

Answer: ab, abab

## File Representation: Three Tables

**Descriptor Table**
- Each process has its own seperate descriptor (only accessible directly by the kernel)
- Entries are indexed by the process's open file descriptors
- Each entry points to a *File Table* entry

**File Table**
- Shared by all processes
- Each entry consists of
    - File Cursor
    - Reference Count: Number of descriptor entries from all processes that point to it
    - Pointer to the v-node table entry, along with status flags

**v-node Table**
- Each entry (1 per file) contains information about the file meta data (e.g., access/modification date, permissions)

## File Sharing

Two discinct DTs sharing the same disk file through two distinct open FT entries
- E.g., calling `open` twice with the same `filename` argument
- Every `open` creates a new DT and a new FT entry, also possibly a vT entry

### File Sharing: Example

Suppose the disk file foobar.txt consists of the six ASCII characters "foobar". Then what is the output of the following program?

```c++
int main() {
    char c;
    int fd1 = open("foobar.txt", O_RDONLY, 0);
    int fd2 = open("foobar.txt", O_RDONLY, 0);
    read(fd1, &c, 1);
    cout << c << endl;
    read(fd2, &c, 1);
    cout << c << endl;
}
```

Answer: the descriptors `fd1` and `fd2` each have their own open file table entry so each descriptor has its own file position for foobar.txt. Thus, the read from `fd1` read the first byte of foobar.txt and the output is "f". Same is true for `fd2` and the output is again "f".





