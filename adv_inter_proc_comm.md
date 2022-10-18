# Advanced Inter-Process Communication

## What is Inter-Process Communication?

Inter-process communication (IPC) is a mechanism that allows processes to communicate with each other and synchronize their actions.

**IPC Methods:**
- Pipes and FIFO
- Message Passing
- Shared Memory
- Semaphore Sets
- Signals

## Message Queue

A message queue is a linked list of messages stored within the kernel and identified by a message queue identifier. A new queue is created or an existing queue opened by `msgget()`.

```c++
#include <sys/msg.h>
int msgget(key_t key, int flag);
```

The `msgget()` function returns the message queue identifier associated with the argument key.

New messages are added to the end of a queue by `msgsnd`.

Messages are fetched from a queue by `msgrcv`. We don't have to fetch the messages in a first-in, first-out order. Instead, we can fetch messages based on their type field.

Each queue has the following `msqid_ds` structure associated with it:

```c++
struct msqid_ds {
    struct ipc_perm msg_perm;
    msgqnum_t       msg_qnum;   // # of messages on queue
    msglen_t        msg_qbytes; // max # of bytes on queue
    pid_t           msg_lspid;  // pid of last msgsnd()
    pid_t           msg_lrpid;  // pid of last msgrcv()
    time_t          msg_stime;  // last msgsnd() time
    time_t          msg_rtime;  // last msgrcv() time
    time_t          msg_ctime   // last change time
}
```

### Features of Message Queues

- Message queues support priority of messages, changing the FIFO order of the messages.
- Multiple processes can read from or write into the same message queue – not possible in FIFO.
- Message queues do not require the ends to be simultanously connected.
- FIFO requires both processes connected.
- FIFO is analagous to makeing a phone call, while message queues are analogous to leaving a voicemail.
- The recipient process can set up notification of message receipt using `mq_notify()`. This way the receiver does not need to block for a message.

### Operations on Message Queues

```c++
mqd_t mq_open(const char *name, int oflag, mode_t mode, struct mq_attr *attr);
int mq_send(mqd_t mqdes, const char *msg_ptr, size_t msg_len, unsigned int msg_prio);
size_t mq_receive(mqd_t mqdes, char *msg_ptr, size_t msg_len, unsigned int *msg_prio);
int mq_close(mqd_t mqdes);
```

### Message Queue: Example

```c++
send(char* msg) {
    mqd_t mq = mq_open("/testqueue", O_RDWR|O_CREAT, 0664, 0);
    if (mq_send(mq, msg, strlen(msg)+1, 0) < 0) {
        perror("MQ send failire");
        exit(0);
    }
    printf("MQ Put: %s\n", av[1]);
    return 0;
}
receive() {
    mqd_t mq = mq_open("/testqueue", O_RDWR|O_CREAT, 0664, 0);
    struct mq_attr attr;
    mq_getattr(mq, &attr);  // get attribute
    char *buf = (char*) malloc(attr.mq_msgsize);
    mq_receive(mq, buf, attr.mq_msgsize, NULL);
    printf("MQ Receive Got: %s\n", buf);
    // clean up
    mq_close(mq);
    mq_unlink("/testqueue");    // remove from kernel
    return 0;
}
```

### Message Queues in the File System

Message queues do not get real file system entries. Rather, they live in a virtual file system that represents the kernel.

The path is `/dev/mqueue`.

If you create a message queue with the name `/my_test_queue`, you will find it as the following path: `/dev/mqueue/my_test_queue`.

You can remove a message queue by either calling `mq_unlink()` or by using the command `rm /dev/mqueue/my_test_queue`.

## Shared Memory

Shared memory allows multiple processes to share a region of memory. This is the fastest form of IPC because there is no need to copy data between the client and the server.

Using shared memory requires synchronizing access to a given region among multiple processes.

If the server places data into a shared memory regin, the client shouldn't try to access the data until the server is done. *Semaphores* are used to synchronize shared memory access.

The OS maintains the shared region.

Processes can map/unmap the region.

## Memory Mapping

