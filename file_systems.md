# File Systems

## Why Study File Systems?

The physical characteristics drive the following key features of file systems:
- **Named Data:** Human-readable names (e.g., file names) given to data
- **Performance:** By grouping, ordering, and scheduling disk operations so that high latency is hidden/amortized
- **Reliability:** Unexpected power-cycled do not corrupt the data
- **Controlled Sharing:** Determine who can read/write/execute certian files sequentially/simultaneously without corrupting

## Why are Magnetic Disks Slow?

- Platters rotate at a constant speed
- Each surface has data in the form of several tracks
- Each track is seperated in sectors/blocks

## How to Read From The Disk?

1. The arm moves the disk head to the right track. This is a "seek". It is extremely slow (> 10 ms).
2. Then, the head waits for the correct sector to come below. This is relatively faster because of constant motion (< 10 ms) determined by the disk RPM.
3. Then, read the sector and send it to the CPU. This is much faster (SATA rate: ~500 MB/s).

## Disk Acess Time

- Disk access time = *seek time* + *rotation time* + *transfer time*
- The mechanical movement of the disk arm makes the seek time often the bottleneck.
- It takes less time to move between adjacent tracks, so a sequential read means less disk seeking.
- If data is read randomly, the disk arm must seek randomly, leading to a very slow operation. Even then, it applies algorithms similar to the ones in elevators to service requests.

## How About SSDs?

Are they "true random access" – meaning do they take the same time as sequential even when accessed randomly?
- No, because accesses are still at sector or page level
- Accessing just 1 byte experiences the full sector latency
- Anticipatory prefetching an adjacent page is common practice, which hides latency for sequential, but not for random
- Overall, random access is faster than magnetic, but not as fast as sequential

From that same logic, even the main memory, which is called Random Access Memory (RAM), is not truly random access
- You will always have some leverage accessing sequentially
- However, the RAM is much better in random access than any type of disk

## File System Abstraction

A file system is an OS abstraction that provides persistent and named data.

Key components:
- **Files:** Contain metadata and actual data
- **Directories:** Special files that contain names and pointers to actual files

## File Systems Terms

- **Path:** A string identifying a file (e.g., `/home/davidkebo/labs/main.cpp`)
- **Root Directory:** Think of the file system as a tree whose root is the root directory often denoted by the `/`
- **Absolute Path:** A path starting from the root `/`
- **Relative Path:** A path relative to the *current working directory* and does not start with a `/`
- **Hard Link:** The mapping between the name and the underlying file
    - There can be multiple hard links to the same file (e.g., short cuts)
    - Means that the directory tree is *not always a tree*
- **Volume:** A logical representation of the disk drive that contains a single file system (i.e., a single directory tree)
    - E.g., A USB drive contains a file system (if there is only 1 volume in it). Each logical drive in your device has a seperate file system.
- **Mounting:** Mounting a volume creates a mapping from some path in an existing file system to the root directory of the mounted volume's file system.

## UNIX Directory API

### Current Directory

```c++
#include <unistd.h>

char* getcwd(char* buf, size_t size);
/* get current working directory */
```

```c++
// Example:
void main(void) {
    char mycwd[PATH_MAX];

    if (getcwd(mycwd, PATH_MAX) == NULL) {
        perror("Failed to get current working directory");
        return 1;
    }
    printf("Current working directory: %s\n", mycwd);
    return 0;
}
```

### Open, Read, Close

Read is *stateful* with a *cursor*.

Reading the same directory again gives back the next file in the directory.

```c++
#include <dirent.h>

int main(int argc, char* argv[]) {
    struct dirent* direntp;
    DIR* dirp = opendir(argv[1]);
    
    while ((direntp = readdir(dirp)) != NULL) {
        printf("%s\n", direntp->d_name);
    }

    closedir(dirp);
    return 0;
}
```

### Traversal

```c++
#include <dirent.h>

DIR* opendir(const char* dirname);
/* returns pointer to directory object */

struct direct* readdir(DIR* dirp);
/* read successive entries in directory 'dirp' */

int closedir(DIR* dirp);
/* close directory stream */

void rewinddir(DIR* dirp);
/* reposition pointer to beginning of directory */
```

## File System Organization

- First, each sector/block is numbered from 0, 1, ...
- The larger the disk, the more sectors there are
- Then, the file system contains the following components:
    - **Superblock:** Contains metadata about the file system and the size is OS dependant
    - **Inode Table:** An array of inode structs where each struct contains info of a file (e.g., size, owner ID, last modification) and each inode as an identification number which is also the index into the inode table
    - **Data Area:** Contains the file content and each file can be ≥ 1 block

