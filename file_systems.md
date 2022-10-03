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

## File System "Tree"

**Hard Link:** The mapping between the name and the underlying file
- There can be multiple hard links to the same file (e.g., short cuts)
- Means that the directory tree is *not always a tree*

## File System Terms

**Volume:** A logical representation of the disk drive that contains a single file system (i.e., a single directory tree)

E.g., A USB drive contains a file system (if there is only 1 volume in it). Each logical drive in your device has a seperate file system.

**Mounting:** Mounting a volume creates a mapping from some path in an existing file system to the root directory of the mounted volume's file system.

## UNIX Directory API: Current Directory

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