The `mmap` function maps portions of a file into the address space of a process.

This is conceptually similar to attaching a shared memory segment using the `shmat` function. The main different is that the memory segment mapped with `mmap` is backed by a file.

### Memory Mapped File

When working with files, it may be inconvenient to process structured information. This could be much easier using simple pointer arithmetic if data was in memory. The system call for that is `mmap`, whose signature is as follows:

```c++
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
```

- `addr` is a hint, which can be set to `NULL`
- `length` is the length of the segment
- `prot` must match with how you opened the file initially
- `flags` tell whether you want a convenient local copy (MAP_PRIVATE) or your changes reflected to the disk (MAP_SHARED) and possibly other processes
- `fd` is the descriptor and `offset` counted from the beginning

### `mmap` Example

```c++
int main() {
    int fd = open("test.txt", O_RDWR);
    off_t len = lseek(fd, 0, SEEK_END); // go to end to get file size
    lseek(fd, 0, SEEK_SET); // seek back to the beginning

    char *buf = (char*) mmap(0, len, PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);
    // copying bytes 20-29 off the file to the test buffer
    char test[11];
    memcpy(test, buf+20, 10);
    test[10] = 0; // putting a NULL so it can be printed without crashing
    cout << "Bytes 20-29 in file: " << test << endl;

    // writing to file
    memcpy(buf, "Hola Mundo", strlen("Hola Mundo"));

    // unmapping
    munmap(buf, len);
    close(fd);
}
```

## Shared Memory – POSIX Functions

- `shm_open`: Creates a shared memory segment
- `ftruncate`: Sets the size of a shared memory segment
- `mmap`: Maps the shared memory object to the process's address space
- `munmap`: Unmaps from process's address space
- `shm_unlink`: Removes the shared memory segment from the kernel
- `close`: Closes the file descriptor associated with the shared memory segment

## Shared Memory Example

```c++
char* my_shm_connect(char* name, int len) {
    int fd = shm_open(name, O_RDWR|O_CREAT, 0644);
    ftruncate(fd, len); // set the length to 1024, the default is 0
    char *ptr = (char*) mmap(NULL, len, PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);   // map
    if (fd < 0) {
        perror("Cannot create shared memory\n");
        exit(0);
    }
    return ptr;
}

void send(char* message) {
    char *name = "/testing";
    int len = 1024;
    char* ptr = my_shm_connect(name, len);

    strcpy(ptr, message);   // putting data by just copying
    printf("Put message: %s\n", message);
    close(fd);  // close desc, does not remove the segment
    munmap(ptr, len);
}

void receive() {
    char *name = "/testing";
    int len = 1024;
    char* ptr = my_shm_connect(name, len);

    printf("Got message: %s\n", ptr);
    shm_unlink(name);   // this removes the segment from Kernel
    exit(0);
}
```

## POSIX Semaphores

A semaphore is a counter used to provide access to a shared data object for multiple processes.

A semaphore isn't a form of IPC similar to the others that we've described (pipes, FIFOs, and message queues).

To obtain a shared resource, a process needs to do the following:
1. Test the semaphore that controls the resource.
2. If the value of the semaphore is positive, the process can use the resource. In that case, the process decrements the semaphore value by 1, indicating that it has used one unit of the resource.
3. Otherwise, if the value of the semaphore is 0, the process goes to sleep until the semaphore value is greater than 0. When the process wakes up, it returns to step 1.

How do we synchronize processes?
- We will again need semaphores, by this time *Kernel Semaphores*
- They are visible to separate processes who do not share address space

Operations on Kernel Semaphore:
- `sem_open(name, ...)`: to create or connect to a semaphore
    - The name argument must start with a "/"
- `sem_close()`: closes a semaphore
    - It does not destroy it from Kernel
- `sem_unlink()`: removes from Kernel
    - Must be put in the destructor for PA3
- `sem_wait()`: waiting for an event
- `sem_post()`: signaling that an event has occured

```c++
#include <semaphore.h>

sem_t *sem_open(const char *name, int oflag, ... /* mode_t mode, unsigned int value */);
```