## Creating a New File

Let us create a file called "newfile" that is 12KB in size, and a disk block is 4KB.

First, we need a free inode to put the file metadata, and then we need to find 3 free disk blocks to put the actual data.

1. Make inode
    - Inode at index 47 has metadata (perms, mod data, etc.) and a list of data blocks: 2, 20, 100
2. Copy data into the blocks 2, 20, and 100 in that order
3. Make a directory entry:

| inode # | file    |
| ------- | ------- |
| 123     | Foo.zip |
| 2       | Bar.txt |
| 47      | newfile |

### Steps in Creating a File

1. **Store Properties:** Kernel looks for a free inode and stores the metadata (e.g., permissions, size, creation date)
2. **Store Data and Record Allocations:** Kernel then looks for free disk blocks (and enough of those) and copies content there, and then updates the inode with which blocks contain data for that file
3. **Add File Name to Directory:** Kernel stores the (inode #, filename) pair in the directory entry

## Reading a File

1. Search the directory for the file name and extract its inode
2. Locate and read the inode, and then find the data block number from there
3. Read each data block in sequence and output that

## Inode's Features: Protection

File owner/creator should be able to control:
- What can be done
- By whom

Types of access:
- Read
- Write
- Execute
- Append
- Delete
- List

## How to Read File Permissions in Linux

```
dwrx------ 2 shum staff 4096 Jan 16 22:04 Mail
drwx------ 3 shum staff 4096 Jan 16 14:15 csc128
drwxr-xr-x 2 shum staff 4096 Jan 13 16:42 public
drwxr-xr-x 2 shum staff 4096 Jan 16 14:07 public_html
-rw-r--r-- 1 shum staff  628 Jan 15 20:04 verse
```

From left to right:
1. Set of ten permission flags
2. Link count
3. Owner
4. Associated group
5. Size
6. Date of last modification
7. Name of file

### Ten Permission Flags

| 1   | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
| --- | - | - | - | - | - | - | - | - | -- |
| d/- | r | w | x | r | w | x | r | w | x  |

- **Flag 1:** "d" if a directory, "-" if a normal file
- **Flags 2, 3, 4:** read, write, execute permission for user (owner) of file
- **Flags 5, 6, 7:** read, write, execute permission for group
- **Flags 8, 9, 10:** read, write, execute permission for other (world)

- **-:** Flag is not set
- **r:** File is readable
- **w:** File is writeable; for directories, files may be created or removed.
- **x:** File is executable; for directories, files may be listed.
- **s:** Set group ID (sgid); for directories, files created there are associated with the same group as the directory rather than the default group of the user. Subdirectories created there will not only have the same group, but will also inherit the sgid setting.

## Characteristics of Files

- Most files are small
- Most of the space is occupied by the rare big ones

## Device Files and Inodes

How to handle device files (e.g., terminal input/output, printer)?

Inodes for device files are regular files are different:
- They both point to disk blocks
- However, device files' inodes point to *device driver code* instead of regular data blocks

## Links

**Hard Link:**
- Directory entry contains the *inode number*
- Creates another name (path) for the file
- Each is "first class"

**Soft Link of Symbolic Link:**
- Directory entry contains the *name of the file*
- Maps one name to another name

### Hard Links

The shell command `ln /dirA/name1 /dirB/name2` is typically implemented using the `link` system call:

```c++
#include <stdio.h>
#include <unistd.h>

if (link("/dirA/name1", "/dirB/name2") == -1) {
    perror("failed to make new link in /dirB");
}
```

### Hard Links: Unlink

```c++
#include <stdio.h>
#include <unistd.h>

if (unlink("/dirA/name1") == -1) {
    perror("failed to delete link in /dirA");
}

if (unlink("/dirB/name2") == -1) {
    perror("failed to delete link in /dirB");
}
```

### Symbolic (Soft) Links

The shell command `ln -s /dirA/name1 /dirB/name2` is typically implemented using the `symlink` system call:

```c++
#include <stdio.h>
#include <unistd.h>

if (symlink("/dirA/name1", "/dirB/name2") == -1) {
    perror("failed to create symbolic link in /dirB");
}
```

### Links: Example

```
$ ls - lrt
total 1
semaphore.h

$ ln semaphore.h ./sema.h
$ ln -s semaphore.h ./softsema.h
$ ls -lrt
total 3
semaphore.h
sema.h
softsema.h -> semaphore.h
```






